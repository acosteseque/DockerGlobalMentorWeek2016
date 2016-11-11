# Docker Global Mentor Week, Windows Containers Labs Sommaire

  * [Prerequis](#chapter-0)
  * [Install CS Docker Engine sur Windows Server 2016](#chapter-1)
  * [Verifications](#chapter-2)
  * [Pull Images](#chapter-3)
  * [Windows Server Container Hello World](#chapter-4)
  * [Le contenu d'un Windows Server Container](#chapter-5)
  * [Hyper-V Container Hello World](#chapter-6)
  * [Utiliser un Dockerfile](#chapter-7)
  * [Se connecter à un Windows Container en cours d'execution](#chapter-8)
  * [Publier son Container sur une Registry](#chapter-9)
  * [Faire le menage](#chapter-10)
  * [Multi Container App avec Docker Compose](#chapter-11)
  * [Bonus si Docker Host dans une VM sur le poste de travail](#chapter-12)
  * [Terminology](#chapter-13)
  * [Ressources](#chapter-14)

# Prerequis <a id="chapter-0"></a>

[Avant de venir à la Docker Global Mentor Week](DockerOnWindows_Prerequis.md)

# Install CS Docker Engine sur Windows Server 2016 <a id="chapter-1"></a>

Le deploiement automatisé du CS Docker Engine se fait à l'aide de OneGet par le package docker fourni avec le module DockerMsftProvider de la PSGallery

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
# Find-Package –providerName DockerMsftProvider
# Find-Package –providerName DockerMsftProvider –AllVersions
Install-Package -Name docker -ProviderName DockerMsftProvider
# Find-Package –ProviderName DockerMsftProvider | Install-Package -Verbose
Restart-Computer -Force
Get-Package –ProviderName DockerMsftProvider | ft -AutoSize
# Pour les Hyper-V Container
Install-WindowsFeature hyper-v
Restart-Computer -Force
```

## Update

```
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Verbose
```

## Uninstall

```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider -Verbose
```

# Verifications <a id="chapter-2"></a>

```
docker version
docker info
docker ps
docker images
```

# Pull Images <a id="chapter-3"></a>

```
docker pull microsoft/windowsservercore
docker pull microsoft/nanoserver
```

# Windows Server Container Hello World <a id="chapter-4"></a>

```
docker run --rm microsoft/windowsservercore cmd /c echo Hello`, from DockerGlobalMentorWeek2016!
docker run --rm microsoft/nanoserver cmd /c echo Hello`, from DockerGlobalMentorWeek2016!
docker run --rm microsoft/sample-dotnet
```

# Le contenu d'un Windows Server Container <a id="chapter-5"></a>

```
docker run --rm -it microsoft/windowsservercore powershell
Get-Process
Get-Service | measure
Get-ChildItem C:\Windows\System32\ | measure
exit
docker run --rm -it microsoft/nanoserver powershell
Get-Process
Get-Service | measure
Get-ChildItem C:\Windows\System32\ | measure
exit
```

# Hyper-V Container Hello World <a id="chapter-6"></a>

```
docker run --rm --isolation=hyperv microsoft/nanoserver cmd /c echo Help`, I`'m trapped in a VM!
# Le refaire (doit etre plus rapide)
docker run --rm -it --isolation=hyperv microsoft/nanoserver cmd
powershell get-process
# Voir taskmgr sur le Docker Host (pas de smss)
```

# Utiliser un Dockerfile <a id="chapter-7"></a>

```
powershell new-item c:\DockerGlobalMentorWeek2016\Dockerfile -Force
notepad c:\DockerGlobalMentorWeek2016\Dockerfile
FROM microsoft/iis
RUN echo "Hello DockerGlobalMentorWeek2016! - Dockerfile" > c:\inetpub\wwwroot\index.html
docker build -t <user>/DGMW-iis-dockerfile c:\DockerGlobalMentorWeek2016
docker run -d -p 8080:80 <user>/DGMW-iis-dockerfile ping -t localhost
docker ps
docker rm -f <container name>
```

# Se connecter à un Windows Container en cours d'execution <a id="chapter-8"></a>

```
docker ps
docker exec -it <ID> cmd
docker exec -it <ID> powershell
```

# Publier son Container sur une Registry (ex : DockerHub) <a id="chapter-9"></a>

```
docker login
docker push <user>/DGMW-iis-dockerfile
docker rmi -f <user>/DGMW-iis-dockerfile
docker pull <user>/DGMW-iis-dockerfile
```

# Faire le menage <a id="chapter-10"></a>

```
# Sup d'un container
docker rm -f <container name>
# Sup de tous les containers créés
docker rm -f $(docker ps -aq)
# Sup d'une image
docker rmi -f <image name>
# Attention Sup de toutes les images
docker rmi -f $(docker images -aq)
```

# Multi Container App avec Docker Compose <a id="chapter-11"></a>

## Install Docker Compose

```
Invoke-WebRequest https://dl.bintray.com/docker-compose/master/docker-compose-Windows-x86_64.exe -UseBasicParsing -OutFile $env:ProgramFiles\docker\docker-compose.exe
docker-compose version
```

## Install git

```
Set-ExecutionPolicy RemoteSigned -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
choco install git -params '"/GitAndUnixToolsOnPath"'
choco list --local-only
```

## Deploiement ASP.NET Core MVC MusicStore app

```
cd c:\DockerGlobalMentorWeek2016
git clone https://github.com/friism/Musicstore
cd Musicstore
docker-compose -f .\docker-compose.windows.yml build
docker-compose -f .\docker-compose.windows.yml up

docker ps
docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" musicstore_web_1
<IP>:5000
```

# Bonus si Docker Host dans une VM sur le poste de travail <a id="chapter-12"></a>

## Sur le Docker Daemon (Host)

```
# Open firewall port 2375
netsh advfirewall firewall add rule name="docker engine" dir=in action=allow protocol=TCP localport=2375
# Configure Docker daemon to listen on both pipe and TCP (replaces docker --register-service invocation above)
Stop-Service docker
dockerd --unregister-service
dockerd -H npipe:// -H 0.0.0.0:2375 --register-service
Start-Service docker
```

## Sur le Docker Client

```
$env:DOCKER_HOST = "<ip-address-de-la-vm>:2375"
docker version
```

# Terminology <a id="chapter-13"></a>

Docker-specific jargon which might be confusing to some. So before you go further, let's clarify some terminology that is used frequently in the Docker ecosystem.

* *Images* - The file system and configuration of our application which are used to create containers. To find out more about a Docker image, run `docker inspect alpine`. In the demo above, you used the `docker pull` command to download the **alpine** image. When you executed the command `docker run hello-world`, it also did a `docker pull` behind the scenes to download the **hello-world** image.
* *Containers* - Running instances of Docker images — containers run the actual applications. A container includes an application and all of its dependencies. It shares the kernel with other containers, and runs as an isolated process in user space on the host OS. You created a container using `docker run` which you did using the alpine image that you downloaded. A list of running containers can be seen using the `docker ps` command.
* *Docker daemon* - The background service running on the host that manages building, running and distributing Docker containers.
* *Docker client* - The command line tool that allows the user to interact with the Docker daemon.
* *Docker Hub* - A [registry](https://hub.docker.com/explore/ "DockerHub") of Docker images. You can think of the registry as a directory of all available Docker images. You'll be using this later in this tutorial.

# Ressources <a id="chapter-14"></a>

* [[Blog] Docker & Microsoft](http://www.docker.com/microsoft)
* [[Doc] Windows Containers](http://aka.ms/containers)
* [[Doc] Windows Containers Deployment](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/deployment/deployment)
* [[Tool] OneGet MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider)

# Have Fun with Windows Containers & Docker ! :metal:

![Dojocat](http://octodex.github.com/images/dojocat.jpg)