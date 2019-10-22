# Resources and Internationalization

In the following text, `<plug-in_name>` is the value of the plug-in name attributes
and `<entry_name>` is the value of the MenuItem/TabPage name attributes.

XenCenter reads the following resources from `<plug-in>.resources.dll`:

- `<plug-in_name>.description` - Shown in the plug-ins dialog.
- `<plug-in_name>.copyright` - Vendor copyright statement. Shown in the plug-ins
dialog.
- `<plug-in_name>.link` - Link to vendor's webpage. Shown in the plug-ins dialog.
- `<entry_name>.label` - The menu entry label.
- `<entry_name>.description` - Shown in the plug-ins dialog.
- `<entry_name>.icon` - The icon to use in the menu entry. This icon is a 16x16
PNG. Can be omitted.
- `<entry_name>.tooltip` - The tooltip to use when the menu entry is disabled.
Can be omitted.

To create the resources file:

1. Create a RESX file containing the appropriate strings (You can do this task
in any project in Visual Studio, for example, console project).

1. Open up Visual Studio command prompt and navigate to the RESX file's directory

1. Run `ResGen.exe <plug-in_name>.resx`

    (`ResGen.exe` is found in `C:\Program Files\Microsoft SDKs\Windows\v7.0A\bin`).

1. Run

    `al.exe /t:lib /embed:<plug-in_name>.resources /out:<plug-in_name>.resources.dll`

    (`al.exe` is also found in  `C:\Program Files\Microsoft SDKs\Windows\v7.0A\bin`).

1. If you want to compile extra resource DLLs for any specific cultures, edit the
RESX file appropriately. Run these commands again with an extra `/culture` argument
in the `al.exe` command specifying the two letter culture string. For example,
for Japanese, run:

    `al.exe /t:lib /embed:<plug-in_name>.resources /culture:ja /out:<plug-in_name>.resources.dll`

1. Place the invariant culture `<plug-in_name>.resources.dll` into the
`<plug-in_dir>` folder (with `<plug-in_name>.xcplugin.xml` and so on). Set up
other cultures as follows: `<plug-in_dir>\<culture>\<plug-in_name>.resources.dll`
where `<culture>` is the two-letter ISO culture name.

Unless stated otherwise, all of these entries are mandatory. If any are missing,
then the problem is logged, and the menu option is disabled.
