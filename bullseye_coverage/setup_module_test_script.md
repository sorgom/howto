# how to setup a script to run module tests with Bullseye coverage
Windows CMD sample

## preconditions
-   install MS build tools or Visual Studio
-   and then install Bullseye coverage

## sample project _DSTW98_
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
|   `-- tests
|       |-- moduletests
|       |-- moduletestsIL
|       `-- systemtests
|
|-- scripts
|   `-- coverage
|       |-- moduletests.cmd (our script)
|       `-- exclude.txt
|
|-- submodules
|   |-- CppUTestSteps
|   `-- cpputest
|
|-- vs
|   |-- DSTW.sln
|   `-- *.vcxporj
|
| temporary:
|-- build
|-- reports
```
## requirements
### general
#### development
-   Script must enable incremental test development without permanent clean builds.
    -   implement
    -   run script
    -   view results in coverage browser
-   instrumentation state should not be changed after script run
#### pipelines e.g. Jenkins
-   Return code of script must mirror if desired coverage reached.
    - 0 desired coverage reached
    - 1 otherwise
-  A minimal text based reporting to be saved as artifact might be provided:
    - coverage overview
    - todo report
### for this sample
-   code coverage required for
```
|-- specification
|-- application
|   `-- components
```
-   Two different test runs required for full coverage
    -  moduletests (with mocked interface locator)
    - moduletestsIL (to test the production interface locator)

## sample ms build solution _DSTW.sln_
- submodules.vcxproj - static lib
```
|-- submodules
|   |-- CppUTestSteps
|   `-- cpputest
```
- moduletests.vcxproj - console app using submodules.lib
```
|-- specification
|-- application
|   `-- components
|
|-- testing
|   |-- testenv
|   `-- tests
|       `-- moduletests
```
- moduletestsIL.vcxproj - console app using submodules.lib
```
|-- specification
|-- application
|   `-- components
|
|-- testing
|   |-- testenv
|   `-- tests
|       `-- moduletestsIL
```
## setting up the script step by step

### setup
- determine relevant directories
```cmd
@echo off
SETLOCAL
cd /d %~dp0
set myDir=%cd%
cd ../..
set repoDir=%cd%
set buildDir=%repoDir%\build
set reportsDir=%repoDir%\reports
set vsDir=%repoDir%\vs
set exeDir=%buildDir%\windows\bullseye
```
- solution and reporting files

```cmd
set vsSolution=%vsDir%\DSTW.sln
set report=%reportsDir%\moduletests_coverage.txt
set todoTxt=%reportsDir%\moduletests_todo.txt
```
-   Bullseye coverage file: %COVFILE%
    -   can have any name
    -   extension _.cov_ makes sense - since assigned to coverage browser
```cmd
set covfile=%buildDir%\moduletests.cov
```
- exclude file (which we don't have yet)
```cmd
set excludeFile=%myDir%\exclude.txt
```

- Bullseye coverage behavior %COVCOPT%
    - top level directory for coverage output
    - macro instrumentation activated
```cmd
set covcopt=--srcdir %repoDir% --macro
```
- desired coverage
(function,decision in %)
```cmd
set covMinima=100,100
```
- exit status
```cmd
set elevel=0
```
- msbuild call (sample)
    - build can be called with any configuration
    - separating instrumented and non instrumented build folders by configuration saves a lot of clean builds
    - see also: [premake5 sample](../premake5/separate_binaries.md)
```cmd
set vsCall=msbuild -m %vsSolution% -p:configuration=bullseye
```
-   provide temporary folders
-   remove old reports
```cmd
md %buildDir% %reportsDir% >NUL 2>&1
DEL /Q %report% %todoTxt% >NUL 2>&1
```
- save current Bullseye instrumentation state (to restore at script end)
```cmd
cov01 -q --push
```
### clean
reasons:
- coverage file does not exist
- command line option -c
```cmd
set clean=0
if not exist %covfile% set clean=1
if "%1" == "-c" set clean=1

if %clean% == 1 (
    DEL /Q %covfile% >NUL 2>&1
    %vsCall% -t:Clean
)
```
### build
#### without coverage
(see [remarks](##remarks))
-   deactivate coverage instrumentation
-   call ms build
 ```cmd
cov01 -q --off
%vsCall% -t:submodules
if %errorlevel% NEQ 0 goto err
```
#### coverage instrumented
-   activate coverage instrumentation
-   call ms build
```cmd
cov01 -q --on
%vsCall% -t:"moduletests,moduletestsIL"
if %errorlevel% NEQ 0 goto err
```
- check if coverage file has been built (not really necessary)
```cmd
if not exist %covfile% (
    echo %covfile% not found
    goto err
)
```
### run
- rewind coverage file (it might not be new)
```cmd
covclear -q
```
- run module tests executables
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
-   change directory to coverage file location (see [remarks](##remarks))
```cmd
cd %buildDir%
```
-   write report
    -   in this sample: _covdir_ for directory wise coverage output
    -   you might as well use:
        - _covsrc_ for source wise output
        - _covclass_ for class wise output
```cmd
covdir -q --by-name > %report%
type %report%
```
-   apply coverage minima reached check and save return for script exit
```cmd
covdir -q --checkmin %covMinima%
set elevel=%errorlevel%
```
-   if not passed: write todo report using _covbr_
```cmd
if %elevel% NEQ 0 covbr -qu > %todoTxt%
```
-   restore previous instrumentation state
-   exit with coverage passed value
```cmd
:end
cov01 -q --pop
exit /b %elevel%
```
-   error jump mark
```cmd
:err
set elevel=1
goto end
```
## 1st run
run script
- script will show error: missing exclude file
- report contains everything
```shell
Exception: cannot open 'c:\git\DSTW98\scripts\coverage\exclude.txt': No such file or directory
Directory                                               Function Coverage        C/D Coverage
-----------------------------------------------------  ------------------  ------------------
application/components/                                 184 /  184 = 100%   317 /  317 = 100%
...
specification/                                           10 /   10 = 100%     0 /    0
...
submodules/                                             369 / 1616 =  22%   346 / 2320 =  14%
...
testing/                                                334 /  367 =  91%   338 / 1093 =  30%
-----------------------------------------------------  ------------------  ------------------
Total                                                   897 / 2177 =  41%  1001 / 3730 =  26%
```

## setup exclude file with coverage browser
### open coverage file in coverage browser

![open in browser](01_open.png)

![coverage](02_coverage.png)
### exclude regions of no interest from coverage
- (right click, context menu)

![exclude](03_exclude_1.png)

![excluded](04_excluded.png)
### export to exclude file
- File, Export Exclusions...

![export...](05_export.png)

![save](06_save.png)

## 2nd run
run script
- script should not show an error
- report contains desired regions

```shell
Directory                        Function Coverage      C/D Coverage
-------------------------------  -----------------  ----------------
application/components/           184 / 184 = 100%  317 / 317 = 100%
application/components/BAS/        43 /  43 = 100%   32 /  32 = 100%
application/components/BAS/src/     7 /   7 = 100%    0 /   0
application/components/COM/        51 /  51 = 100%   70 /  70 = 100%
application/components/COM/src/    40 /  40 = 100%   70 /  70 = 100%
application/components/LCR/        12 /  12 = 100%   37 /  37 = 100%
application/components/LCR/src/     7 /   7 = 100%   37 /  37 = 100%
application/components/SIG/        26 /  26 = 100%   91 /  91 = 100%
application/components/SIG/src/    19 /  19 = 100%   91 /  91 = 100%
application/components/SYS/        44 /  44 = 100%   65 /  65 = 100%
application/components/SYS/src/    16 /  16 = 100%   59 /  59 = 100%
application/components/TSW/         8 /   8 = 100%   22 /  22 = 100%
application/components/TSW/src/     6 /   6 = 100%   22 /  22 = 100%
specification/                      8 /   8 = 100%    0 /   0
specification/codebase/             2 /   2 = 100%    0 /   0
specification/ifs/                  6 /   6 = 100%    0 /   0
-------------------------------  -----------------  ----------------
Total                             192 / 192 = 100%  317 / 317 = 100%
```
## remarks
### decent relative paths output
Bullseye output is a bit tricky to handle.

To achieve decent paths output with covdir or covsrc there are two options.
1) change dir to coverage file location (as in our script)
```cmd
cd %buildDir%
covdir -q --by-name > %report%
```
2) or change dir to application root (as set with %COVCOPT%) and use _--srcdir ._
```cmd
cd %repoDir%
covdir -q --by-name --srcdir . > %report%
```
### exclude regions from instrumentation during build

There is no actual need to exclude parts from instrumented build.

- What you can safely exclude:
    - third party sources (like CppUTest) that you have linked as git submodules

- What you should not exclude:
    - your own test environment - you might finally want to check what was really used
### HTML reports from CI pipelines
There is no reason to generate and save HTML reports cause no one reads them.

Required information:
- coverage passed / failed
- a basic overview (covdir / covsrc)
- missing coverage (covbr)

For test development you only need the coverage browser.

### appendix: the files
-   the script
```cmd
@echo off
SETLOCAL
cd /d %~dp0
set myDir=%cd%
cd ../..
set repoDir=%cd%
set buildDir=%repoDir%\build
set reportsDir=%repoDir%\reports
set vsDir=%repoDir%\vs
set exeDir=%buildDir%\windows\bullseye

set vsSolution=%vsDir%\DSTW.sln
set report=%reportsDir%\moduletests_coverage.txt
set todoTxt=%reportsDir%\moduletests_todo.txt
set covfile=%buildDir%\moduletests.cov

set covcopt=--srcdir %repoDir% --macro
set excludeFile=%myDir%\exclude.txt
set covMinima=100,100

set vsCall=msbuild -m %vsSolution% -p:configuration=bullseye

set elevel=0

md %buildDir% %reportsDir% >NUL 2>&1
DEL /Q %report% %todoTxt% >NUL 2>&1

cov01 -q --push

set clean=0
if not exist %covfile% set clean=1
if "%1" == "-c" set clean=1
if %clean% == 1 (
    %vsCall% -t:Clean
    DEL /Q %covfile% >NUL 2>&1
)

cov01 -q --off
%vsCall% -t:submodules
if %errorlevel% NEQ 0 goto err

cov01 -q --on
%vsCall% -t:"moduletests,moduletestsIL"
if %errorlevel% NEQ 0 goto err

if not exist %covfile% (
    echo %covfile% not found
    goto err
)

covclear -q
for %%t in (moduletests moduletestsIL) do (
    %exeDir%\%%t.exe
    if %errorlevel% NEQ 0 goto err
)

covselect -qd --import %excludeFile%

cd %buildDir%
covdir -q --by-name > %report%
type %report%

covdir -q --checkmin %covMinima%
set elevel=%errorlevel%
if %elevel% NEQ 0 covbr -qu -f %covfile% > %todoTxt%

:end
cov01 -q --pop
exit /b %elevel%

:err
set elevel=1
goto end
```
-   the exclude file
```
exclude folder submodules/
exclude folder testing/
```
or:
```
exclude all /
include folder application/
include folder specification/
```
