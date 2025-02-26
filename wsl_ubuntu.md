## How to Install WSL and Ubuntu on Windows 10
### Install Windows Subsystem for Linux (WSL) on Windows
1. Open PowerShell as an administrator
2. ``Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux``
3. Restart the computer when prompted.

### Install Ubuntu
1.  Open the Microsoft Store (search for "store" from the start menu)
2.  Search the store for "Ubuntu"
3.  Install Ubuntu (it is not necessary to sign in to the store)
4.  Launch Ubuntu
5.  Enter a username. This will create a local user account and you will be automatically logged in to Ubuntu as this user.
6.  Enter a password for the user and enter a second time to confirm.
7.  Update all Ubuntu software packages with ``sudo apt update && sudo apt upgrade -y``
8.  setup development environment
```
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install gcc
sudo apt-get install cloc
sudo apt install net-tools
```
9. find home drive, sample:
C:\Users\MS\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_79rhkp1fndgsc\LocalState\rootfs\home\ms

10. subst
```
subst U: /D
subst U: C:\Users\MS\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu22.04LTS_79rhkp1fndgsc\LocalState\rootfs\home\ms
```
