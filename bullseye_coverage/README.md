# how to setup a script to run Bullseye coverage
Windows CMD sample

## preconditions
-   install MS build tools or Visual Studio
-   and then install Bullseye coverage

## sample project DSTW
```
repo
|-- specification
|-- application
|   |-- components
|   `-- main
|       `-- AppMain.cpp
|
|-- testing
|   |-- testenv
|   |-- tests
|   |   |-- moduletests
|   |   |-- moduletestsIL
|   |   `-- systemtests
|   `-- testmain
|       `-- testMain.cpp
|
|-- make
|   |-- coverage
|   |   `-- coverage.cmd
|
|-- submodules
|   |-- CppUTestSteps
|   `-- cpputest
|
| temporary:
|-- build
|-- reports
```
code coverage required for
```
|-- specification
|-- application
|   `-- components
```


## ms build solutions
### variation A: all in
moduletests.vcxproj - console app
```
|-- specification
|-- application
|   `-- components
|
|-- testing
|   |-- testenv
|   |-- tests
|   |   `-- moduletests
|   `-- testmain
|       `-- testMain.cpp
|
|-- submodules
|   |-- CppUTestSteps
|   `-- cpputest
```
moduletestsIL.vcxproj - console app
```
|-- specification
|-- application
|   `-- components
|
|-- testing
|   |-- testenv
|   |-- tests
|   |   `-- moduletestsIL
|   `-- testmain
|       `-- testMain.cpp
|
|-- submodules
|   |-- CppUTestSteps
|   `-- cpputest
```
### variation B: separation of testenv and tests
testenv.vcxproj - static lib
```
|-- testing
|   `-- testenv
|
|-- submodules
|   |-- CppUTestSteps
|   `-- cpputest
```
moduletests.vcxproj - console app
```
|-- specification
|-- application
|   `-- components
|
|-- testing
|   |-- tests
|   |   `-- moduletests
|   `-- testmain
|       `-- testMain.cpp
```
moduletestsIL.vcxproj - console app
```
|-- specification
|-- application
|   `-- components
|
|-- testing
|   |-- tests
|   |   `-- moduletestsIL
|   `-- testmain
|       `-- testMain.cpp
```
If you are not sure about what can be safely excluded from instrumentation - choose: all in.
## setting up thew script
The script shall use variation B solution. 
### setup
- determine relevant directories
```cmd
@echo off
SETLOCAL
cd /d %~dp0
set myDir=%cd%
cd ..
set makeDir=%cd%
cd ..
set repoDir=%cd%
set buildDir=%cd%\build
set reportsDir=%cd%\reports
set exeDir=%buildDir%\windows\bullseye
```
- solution and reporting files

```cmd
set vsSolution=%makeDir%\DSTW.sln
set report=%reportsDir%\coverage.txt
set todoTxt=%reportsDir%\todo.txt
```
- coverage file: %COVFILE% 
```cmd
set covfile=%buildDir%\coverage.cov
```
- exclude file (which we don't have yet) 
```cmd
set excludeFile=%myDir%\exclude.txt
```

- coverage behavior %COVCOPT%
    - top level directory for coverage output
    - macro instrumentation on
```cmd
set covcopt=--srcdir %repoDir% --macro
```
- desired coverage
function,decision in %
```cmd
set covMin=100,100
```
- exit status
```cmd
set elevel=0
```
- msbuild call
    - could be called with any configuration
    - separating instrumented and non instrumented builds saves a lot of clean builds
```cmd
set vsCall=msbuild -m %vsSolution% -p:configuration=bullseye
```
- provide temporary folders
```cmd
md %buildDir% %reportsDir% >NUL 2>&1
```
- save current instrumentation state
```cmd
cov01 -q --push
```
### optional clean
```cmd
if "%1" == "-c" (
    %vsCall% -t:Clean
    DEL /Q %covfile% >NUL 2>&1
)
```
### build
- without instrumentation
```cmd
cov01 -q --off
%vsCall% -t:testenv
if %errorlevel% NEQ 0 goto err 
```
- with instrumentation
```cmd
cov01 -q --on
%vsCall% -t:"moduletests,moduletestsIL"
if %errorlevel% NEQ 0 goto err
```
- check if coverage file has been built
(not really necessary)
```cmd
if not exist %covfile% (
    echo %covfile% not found
    goto err
)
```
### run
- rewind coverage file if it was not removed before
```cmd
covclear -q
```
- run moduletests executables
```cmd
for %%t in (moduletests moduletestsIL) do (
    %exeDir%\%%t.exe
    if %errorlevel% NEQ 0 goto err
)
```
### report
-   apply exclude file (which we don't have at 1st run)
```cmd
covselect -qd --import %excludeFile%
```
-   change directory to coverage file location 
```cmd
cd %buildDir%
```
-   write report (directory coverage in this sample)
```cmd
covdir -q --by-name > %report%
type %report%
```
-   apply coverage minima reached check
```cmd
covdir -q --checkmin %covMin% -f %covfile%
```
-   write todo output if not passed
```cmd
if %errorlevel% NEQ 0 covbr -qu > %todoTxt%
```
-   restore previous instrumentation state
-   exit value
```cmd
:end
cov01 -q --pop
exit /b %elevel%

:err
set elevel=1
goto end
```
```cmd
```
