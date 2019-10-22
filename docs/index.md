# Citrix XenCenter Plug-in Specification Guide

This document explains how to write a plug-in to Citrix XenCenter, the GUI for Citrix Hypervisor. Using the plug-in mechanism third-parties can:

-  Create menu entries in the XenCenter menus linked to an executable file or PowerShell script, including full use of the Citrix Hypervisor PowerShell Module (XenServerPSModule) cmdlets.

-  Cause a URL to be loaded into a tab in XenCenter.

The XenCenter plug-in mechanism is context aware, allowing you to use XenSearch to specify complicated queries. Also, plug-ins can take advantage of contextual information passed as arguments to executables or as replaceable parameters in URLs.

A XenCenter plug-in consists of the following components:

-  An XML configuration file.

-  A resource DLL for each supported locale. Currently XenCenter exists in English and Japanese versions only.

-  The application and any resources it requires.

Put these components of a plug-in in a subdirectory of the XenCenter installation directory. For example, a default installation of XenCenter requires that a plug-in reside in `C:\Program Files\Citrix\XenCenter\Plugins\<organization_name>\<plug-in_name>`

XenCenter loads all valid plug-ins found in subdirectories of the plug-ins directory when it starts:

-  The plug-in name (&lt;plug-in_name&gt;) must be the same as the directory in which it is placed.

-  The resource DLL and the XML configuration file must follow these naming conventions:

    -  `<plug-in_name>.resources.dll`

    -  `<plug-in_name>.xcplugin.xml`

For example, if your organization is called Citrix and you write a plug-in called Example which runs a batch file called `do_something.bat`, the following files must exist:

-  `C:\Program Files\Citrix\XenCenter\Plugins\Citrix\Example\Example.resources.dll`
-  `C:\Program Files\Citrix\XenCenter\Plugins\Citrix\Example\example.xcplugin.xml`
-  `C:\Program Files\Citrix\XenCenter\Plugins\Citrix\Example\do_something.bat`

These paths assume that you use the default XenCenter installation directory.