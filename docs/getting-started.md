# Getting Started

Citrix Hypervisor includes a Remote Procedure Call (RPC) based API providing programmatic
access to the extensive set of Citrix Hypervisor management features and
tools. You can call the Citrix Hypervisor Management API from a remote system or
from local to the Citrix Hypervisor server.

It's possible to write applications that use the Citrix Hypervisor Management
API directly through raw RPC calls. However, the task of developing third-party
applications is greatly simplified by using a language binding. These language
bindings expose the individual API calls as first-class functions in the
target language. The Citrix Hypervisor SDK provides language bindings and
example code for the C, C\#, Java, Python, and PowerShell programming
languages.

## System Requirements and Preparation

The first step towards using the SDK is to install
Citrix Hypervisor. Citrix Hypervisor is available for
download at <http://www.citrix.com/downloads/citrix-hypervisor/>. For detailed instructions on
how to set up your development host, see the [Citrix Hypervisor Product Documentation](https://docs.citrix.com/en-us/citrix-hypervisor).
When the installation is complete, note the *host IP address* and the *host password*.

## Downloading

The SDK is packaged as a ZIP file and is available as a free download
from <http://www.citrix.com/downloads/citrix-hypervisor/>.

## SDK Languages

The extracted contents of the SDK ZIP file are in the `CitrixHypervisor-SDK`
directory. The following is an overview of its structure. Where
necessary, subdirectories have their own individual README files.

>     **Note**: The examples provided aren't the same across all
>     the SDK languages. If you intend to use one language, it's
>     advisable to browse the sample code available in the others as well.

The top level of the CitrixHypervisor-SDK directory includes the Citrix Hypervisor Management API
Reference document. This document describes in more detail the API semantics and
the wire protocol of the RPC messages.

The API supports two wire formats, one based upon XML-RPC and one based upon JSON-RPC (v1.0 and v2.0 are both recognised).
The format supported by each of the SDK languages is specified in the following sections.

### C

The CitrixHypervisor-SDK directory contains the following folders that are
relevant to C programmers:

-  `libxenserver`

    The Citrix Hypervisor SDK for C.

    -  `libxenserver/bin`

        The libxenserver compiled binaries.

    -  `libxenserver/src`

        The libxenserver source code and examples and a Makefile to build
        them. Every API object is associated with a header file, which
        contains declarations for all that object's API functions. For
        example, the type definitions and functions required to invoke
        VM operations are all contained in `xen_vm.h`.

#### C SDK dependencies

The C SDK supports the XML-RPC protocol.

**Platform supported**:

-  Linux
-  Windows (under cygwin)

**Library**:

The library is generated as `libxenserver.so` that is linked by C programs.

**Dependencies**:

-  XML library (libxml2.so on GNU Linux)
-  Curl library (libcurl3.so)

**Examples**:
The following simple examples are included with the C SDK:

-  `test_vm_async_migrate`: Shows how to use asynchronous API
    calls to migrate running VMs from a slave host to the pool master.

-  `test_vm_ops`: Shows how to query the capabilities of a host,
    create a VM, attach a fresh blank disk image to the VM and then
    perform various powercycle operations.

-  `test_failures`: Shows how to translate between error strings and
    enum\_xen\_api\_failure.

-  `test_event_handling`: Shows how to listen for events on a
    connection.

-  `test_enumerate`: Shows how to enumerate the various API
    objects.

-  `test_get_records`: Shows how to obtain information on API
    objects such as hosts, VMs, and storage repositories.

### C &#35;

The CitrixHypervisor-SDK directory contains the following folders that are
relevant to C\# programmers:

-  `XenServer.NET`

    The Citrix Hypervisor SDK for C\#.NET.

    -  `XenServer.NET/bin`

        XenServer.NET ready compiled binaries.

    -  `XenServer.NET/samples`

        XenServer.NET examples shipped as a Microsoft Visual studio
        solution.

    -  `XenServer.NET/src`

        XenServer.NET source code shipped as a Microsoft Visual Studio
        project. Every API object is associated with one C\# file. For
        example, the functions implementing the VM operations are
        contained within the file `VM.cs`.

#### C\# SDK dependencies

The C\# SDK supports the JSON-RPC v2.0 protocol.
The C\# SDK is backwards compatible and can be used to communicate with hosts running an older version of Citrix Hypervisor or XenServer.
For server versions prior to XenServer 7.3, the C# SDK uses the XML-RPC protocol, with the exception of XenServer 7.1 LTSR where the JSON-RPC v1.0 protocol is used.

**Platform supported**:

-  Windows with .NET version 4.5

**Library**:

-  The library is generated as a Dynamic Link Library `XenServer.dll` that C\# programs can reference.

**Dependencies**:

-  `Newtonsoft.Json.dll` is needed for the `XenServer.dll` to be able to communicate with the JSON-RPC backend. We ship a patched 10.0.2 version and recommend that you use this one, though others may work.

-  `CookComputing.XMLRpcV2.dll` is needed for the `XenServer.dll` to be able to communicate with the XML-RPC backend. We ship a patched 2.5 version and recommend that you use this one, though others may work.

**Examples**:

The following examples are included with the C\# SDK in the directory
`CitrixHypervisor-SDK/XenServer.NET/samples` as separate projects of the
`XenSdkSample.sln` solution:

-  `GetVariousRecords`: logs to a Citrix Hypervisor server and displays
    information about hosts, storage, and virtual machines;

-  `VmPowerStates`: logs to a Citrix Hypervisor server, finds a VM and
    takes it through the various power states. Requires a shutdown VM
    to be already installed.

### Java

The CitrixHypervisor-SDK directory contains the following folders that are relevant to Java programmers:

-  `XenServerJava`

    The Citrix Hypervisor SDK for Java

    -  `XenServerJava/bin`

        Java compiled binaries.

    -  `XenServerJava/javadoc`

        Java documentation.

    -  `XenServerJava/samples`

        Java examples.

    -  `XenServerJava/src`

        Java source code and a Makefile to build the code and the
        examples. Every API object is associated with one Java file. For
        example the functions implementing the VM operations are
        contained within the file `VM.java`.

#### Java SDK dependencies

The Java SDK supports the XML-RPC protocol.

**Platform supported**:

-  Linux
-  Windows

**Library**:

-  The language binding is generated as a Java Archive file `xenserver.jar` that is linked by Java programs.

**Dependencies**:

-  `xmlrpc-client-3.1.jar`
-  `xmlrpc-common-3.1.jar`
-  `ws-commons-util-1.0.2.jar`

These jars are needed for the `xenserver.jar` to be able to communicate with the xml-rpc server. These jars are shipped alongside the `xenserver.jar`.

**Examples**:

Running the main file `CitrixHypervisor-SDK/XenServerJava/samples/RunTests.java` runs a series of examples included in the same directory:

-  `AddNetwork`: Adds a new internal network not attached to any NICs;

-  `SessionReuse`: Shows how a Session object can be shared
    among multiple Connections;

-  `AsyncVMCreate`: Makes asynchronously a new VM from a built-in
    template, starts, and stops it;

-  `VdiAndSrOps`: Performs various SR and VDI tests, including creating
    a dummy SR;

-  `CreateVM`: Creates a VM on the default SR with a network and DVD
    drive;

-  `DeprecatedMethod`: Tests a warning is displayed when a deprecated
    API method is called;

-  `GetAllRecordsOfAllTypes`: Retrieves all the records for all types
    of objects;

-  `SharedStorage`: Creates a shared NFS SR;

-  `StartAllVMs`: Connects to a host and tries to start each VM on it.

### PowerShell

The CitrixHypervisor-SDK directory contains the following folders that are
relevant to PowerShell users:

-  `XenServerPowerShell`

    The Citrix Hypervisor SDK for PowerShell.

    -  `XenServerPowerShell/XenServerPSModule`

        The Citrix Hypervisor PowerShell module.

    -  `XenServerPowerShell/samples`

        PowerShell example scripts.

    -  `XenServerPowerShell/src`

        C\# source code for the Citrix Hypervisor PowerShell cmdlets.

Detailed installation instructions are provided within the README file
accompanying the module. After the module is installed, you can obtain an overview of
the cmdlets by typing:

    PS> Get-Help about_XenServer

#### PowerShell SDK dependencies

The PowerShell module supports the same RPC protocols as the C# SDK.
> **Note** This module is generally, but not fully, backwards compatible. To communicate with hosts running older versions of Citrix Hypervisor or XenServer, it is advisable to use the module of the same version as the host.

**Platform supported**:

-  Windows with .NET Framework 4.5 and PowerShell v4.0

**Library**:

-  `XenServerPSModule`

**Dependencies**:

-  `Newtonsoft.Json.dll` is needed for the module to be able to communicate with the JSON-RPC backend. We ship a patched 10.0.2 version and recommend that you use this one, though others may work.
-  `CookComputing.XMLRpcV2.dll` is needed for the module to be able to communicate with the XML-RPC backend. We ship a patched 2.5 version and recommend that you use this one, though others may work.

**Examples**:

The following example scripts are included with the PowerShell module
in the directory `CitrixHypervisor-SDK/XenServerPowerShell/samples`:

-  `AutomatedTestCore.ps1`: Shows how to log into a
    Citrix Hypervisor server, create a storage repository and a VM, and then
    perform various powercycle operations.

-  `HttpTest.ps1`: Shows how to log into a Citrix Hypervisor server,
    create a VM, and then perform operations such as VM importing and
    exporting, patch upload, and retrieval of performance statistics.

### Python

The CitrixHypervisor-SDK directory contains the following folders that are
relevant to Python developers:

-  `XenServerPython`

    This directory contains the Citrix Hypervisor Python module
    *XenAPI.py*.

    -  `XenServerPython/samples`

        Citrix Hypervisor Management API examples using Python.

#### Python module dependencies

The Python module supports the XML-RPC protocol.

**Platform supported**:

-  Linux

**Library**:

-  XenAPI.py

**Dependencies**:

-  xmlrpclib

**Examples**:

The SDK includes the following Python examples:

-  `fixpbds.py` - reconfigures the settings used to access shared
    storage;

-  `install.py` - installs a Debian VM, connects it to a network,
    starts it up and waits for it to report its IP address;

-  `license.py` - uploads a fresh license to a Citrix Hypervisor server;

-  `permute.py` - selects a set of VMs and uses live migration to move them
    simultaneously among hosts;

-  `powercycle.py` - selects a set of VMs and powercycles them;

-  `vm_start_async.py` - shows how to invoke operations
    asynchronously;

-  `watch-all-events.py` - registers for all events and prints details
    when they occur.

## Command Line Interface (CLI)

Besides using raw RPC or one of the supplied SDK languages, third-party software developers can integrate with Citrix Hypervisor servers
by using the xe command line interface `xe`. The xe CLI is installed by default on Citrix Hypervisor servers. A stand-alone remote CLI is also
available for Linux. On Windows, the `xe.exe` CLI executable is
installed along with XenCenter.

### CLI dependencies

**Platform supported**:

-  Linux
-  Windows

**Library**:

-  None

**Binary**:

-  xe on Linux
-  xe.exe on Windows

**Dependencies**:

-  None

The CLI allows almost every API call to be directly invoked from a
script or other program, silently taking care of the required session
management. The xe CLI syntax and capabilities are described in detail
in the [Citrix Hypervisor Product Documentation](https://docs.citrix.com/en-us/citrix-hypervisor/command-line-interface.html). For more resources
and examples, visit the [Citrix Knowledge Center](http://support.citrix.com).

> **Note**
>
> When running the CLI from a Citrix Hypervisor server console,
> tab completion of both command names and arguments is available.