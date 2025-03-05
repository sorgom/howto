# VS code

## integrate VS developer prompt
### preconditions
MSVC build tools or Visual Studio installed
#### hint
If you have not yet installed yet - think of the paths!
- Default paths like "C:\Program Files\Microsoft Visual Studio" are a pain in the as to handle.
- Choose a path without blanks like "C:\apps\VisualStudio".

### setup
- press CTRL+SHIFT+P for command palette
- select "Preferences: Open User Settings (JSON)"
- you'l get something like this in the editor:
```json
{
    "explorer.confirmDelete": false,
    "explorer.copyRelativePathSeparator": "/",
    "cSpell.checkVSCodeSystemFiles": true,
    "json.schemas": []
}
```
- add entry for CMD terminal with VS developer environment
```json
    "terminal.integrated.profiles.windows": {
        "Developer Command Prompt": {
        "path": [
            "${env:windir}\\System32\\cmd.exe"
        ],
        "args": [
            "/k",
            "<PATH_TO VsDevCmd.bat>"
        ],
        "icon": "terminal-cmd"
        }
    }
```
- ``<PATH_TO VsDevCmd.bat>`` depends on your Visual Studio or tools installation.
- e.g. ``C:\\Program^ Files\\Microsoft^ Visual^ Studio\\2022\\Enterprise\\Common7\\Tools\\VsDevCmd.bat"``
- I decided to install Visual Studio as ``C:/apps/VisualStudio`` to get rid of the annoying path blanks
- VS code can also handle slashes instead of backslashes
- so ``<PATH_TO VsDevCmd.bat>`` is ``C:/apps/VisualStudio/Common7/Tools/VsDevCmd.bat`` in my case

The settings file now looks like this:
```json
{
    "explorer.confirmDelete": false,
    "explorer.copyRelativePathSeparator": "/",
    "cSpell.checkVSCodeSystemFiles": true,
    "json.schemas": [],
    "terminal.integrated.profiles.windows": {
        "Developer Command Prompt": {
        "path": [
            "${env:windir}/System32/cmd.exe"
        ],
        "args": [
            "/k",
            "C:/apps/VisualStudio/Common7/Tools/VsDevCmd.bat"
        ],
        "icon": "terminal-cmd"
        }
    }
}
```
- save the file.
### usage
- press CTRL+SHIFT+P for command palette
- select "Create New Terminal (With Profile)"
- select folder to run in
- then select profile "Developer Command Prompt" or however you named it

