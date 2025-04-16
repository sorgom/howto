# separate binary output by configuration and target OS
Using premake5 to generate the solutions separation can be achieved like this.
-   C++ project with several configurations
```lua
workspace 'MyProject'
    configurations { 'ci', 'debug', 'bullseye' }
    language 'C++'
```
-   objects output folder
    -   ``<build folder>/<target OS>/obj/<config>`` 
    -   (premake adds config to paths automatically)
```lua
    objdir  '../build/%{_TARGET_OS}/obj'
```
-   library search folder
    -   ``<build folder>/<target OS>/lib/<config>`` 
```lua
    libdirs { '../build/%{_TARGET_OS}/lib/%{cfg.name}' }
```
-   target folder for executables
    -   ``<build folder>/<target OS>/<config>``
```lua
    filter { 'kind:ConsoleApp' }
        targetdir '../build/%{_TARGET_OS}/%{cfg.name}'
```
-   target folder for libraries (same as search folder)
    -   ``<build folder>/<target OS>/lib/<config>``
```lua
    filter { 'kind:StaticLib' }
        targetdir '../build/%{_TARGET_OS}/lib/%{cfg.name}'
```
