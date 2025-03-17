### install Docker Desktop Win
Install from the command line

After downloading Docker Desktop Installer.exe

-   rename "Docker Desktop Installer.exe" to Docker_Desktop_Installer.exe

run the following command in a terminal to install Docker Desktop:

```shell
Docker_Desktop_Installer.exe install
```

If you’re using PowerShell you should run it as:

```shell
Start-Process Docker_Desktop_Installer -Wait install
```

If using the Windows Command Prompt:

```shell
start /w "" Docker_Desktop_Installer install
```

By default, Docker Desktop is installed at C:\Program Files\Docker\Docker.

The ``install`` command accepts the following flags:
-    ``--quiet``: Suppresses information output when running the installer

-    ``--accept-license``: Accepts the Docker Subscription Service Agreement now, rather than requiring it to be accepted when the application is first run

-   ``--no-windows-containers``: Disables the Windows containers integration. This can improve security. For more information, see Windows containers.

-   ``--backend=<backend name>``: Selects the default backend to use for Docker Desktop, ``hyper-v``, windows or ``wsl-2`` (default)

-   ``--installation-dir=<path>``: Changes the default installation location (C:\Program Files\Docker\Docker)

- ``--always-run-service``: After installation completes, starts com.docker.service and sets the service startup type to Automatic. This circumvents the need for administrator privileges, which are otherwise necessary to start com.docker.service. com.docker.service is required by Windows containers and Hyper-V backend.

#### Sample cmd shell
(Powershell did not work)
```shell
cd /d K:/MyDownloads
start /w "" Docker_Desktop_Installer install --installation-dir=D:/programs/docker --accept-license --no-windows-containers --always-run-service
```
Docker installation output:
```shell
CommandLine: "K:\MyDownloads\Docker_Desktop_Installer.exe" "install" -package "res:DockerDesktop" --installation-dir="D:\programs\docker" --accept-license --no-windows-containers --always-run-service --relaunch-as-admin
```
