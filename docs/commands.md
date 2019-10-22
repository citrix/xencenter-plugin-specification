# Commands

In your configuration file, each MenuItem feature has a single command as a child.
This child defines which executable or script to run when the user clicks the MenuItem.

This version of the XenCenter plug-in specification includes the following types
of command:

- Shell
- PowerShell
- XenServerPowerShell

PowerShell and XenServerPowerShell are extensions of Shell and inherit all the
properties of Shell. However, they both have extra features which make it easier
to run PowerShell scripts. For example, XenServerPowerShell commands automatically
load the Citrix Hypervisor PowerShell Module (XenServerPSModule) before executing.

![Shells](media/shells.png)
**Figure** *PowerShell and XenServerPowerShell extend Shell*

## Parameter Sets

A parameter set is a collection of four parameters that describe which items are
selected in the XenCenter resource list when your command is executed.

While each command has its own way of receiving parameters from XenCenter, the
parameters are always the same. They are delivered in sets of four that describe
the selection in the XenCenter resource list: `url`, `sessionRef`, `class`, and
`objUuid`.

Two of the parameters are used to allow communication to the relevant server:

- The `url` parameter indicates the address of the applicable standalone server
or pool master.
- The `sessionRef` parameter is the session opaque ref that can be used to
communicate with this server.

Two of the parameters are used to describe which specific object is selected:

- The `class` parameter is used to show the class of the object which is selected
in the resource list.
- The `objUuid` parameter is the UUID of this selected object.

**Example:** If you select both the local SR from a standalone server (Server A)
and the pool node from a separate pool (Pool B), two parameter sets are passed
into your plug-in:

![Four-parameter example](media/example-fourparams.png)
**Figure** *An example of two sets of four parameters.*

In general, the plug-in receives one parameter set per object selected in the
tree view, with two exceptions.

- Selecting a folder adds a parameter set per object in the folder, not the
folder itself.
- Selected the XenCenter node adds a parameter set for each stand-alone server
or pool that is connected, with the `class` and `objUuid` parameters marked with
the keyword 'blank'.

    If the XenCenter node is selected, the plug-in receives the necessary information
    to perform actions on any connected servers. However, selecting this node
    provides no contextual information about what the user wants to target. The
    'blank' keyword is used for the parameters that identify specifically what
    is selected.

**Example:** If you connect to Server A and Pool B from the previous example,
and you selected both the XenCenter node and the local storage on Server A, you
get the following parameter sets:

![Multi-parameter example](media/example-paramsets.png)
**Figure** *An example of multiple sets of parameters*

## Shell

Shell commands are the most generic command type and launch executables, batch
files, and other files which have a registered Windows extension.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE XenCenterPlugin PUBLIC "-//XENCENTERPLUGIN//DTD XENCENTERPLUGIN1//EN" "xencenter-1.dtd">

<XenCenterPlugin
      xmlns="http://www.citrix.com/XenCenter/Plugins/schema"
      version="1"
      plugin_version="1.0.0.0">

  <MenuItem
        name="hello-menu-item"
        menu="file"
        serialized="obj">

    <Shell
          filename="Plugins\Citrix\HelloWorld\HelloWorld.bat"
          window="true"
          log_output="true"
          dispose_time="0"
          param="{$type}" />

  </MenuItem>

</XenCenterPlugin>
```

### Shell Parameters

For Shell commands the parameter sets are passed through as command-line parameters
to the batch file or executable. Any extra parameters supplied using the `param`
XML attribute (see the following table) are first in the list of parameters,
followed by sets of four command line parameters representing each parameter set.

### Shell XML Attributes

> **Note:**
>
> The `filename` attribute is required and your plug-in only loads when it is set.

| Key          | Value         | Description                                  | Optional/Required | Default | Accepts Placeholders |
|--------------|---------------|----------------------------------------------|-------------------|---------|----------------------|
| `filename`     | [string]      | The file to execute, for example, an executable or a Windows batch file. The value is a relative path from your XenCenter installation directory. | Required | - | True |
| `window`       | true or false | Whether to start the process in a window or run it in the background. | Optional | true | - |
| `log_output`   | true or false | Redirects the standard output and standard error streams of the plug-in process to the XenCenter log file. | Optional | false | - |
| `param`        | [string] | A comma-separated string of extra parameters to pass to the process. You can pass a parameter with spaces by encasing it in XML-escaped quotes (&amp;quot;) | Optional | - | True |
| `dispose_time` | [float]       | The grace period XenCenter gives the plug-in after it has requested it to cancel. After this period, XenCenter assumes the plug-in has hung and warns the user. | Optional | 20.0 | - |
| `required_methods` | [string]  | A comma-separated string of API calls the plug-in wants to run. XenCenter blocks the plug-in launch and warns the user when these requirements are not met due to a role restriction under RBAC. | Optional | - | False |
| `required_method_list` | [string] | The name of a list of methods called out in the MethodList node (child of XenCenterPlugin). If `required_methods` is set, this parameter is ignored. | Optional | - | False |

For more information, see the section on RBAC protection.

## PowerShell

PowerShell commands specifically target PowerShell scripts and have several
enhancements over a basic Shell command to help you access the various parameters
XenCenter can pass to your plug-in.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE XenCenterPlugin PUBLIC "-//XENCENTERPLUGIN//DTD XENCENTERPLUGIN1//EN" "xencenter-1.dtd">

<XenCenterPlugin
      xmlns="http://www.citrix.com/XenCenter/Plugins/schema"
      version="1"
      plugin_version="1.0.0.0">

  <MenuItem
        name="hello-menu-item"
        menu="file"
        serialized="obj" />

    <PowerShell
          filename="Plugins\Citrix\HelloWorld\HelloWorld.ps1"
          debug="true"
          window="true"
          log_output="true"
          dispose_time="0"
          param="{$type}"
          function="Write-Output {$type}; read-host '[Press Enter to Exit]'" />

</MenuItem>

</XenCenterPlugin>
```

### PowerShell Object Information Array

Information regarding the target items selected in the XenCenter resource list
are stored in a PowerShell variable for easy access by your script. Inside the
`$objInfoArray` variable is an array of hash maps, each representing a parameter
set. Use the following keys to access the parameters in each hash map:

- `url`
- `sessionRef`
- `class`
- `objUuid`

```powershell
[reflection.assembly]::loadwithpartialname('system.windows.forms')

foreach ($objInfo in $objInfoArray)
{
    $outputString = "url={0}, sessionRef={1}, objName={2}, objUuid={3}" `
      -f $objInfo["url"], $objInfo["uuid"], $objInfo["class"], $objInfo["objUuid"]
    [system.Windows.Forms.MessageBox]::show("Hello from {0}!" -f $outputString)
}
```

### PowerShell Extra Parameter Array

Any additional parameters you define using the `param` XML attribute inherited
from Shell are stored in the `$ParamArray` variable as a simple array.

```powershell
[reflection.assembly]::loadwithpartialname('system.windows.forms')

foreach ($param in $ParamArray)
{
    [system.Windows.Forms.MessageBox]::show("Hello from {0}!" -f $param)
}
```

### PowerShell XML Attributes

> **Note:**
>
> The `filename` attribute inherited from Shell is required. Your plug-in only
> loads when this attribute points to a PowerShell script.

| Key      | Value         | Description                           | Optional/Required | Default        | Accepts Placeholders   |
|----------|---------------|---------------------------------------|-------------------|----------------|------------------------|
| -        | -             | [All attributes inherited from Shell] | -                 | -              | -                      |
| `debug`    | true or false | Enables debugging output that traps and details any uncaught exceptions. It is highly recommended you set `window=true` if debug is enabled. | Optional | false | - |
| `function` | [string]      | Executes the provided string as PowerShell code after the main script has finished executing. | Optional | - | True |

## XenServerPowerShell

In addition to the features provided by the PowerShell Command, the
XenServerPowerShell Command loads the Citrix Hypervisor PowerShell Module
(XenServerPSModule before executing your target PowerShell script.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE XenCenterPlugin PUBLIC "-//XENCENTERPLUGIN//DTD XENCENTERPLUGIN1//EN" "xencenter-1.dtd">

<XenCenterPlugin
      xmlns="http://www.citrix.com/XenCenter/Plugins/schema"
      version="1"
      plugin_version="1.0.0.0">

  <MenuItem
        name="hello-menu-item"
        menu="file"
        serialized="obj" />

    <XenServerPowerShell
          filename="Plugins\Citrix\HelloWorld\HelloWorld.ps1"
          debug="true"
          window="true"
          log_output="true"
          dispose_time="0"
          param="{$type}"
          function="Write-Output {$type}; read-host '[Press Enter to Exit]'" />

  </MenuItem>

</XenCenterPlugin>
```

### XenServerPowerShell Initialization Details

The full setup done to prepare your PowerShell environment for communicating
with the server is in the Initialize-Environment script in your Citrix Hypervisor
PowerShell Module (XenServerPSModule) installation directory. When your target
script begins executing, the following setup happens:

- All the cmdlet aliases are initialized
- The global session variable is initialized to store your session information

### XenServerPowerShell Parameters

The parameter sets and extra parameters can be accessed through the `$objInfoArray`
and the `$ParamArray` variables as detailed in the previous PowerShell Command section.

### XenServerPowerShell XML Attributes

> **Important:**
>
> The `filename` attribute inherited from Shell is required and your plug-in does
> only loads when it is set to point to a PowerShell script

| Key      | Value         | Description                                  | Optional/Required | Default | Accepts Placeholders |
|----------|---------------|----------------------------------------------|-------------------|---------|----------------------|
| -        | -             | [All attributes inherited from Shell]        | -                 | -       | -                    |
| `debug`    | true or false | Enables debugging output that traps and details any uncaught exceptions. It is highly recommended you set `window=true` | Optional | false | - |
| `function` | [string]      | Executes the provided string as PowerShell code after the main script has finished executing. | Optional | - | True |

## Preparing for Role Based Access Control (RBAC)

If you define the methods that a command requires in your configuration file,
the command can be prepared for when XenCenter connects to a server that uses
Role Based Access Control.

If the user does not have permission to execute an API call on a server due to
RBAC, the call fails with an `RBAC_PERMISSION_DENIED` exception. You can handle
these exceptions from within the plug-in (examine the `ErrorDescription` field
on the response for details). Alternatively, you can ask XenCenter to ensure that
the user can execute all possible commands you might need before the plug-in is run:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE XenCenterPlugin PUBLIC "-//XENCENTERPLUGIN//DTD XENCENTERPLUGIN1//EN" "xencenter-1.dtd">

<XenCenterPlugin
      xmlns="http://www.citrix.com/XenCenter/Plugins/schema"
      version="1"
      plugin_version="1.0.0.0">

  <MenuItem
        name="Hello Exe World"
        menu="file"
        serialized="obj"
        description="The worlds most friendly plug-in, it loves to say hello">

    <Shell
          filename="Plugins\Citrix\HelloWorld\HelloWorld.exe"
          required_methods="host.reboot, vm.start" />

  </MenuItem>

</XenCenterPlugin>
```

The `required_methods` attribute accepts a comma separated list of API calls in
the format `object.method`.

If the user is operating on an RBAC enabled server, XenCenter checks that the
user can execute all of these API calls on their current role. If they can't,
the plug-in is not launched and an error displayed:

![Rbac error](media/rbac-error.png)
**Figure** *An error in the Logs tab. The error is "Your current role is not
authorized to perform this action on POOL NAME".*

### Preparing for RBAC - Key White Lists

> **Important:**
>
> - Key white lists apply to advanced XenCenter keys. Only modify these lists
> if you know what you are doing.

In general, when operating under RBAC your role restricts you to modifying the
`other-config` map on objects which are directly relevant to your role. For
example, a VM Admin can modify the `other-config` map on a VM, but it cannot
modify the `other-config` map on a server.

Some specific keys have been white listed to all roles above read-only. This
setting allows XenCenter to set some advanced keys on `other-config` maps that
are otherwise inaccessible to the user's role:

| Target object                    | Key                       |
|----------------------------------|---------------------------|
| VDI, SR, network, host, VM, pool | XenCenter.CustomFields.*  |
| VDI, SR, network, host, VM, pool | folder                    |
| pool                             | EMPTY_FOLDERS             |
| task                             | XenCenterUUID             |
| task                             | applies_to                |
| network                          | XenCenterCreateInProgress |

You can enter these checks into your method list with the following
syntax:

```xml
    <MethodList name="methodList1">
        pool.set_other_config/key:folder,
        pool.set_other_config/key:XenCenter.CustomFields.*
    </MethodList>
```

## Placeholders

If you use placeholders in your strings, XenCenter can call different functions,
use different URLs, or provide different parameters based on what object is
selected in the resource list.

When an XML attribute is marked as being able to accept placeholders you can leave
wildcards for XenCenter to fill in based on the properties of the object that is
selected in the resource list.

| Placeholder              | Description                                                              |
|--------------------------|--------------------------------------------------------------------------|
| `{$type}`                  | The type of the selected object, for example, VM, Network                |
| `{$label}`                 | The label of the selected object                                         |
| `{$uuid}`                  | The UUID of the selected object, or the full pathname of a folder        |
| `{$description}`           | The description of the selected object                                   |
| `{$tags}`                  | Comma-separated list of the tags of the selected object                  |
| `{$host}`                  | The host name                                                            |
| `{$pool}`                  | The pool name                                                            |
| `{$networks}`              | Comma-separated list of the names of the networks attached to the object |
| `{$storage}`               | Comma-separated list of the names of the storage attached to the object  |
| `{$disks}`                 | Comma-separated list of the types of the storage attached to the object  |
| `{$memory}`                | The host memory, in bytes                                                |
| `{$os_name}`               | The name of the operating system that a VM is running                    |
| `{$power_state}`           | The VM power state, for example Halted, Running                          |
| `{$virtualisation_status}` | The state of the pure virtualization drivers installed on a VM           |
| `{$start_time}`            | Date and time that the VM was started                                    |
| `{$ha_restart_priority}`   | The HA restart priority of the VM                                        |
| `{$size}`                  | The size in bytes of the attached disks                                  |
| `{$ip_address}`            | Comma-separated list of IP addresses associated with the selected object |
| `{$uptime}`                | Uptime of the object, in a form such as '2 days, 1 hour, 26 minutes'     |
| `{$ha_enabled}`            | true if HA is enabled, false otherwise                                   |
| `{$shared}`                | Applicable to storage, true if storage is shared, false otherwise        |
| `{$vm}`                    | Comma-separated list of VM names                                         |
| `{$folder}`                | The immediate parent folder of the selected object                       |
| `{$folders}`               | Comma-separated list of all the ancestor folders of the selected object  |

If the user has selected more than one target for the plug-in (by multiselect or
by selecting a folder), it is not possible for XenCenter to know which object to
use for the placeholder context. In a multi-target scenario all placeholders are
substituted with the keyword `multi_target`. The plug-in can use this information
to detect this situation.

Also, if there has been an error filling in a particular placeholder then the
keyword `null` is substituted in to indicate the error. One example of where you
see this keyword is if the XenCenter node was selected, which has no object
properties to fill in.

**Example:** A community group adds their HTML help guides into a XenCenter tab
based on which object is selected

```xml
<XenCenterPlugin
      xmlns="http://www.citrix.com/XenCenter/Plugins/schema"
      version="1"
      plugin_version="1.0.0.0">

  <TabPage
        name="Extra Support"
        url="http://www.extra-help-for-xencenter.com/loadhelp.php?type={$type}" />

</XenCenterPlugin>
```
