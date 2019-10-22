# Deploying

Each plug-in is installed into: `<XenCenter_install_dir>\Plugins\<organization_name>\<plug-in_name>`.

> **Notes:**
>
> -  The author's installation directory (`<XenCenter_install_dir>\Plugins\<organization_name>`) is known in this specification as `<org_root>`.
> -  `<org_root>` is read-only at runtime.
> -  By default `<XenCenter_install_dir>` is `C:\Program Files\Citrix\XenCenter`.
> -  `<organization_name>` is the name of the organization or individual authoring the plug-in.
> -  `<plug-in_name>` is the name of the plug-in.
> -  `<XenCenter_install_dir>` can be looked up in the registry. In a default XenCenter installation, the XenCenter install directory can be found at `HKEY_CURRENT_USER\SOFTWARE\Citrix\XenCenter\InstallDir`. If XenCenter was installed for all users, this key is under `HKEY_LOCAL_MACHINE`.

In `<org_root>\<plug-in_name>`, ensure that the following items exist:

-  A plug-in declaration file, named `<plug-in_name>.xcplugin.xml`.
-  A satellite assembly for resources, named `<plug-in_name>.resources.dll`.
-  Anything else that the plug-in needs for its proper functions.

Other than as specified in this document, XenCenter ignores the contents of `<org_root>`. Plug-in authors can install whatever content they require within their own `<org_root>`. You can, for example, have libraries or graphics that are shared between all plug-ins. In which case `<org_root>` (or a shared subdirectory of it) is a good place for these artifacts.

The same freedom applies to `<org_root>\<plug-in_name>`. Material in there that XenCenter does not recognize is ignored.

The `<XenCenter_install_dir>\Plugins` directory is scanned when XenCenter starts and plug-ins that are found are loaded. To rescan the directory, restart XenCenter or use the **Re-Scan Plug-in Directory** button on the **XenCenter plug-ins** dialog. Any subdirectory that does not contain  `<plug-in_name>.xcplugin.xml` is silently ignored.
