## 目录

- [我的配置文件](#我的配置文件)
- [参考](#参考)

## 我的配置文件

```
// This file was initially generated by Windows Terminal Preview 1.4.2652.0
// It should still be usable in newer versions, but newer versions might have additional
// settings, help text, or changes that you will not see unless you clear this file
// and let us generate a new one for you.

// To view the default settings, hold "alt" while clicking on the "Settings" button.
// For documentation on these settings, see: https://aka.ms/terminal-documentation
{
    "$schema": "https://aka.ms/terminal-profiles-schema",

    "defaultProfile": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",

    // You can add more global application settings here.
    // To learn more about global settings, visit https://aka.ms/terminal-global-settings

    // If enabled, selections are automatically copied to your clipboard.
    "copyOnSelect": true,

    // If enabled, formatted data is also copied to your clipboard
    "copyFormatting": false,

    // Size
    "initialCols" : 130,
    "initialRows" : 33,

    // A profile specifies a command to execute paired with information about how it should look and feel.
    // Each one of them will appear in the 'New Tab' dropdown,
    //   and can be invoked from the commandline with `wt.exe -p xxx`
    // To learn more about profiles, visit https://aka.ms/terminal-profile-settings
    "profiles":
    {
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.
            "fontFace": "Consolas",
            "colorScheme": "WarmNeon"
        },
        "list":
        [
            {
                // Make changes here to the powershell.exe profile.
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false
            },
            {
                // Make changes here to the cmd.exe profile.
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "name": "命令提示符",
                "commandline": "cmd.exe",
                "hidden": false
            },
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": false,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            },
            {
                // Make changes here to the cmd.exe profile.
                "guid": "{06f1f22a-c2d6-826d-71b9-c9bc81fb79c9}",
                "name": "Node",
                "commandline": "E:\\Nodejs\\node.exe",
                "hidden": false,
                "icon": ""
            }
        ]
    },

    // Add custom color schemes to this array.
    // To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
    "schemes": [
        {
        "name": "WarmNeon",
        "black": "#000000",
        "red": "#e24346",
        "green": "#39b13a",
        "yellow": "#dae145",
        "blue": "#4261c5",
        "purple": "#f920fb",
        "cyan": "#2abbd4",
        "white": "#d0b8a3",
        "brightBlack": "#fefcfc",
        "brightRed": "#e97071",
        "brightGreen": "#9cc090",
        "brightYellow": "#ddda7a",
        "brightBlue": "#7b91d6",
        "brightPurple": "#f674ba",
        "brightCyan": "#5ed1e5",
        "brightWhite": "#d8c8bb",
        "background": "#404040",
        "foreground": "#afdab6"
      }
    ],

    // Add custom actions and keybindings to this array.
    // To unbind a key combination from your defaults.json, set the command to "unbound".
    // To learn more about actions and keybindings, visit https://aka.ms/terminal-keybindings
    "actions":
    [
        // Copy and paste are bound to Ctrl+Shift+C and Ctrl+Shift+V in your defaults.json.
        // These two lines additionally bind them to Ctrl+C and Ctrl+V.
        // To learn more about selection, visit https://aka.ms/terminal-selection
        { "command": {"action": "copy", "singleLine": false }, "keys": "ctrl+c" },
        { "command": "paste", "keys": "ctrl+v" },

        // Press Ctrl+Shift+F to open the search box
        { "command": "find", "keys": "ctrl+shift+f" },

        // Press Alt+Shift+D to open a new pane.
        // - "split": "auto" makes this pane open in the direction that provides the most surface area.
        // - "splitMode": "duplicate" makes the new pane use the focused pane's profile.
        // To learn more about panes, visit https://aka.ms/terminal-panes
        { "command": { "action": "splitPane", "split": "auto", "splitMode": "duplicate" }, "keys": "alt+shift+d" }
    ]
}
```

## 参考

- [Windows Terminal 安装及美化](https://www.cnblogs.com/KiraYoshikage/p/11443741.html)
