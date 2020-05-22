# Citrix Hypervisor Management API extensions

The Management API is a general and comprehensive interface to managing the
life-cycles of virtual machines, and offers a lot of flexibility in the
way that Management API providers may implement specific functionality (for example,
storage provisioning, or console handling). Citrix Hypervisor has several
extensions which provide useful functionality used in our own
XenCenter interface. The workings of these mechanisms are described
in this chapter.

Extensions to the Management API are often provided by specifying `other-config`
map keys to various objects. The use of this parameter indicates that
the functionality is supported for that particular release of
Citrix Hypervisor, but *not* as a long-term feature. We are constantly
evaluating promoting functionality into the API, but this requires the
nature of the interface to be well-understood. Developer feedback as to
how you are using some of these extensions is always welcome to help us
make these decisions.

## VM console forwarding

Most Management API graphical interfaces will want to gain access to the VM
consoles, in order to render them to the user as if they were physical
machines. There are several types of consoles available, depending on
the type of guest or if the physical host console is being accessed:

### Console access

 | Operating System  | Text                              | Graphical               | Optimized graphical       |
 | ------------------|-----------------------------------|-------------------------| --------------------------|
 | Windows           | No                                | VNC, using an API call  | RDP, directly from guest  |
 | Linux             | Yes, through VNC and an API call  | No                      | VNC, directly from guest  |
 | Physical Host     | Yes, through VNC and an API call  | No                      | No                        |

Hardware-assisted VMs, such as Windows, directly provide a graphical
console over VNC. There is no text-based console, and guest networking
is not necessary to use the graphical console. Once guest networking has
been established, it is more efficient to setup Remote Desktop Access
and use an RDP client to connect directly (this must be done outside of
the XenAPI).

Paravirtual VMs, such as Linux guests, provide a native text console
directly. Citrix Hypervisor provides a utility (called `vncterm`) to
convert this text-based console into a graphical VNC representation.
Guest networking is not necessary for this console to function. As with
Windows above, Linux distributions often configure VNC within the guest,
and directly connect to it over a guest network interface.

The physical host console is only available as a `vt100` console, which
is exposed through the XenAPI as a VNC console by using `vncterm` in the
control domain.

RFB (Remote Framebuffer) is the protocol which underlies VNC, specified
in [The RFB Protocol](http://www.realvnc.com/docs/rfbproto.pdf).
Third-party developers are expected to provide their own VNC viewers,
and many freely available implementations can be adapted for this
purpose. RFB 3.3 is the minimum version which viewers must support.

### Retrieving VNC consoles using the API

VNC consoles are retrieved using a special URL passed through to the
host agent. The sequence of API calls is as follows:

1.  Client to Master/443: RPC: `Session.login_with_password()`.

2.  Master/443 to Client: Returns a session reference to be used with
    subsequent calls.

3.  Client to Master/443: RPC: `VM.get_by_name_label()`.

4.  Master/443 to Client: Returns a reference to a particular VM (or the
    "control domain" if you want to retrieve the physical host console).

5.  Client to Master/443: RPC: `VM.get_consoles()`.

6.  Master/443 to Client: Returns a list of console objects associated
    with the VM.

7.  Client to Master/443: RPC: `VM.get_location()`.

8.  Returns a URI describing where the requested console is located. The
    URIs are of the
    form: `https://192.168.0.1/console?ref=OpaqueRef:c038533a-af99-a0ff-9095-c1159f2dc6a0`
    or `https://192.168.0.1/console?uuid=026e34fe-f0f2-20ee-5344-46d1aa922d5b`

9.  Client to 192.168.0.1: HTTP CONNECT "/console?ref=(...)". You will
    also need to pass in "session_id=<session reference>" as a cookie.

The final HTTP CONNECT is slightly non-standard since the HTTP/1.1 RFC
specifies that it should only be a host and a port, rather than a URL.
Once the HTTP connect is complete, the connection can subsequently
directly be used as a VNC server without any further HTTP protocol
action.

This scheme requires direct access from the client to the control
domain's IP, and will not work correctly if there are Network Address
Translation (NAT) devices blocking such connectivity. You can use the
CLI to retrieve the console URI from the client and perform a
connectivity check.

Retrieve the VM UUID by running:

```bash
    xe vm-list params=uuid --minimal name-label=name
```

Retrieve the console information:

```bash
    xe console-list vm-uuid=uuid
    uuid ( RO): 714f388b-31ed-67cb-617b-0276e35155ef
    vm-uuid ( RO): 8acb7723-a5f0-5fc5-cd53-9f1e3a7d3069
    vm-name-label ( RO): etch
    protocol ( RO): RFB
    location ( RO): https://192.168.0.1/console?ref=(...)
```

Use command-line utilities like `ping` to test connectivity to the IP
address provided in the `location` field.

### Disabling VNC forwarding for Linux VM

When creating and destroying Linux VMs, the host agent automatically
manages the `vncterm` processes which convert the text console into VNC. Advanced
users who wish to directly access the text console can disable VNC
forwarding for that VM. The text console can then only be accessed
directly from the control domain directly, and graphical interfaces such
as XenCenter will not be able to render a console for that VM.

Before starting the guest, set the following parameter on the VM record:

    xe vm-param-set uuid=uuid other-config:disable_pv_vnc=1

Start the VM.

Use the CLI to retrieve the underlying domain ID of the VM with:

    xe vm-list params=dom-id uuid=uuid --minimal

On the host console, connect to the text console directly by:

    /usr/lib/xen/bin/xenconsole domain_id

This configuration is an advanced procedure, and we do not recommend
that the text console is directly used for heavy I/O operations.
Instead, connect to the guest over SSH or some other network-based
connection mechanism.

## Paravirtual Linux installation

The installation of paravirtual Linux guests is complicated by the fact
that a Xen-aware kernel must be booted, rather than simply installing
the guest using hardware-assistance. This does have the benefit of
providing near-native installation speed due to the lack of emulation
overhead. Citrix Hypervisor supports the installation of several different
Linux distributions, and abstracts this process as much as possible.

To this end, a special bootloader known as `eliloader` is present in the
control domain which reads various `other-config` keys in the VM record
at start time and performs distribution-specific installation behavior.

-  `install-repository` - Required. Path to a repository; 'http',
    'https', 'ftp', or 'nfs'. Should be specified as would be used by
    the target installer, but not including prefixes, for example,  method=.

-  `install-vnc` - Default: false. Use VNC where available during the
    installation.

-  `install-vncpasswd` - Default: empty. The VNC password to use, when
    providing one is possible using the command-line of the target
    distribution.

-  `install-round` - Default: 1. The current bootloader round. Not to
    be edited by the user.

## Adding xenstore entries to VMs

Developers may wish to install guest agents into VMs which take special
action based on the type of the VM. In order to communicate this
information into the guest, a special Xenstore name-space known as
`vm-data` is available which is populated at VM creation time. It is
populated from the `xenstore-data` map in the VM record.

Set the `xenstore-data` parameter in the VM record:

    xe vm-param-set uuid=vm_uuid xenstore-data:vm-data/foo=bar

Start the VM.

If it is a Linux-based VM, install the Citrix VM Tools and use the `xenstore-read` to verify that the node exists in xenstore.

> **Note**
>
> Only prefixes beginning with `vm-data` are permitted, and anything not
> in this name-space will be silently ignored when starting the VM.

## Security enhancements

The control domain in Citrix Hypervisor has
various security enhancements in order to harden it against attack from
malicious guests. Developers should never notice any loss of correct
functionality as a result of these changes, but they are documented here
as variations of behavior from other distributions.

-  The socket interface, `xenstored`, access using `libxenstore`. Interfaces are restricted by `xs_restrict()`.

-  The device `/dev/xen/evtchn`, which is accessed by calling
    `xs_evtchn_open()` in `libxenctrl`. A handle can be restricted using
    `xs_evtchn_restrict()`.

-  The device `/proc/xen/privcmd`, accessed through
    `xs_interface_open()` in `libxenctrl`. A handle is restricted using
    `xc_interface_restrict()`. Some privileged commands are naturally
    hard to restrict (for example,  the ability to make arbitrary hypercalls),
    and these are simply prohibited on restricted handles.

-  A restricted handle cannot later be granted more privilege, and so
    the interface must be closed and re-opened. Security is only gained
    if the process cannot subsequently open more handles.

The control domain privileged user-space interfaces can now be
restricted to only work for certain domains. There are three interfaces
affected by this change:

-  The `qemu` device emulation processes and `vncterm` terminal emulation processes run as a non-root user ID
    and are restricted into an empty directory. They uses the
    restriction API above to drop privileges where possible.

-  Access to xenstore is rate-limited to prevent malicious guests from
    causing a denial of service on the control domain. This is
    implemented as a token bucket with a restricted fill-rate, where
    most operations take one token and opening a transaction takes 20.
    The limits are set high enough that they should never be hit when
    running even a large number of concurrent guests under loaded
    operation.

-  The VNC guest consoles are bound only to the `localhost` interface,
    so that they are not exposed externally even if the control domain
    packet filter is disabled by user intervention.

## Advanced settings for network interfaces

Virtual and physical network interfaces have some advanced settings that
can be configured using the `other-config` map parameter. There is a set of custom ethtool settings and some miscellaneous settings.

### ethtool settings

Developers might wish to configure custom ethtool settings for physical
and virtual network interfaces. This is accomplished with
`ethtool-<option>` keys in the `other-config` map parameter.

|  Key                | Description                                        |  Valid settings                                                              |
|  -------------------|----------------------------------------------------| -----------------------------------------------------------------------------|
|  ethtool-rx         | Specify if RX checksumming is enabled              | `on` or `true` to enable the setting, `off` or `false` to disable it         |
|  ethtool-tx         | Specify if TX checksumming is enabled              | `on` or `true` to enable the setting, `off` or `false` to disable it         |
|  ethtool-sg         | Specify if scatter-gather is enabled               | `on` or `true` to enable the setting, `off` or `false` to disable it         |
|  ethtool-tso        | Specify if tcp segmentation offload is enabled     | `on` or `true` to enable the setting, `off` or `false` to disable it         |
|  ethtool-ufo        | Specify if UDP fragmentation offload is enabled    | `on` or `true` to enable the setting, `off` or `false` to disable it         |
|  ethtool-gso        | Specify if generic segmentation offload is enabled | `on` or `true` to enable the setting, `off` or `false` to disable it         |
|  ethtool-autoneg    | Specify if autonegotiation is enabled              | `on` or `true` to enable the setting, `off` or `false` to disable it         |
|  ethtool-speed      | Set the device speed in Mb/s                       | 10, 100, or 1000                                                             |
|  ethtool-duplex     | Set full or half duplex mode                       | half or full                                                                 |

For example, to enable TX checksumming on a virtual NIC using the xe
CLI:

    xe vif-param-set uuid=<VIF UUID> other-config:ethtool-tx="on"

or:

    xe vif-param-set uuid=<VIF UUID> other-config:ethtool-tx="true"

To set the duplex setting on a physical NIC to half duplex using the xe
CLI:

    xe vif-param-set uuid=<VIF UUID> other-config:ethtool-duplex="half"

### Miscellaneous settings

You can also set a promiscuous mode on a VIF or PIF by setting the `promiscuous` key to `on`. For example, to enable promiscuous mode on a physical NIC using the xe CLI:

    xe pif-param-set uuid=<PIF UUID> other-config:promiscuous="on"

or:

    xe pif-param-set uuid=<PIF UUID> other-config:promiscuous="true"

The VIF and PIF objects have a `MTU` parameter that is read-only and
provide the current setting of the maximum transmission unit for the
interface. You can override the default maximum transmission unit of a
physical or virtual NIC with the `mtu` key in the `other-config` map
parameter. For example, to reset the MTU on a virtual NIC to use jumbo
frames using the xe CLI:

    xe vif-param-set uuid=<VIF UUID> other-config:mtu=9000

Note that changing the MTU of underlying interfaces is an advanced and
experimental feature, and may lead to unexpected side-effects if you
have varying MTUs across NICs in a single resource pool.

## Internationalization for SR names

The SRs created at install time now have an `other_config` key
indicating how their names may be internationalized.

`other_config["i18n-key"]` may be one of

-  `local-hotplug-cd`

-  `local-hotplug-disk`

-  `local-storage`

-  `Citrix Hypervisor-tools`

Additionally, `other_config["i18n-original-value-<field name>"]` gives
the value of that field when the SR was created. If XenCenter sees
a record where `SR.name_label` equals `other_config["i18n-original-value-name_label"]` (that is, the record has not changed since it was created during
Citrix Hypervisor installation), then internationalization will be applied.
In other words, XenCenter will disregard the current contents of
that field, and instead use a value appropriate to the user's own
language.

If you change `SR.name_label` for your own purpose, then it no longer is
the same as `other_config["i18n-original-value-name_label"]`. Therefore,
XenCenter does not apply internationalization, and instead
preserves your given name.

## Hiding objects from XenCenter

Networks, PIFs, and VMs can be hidden from XenCenter by adding the
key `HideFromXenCenter=true` to the `other_config` parameter for the object. This capability
is intended for ISVs who know what they are doing, not general use by
everyday users. For example, you might want to hide certain VMs because
they are cloned VMs that shouldn't be used directly by general users in
your environment.

In XenCenter, hidden Networks, PIFs, and VMs can be made visible,
using the View menu.
