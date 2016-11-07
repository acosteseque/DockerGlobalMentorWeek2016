# Install CS Docker Engine sur Windows Server 2016

Ressources :

https://msdn.microsoft.com/en-us/virtualization/windowscontainers/deployment/deployment

https://github.com/OneGet/MicrosoftDockerProvider

Le deploiement automatisé du CS Docker Engine se fait à l'aide de OneGet par le package docker fourni avec le module DockerMsftProvider de la PSGallery

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

## Update

    Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Verbose

## Uninstall

    Uninstall-Package -Name docker -ProviderName DockerMsftProvider -Verbose

# Verifications

    docker version
    docker info
    docker ps
    docker images

# Pull Images

    docker pull microsoft/windowsservercore
    docker pull microsoft/nanoserver

# Windows Server Container Hello World

    docker run --rm microsoft/windowsservercore powershell /c echo Hello GlobalMentorWeek2016!
    docker run --rm microsoft/nanoserver powershell /c echo Hello GlobalMentorWeek2016!
    docker run --rm microsoft/sample-dotnet

# Le contenu d'un Windows Server Container

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

# Hyper-V Container Hello World

    docker run --rm --isolation=hyperv microsoft/nanoserver cmd /c echo Help`, I`'m trapped in a VM!
    # Le refaire (doit etre plus rapide)
    docker run --rm -it --isolation=hyperv microsoft/nanoserver cmd
    powershell get-process
    # Voir taskmgr sur le Docker Host (pas de smss)

# Utiliser un Dockerfile

    powershell new-item c:\GlobalMentorWeek2016\Dockerfile -Force
    notepad c:\GlobalMentorWeek2016\Dockerfile
    FROM microsoft/iis
    RUN echo "Hello GlobalMentorWeek2016! - Dockerfile" > c:\inetpub\wwwroot\index.html
    docker build -t <user>/DGMW-iis-dockerfile c:\GlobalMentorWeek2016
    docker run -d -p 8080:80 <user>/DGMW-iis-dockerfile ping -t localhost
    docker ps
    docker rm -f <container name>

# Publier son Container sur DockerHub

    docker login
    docker push <user>/DGMW-iis-dockerfile
    docker rmi -f <user>/DGMW-iis-dockerfile
    docker pull <user>/DGMW-iis-dockerfile

# Faire le menage

    # Sup d'un container
    docker rm -f <container name>
    # Sup de tous les containers créés
    docker rm -f $(docker ps -aq)
    # Sup d'une image
    docker rmi -f <image name>
    # Attention Sup de toutes les images
    docker rmi -f $(docker images -aq)

# Multi Container App avec Docker Compose

## Install Docker Compose

    Invoke-WebRequest https://dl.bintray.com/docker-compose/master/docker-compose-Windows-x86_64.exe -UseBasicParsing -OutFile $env:ProgramFiles\docker\docker-compose.exe
    docker-compose version

## Install git

    Set-ExecutionPolicy RemoteSigned -Force
    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    choco install git -params '"/GitAndUnixToolsOnPath"'
    choco list --local-only

## Deploiement ASP.NET Core MVC MusicStore app

    cd c:\GlobalMentorWeek2016
    git clone https://github.com/friism/Musicstore
    cd Musicstore
    docker-compose -f .\docker-compose.windows.yml build
    docker-compose -f .\docker-compose.windows.yml up

    docker ps
    docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" musicstore_web_1
    <IP>:5000

# Bonus si Docker Host dans une VM sur le poste de travail

## Sur le Docker Host

    # Open firewall port 2375
    netsh advfirewall firewall add rule name="docker engine" dir=in action=allow protocol=TCP localport=2375
    # Configure Docker daemon to listen on both pipe and TCP (replaces docker --register-service invocation above)
    Stop-Service docker
    dockerd --unregister-service
    dockerd -H npipe:// -H 0.0.0.0:2375 --register-service
    Start-Service docker

## Sur le Docker Client

    $env:DOCKER_HOST = "<ip-address-of-vm>:2375"
    docker version