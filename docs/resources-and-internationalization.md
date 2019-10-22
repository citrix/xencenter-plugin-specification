# Resources and Internationalization

In the text below, *plug-in_name* is the value of the plug-in name attributes and *entry_name* is the value of the MenuItem/TabPage name attributes.

The following resources will be read from `<plug-in>.resources.dll`:

-  *plug-in_name*.description – Shown in the plug-ins dialog.
-  *plug-in_name*.copyright – Vendor copyright statement. Shown in the plug-ins dialog.
-  *plug-in_name*.link – Link to vendor's webpage. Shown in the plug-ins dialog.
-  *entry_name*.label – The menu entry label.
-  *entry_name*.description – Shown in the plug-ins dialog.
-  *entry_name*.icon – The icon to use in the menu entry. Should be a 16x16 PNG. May be omitted.
-  *entry_name*.tooltip – The tooltip to use when the menu entry is disabled. May be omitted.

To create the resources file:

1.  Create a RESX file containing the appropriate strings (this can be  done in any project in Visual Studio: e.g., console project).

1.  Open up Visual Studio cmd prompt and navigate to the resx's directory

1.  Run `ResGen.exe <plug-in_name>.resx`

    (`ResGen.exe` is found in `C:\Program Files\Microsoft SDKs\Windows\v7.0A\bin`).

1.  Run `al.exe /t:lib /embed:<plug-in_name>.resources /out:<plug-in_name>.resources.dll`

    (`al.exe` is also found in  `C:\Program Files\Microsoft SDKs\Windows\v7.0A\bin`).

1.  If you wish to compile additional resource DLLs for any specific cultures edit the resx file appropriately and run these commands again with an additional `/culture` argument in the `al.exe` command specifying the two letter culture string. For example, for Japanese, run: `al.exe /t:lib /embed:<plug-in_name>.resources /culture:ja /out:<plug-in_name>.resources.dll`

1.  Place the invariant culture `<plug-in_name>.resources.dll` into the `<plug-in_dir>` folder (with `<plug-in_name>.xcplugin.xml` etc)

1.  Other cultures should be as follows: `<plug-in_dir>\<culture>\<plug-in_name>.resources.dll` where `<culture>` is the two-letter ISO culture name

Unless stated otherwise, all of these entries are mandatory. If any are missing, then the problem will be logged, and the menu option will be disabled.