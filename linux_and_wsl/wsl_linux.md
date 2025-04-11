# Install Linux on Windows
## Enable Windows Subsystem for Linux (WSL) on Windows 10
1. Open PowerShell as an administrator
2. ``Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux``
3. Restart the computer when prompted.

## Install Linux
1.  Open the Microsoft Store (search for "store" from the start menu)
2.  Search the store for "Ubuntu" or "Debian"
3.  Install Linux (it is not necessary to sign in to the store)
4.  Launch the Linux
5.  Enter a username. This will create a local user account and you will be automatically logged in to Ubuntu as this user.
6.  Enter a password for the user and enter a second time to confirm.
## Setup development environment
Update all software packages and setup development environment
-   change user to root
```
sudo su -
```
-   update all software packages
```
apt update && sudo apt upgrade -y
```
- save typing with function:
```
_install() { apt-get -y --no-install-recommends install "$@"; }
```
- install
```
_install make g++ uuid-dev
_install cloc valgrind cppcheck net-tools
_install python3 python3-pip
_install git ssh
_install vim tree
```
- upgrade python
```
apt-get upgrade -y python3
```
# Remove Linux from Windows
## De-install Linux
- launch _settings - apps - installed apps_ and de-install
## Remove distro from wsl
Note: de-installation of the Linux does not remove the wsl virtual drive data of the distro (approx 7 GB):
- open a shell (command prompt)
- list distros: _wsl --list --all_ or _wsl -l_
```
C:\Users\MS>wsl --list --all
Windows Subsystem für Linux-Distributionen:
Ubuntu-22.04 (Standard)
docker-desktop
Debian
```
- remove distro: _wsl --unregister_ distro
```
C:\Users\MS>wsl --unregister Ubuntu-22.04
Registrierung wird aufgehoben.
Der Vorgang wurde erfolgreich beendet.
```
- check success
```
C:\Users\MS>wsl -l
Windows Subsystem für Linux-Distributionen:
docker-desktop (Standard)
Debian
```
- change default if desired: _wsl --setdefault_ distro
```
wsl --setdefault Debian
Der Vorgang wurde erfolgreich beendet.

C:\Users\MS>wsl -l
Windows Subsystem für Linux-Distributionen:
Debian (Standard)
docker-desktop
```
