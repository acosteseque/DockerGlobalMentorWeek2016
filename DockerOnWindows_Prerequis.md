# Avant de venir à la Docker Global Mentor Week

Pour ne pas perdre de temps au demarrage des Labs voici ce qu'il vous faut preparer avant de venir.

Attention : le plus important des prerequis est l'espace disponible sur le Docker Host (votre poste ou VM de travail) est l'espace disponible ... pour pouvoir faire tous les labs Windows Container il faudra pull les 2 Base Images de Microsoft :
* Windows Server Core (microsoft/windowsservercore) qui fait 8Go
* Nano Server (microsoft/nanoserver) qui fait 800Mo

Les explications sur pourquoi cette taille vous seront données durant la presentation :)

Il vous faudra donc 10Go d'espace disponible (qui seront liberés apres les Labs)

# 0 - BYOD (Bring your Own Device) ;)

Pas de postes disponibles, il vous faut votre materiel pour realiser les Labs

# 1 - Set-up Docker sur votre ordinateur (OS Windows)

## Windows 7 / 8 / 8.1 / 10 **avant** Anniversary Update

il vous faudra installer une VM Windows Server 2016 en evaluation ou bien Windows 10 pro 1607 (AE (Anniversary Update (du mois d'aout 2016)))

Puis installer dans la VM [Docker for Windows](https://docs.docker.com/docker-for-windows/) en version beta

## Windows 10 Anniversary Update (du mois d'aout 2016) (version 1607)

il vous faudra installer [Docker for Windows](https://docs.docker.com/docker-for-windows/) en version beta

Note : Il ne faut pas mixer les hyperviseurs, si VirtualBox ou VMware Workstation est installé il faudra les desinstaller. Durant le process d'installation Hyper-V est activé et doit être le seul hyperviseur sur l'ordinateur.

> Pour plus de details voir [setup steps in the Windows Container lab](https://github.com/docker/labs/blob/master/windows/windows-containers/Setup.md)

### ! Attention !

[Docker Toolbox](https://docker.github.io/toolbox/) ne permet que le support des containers Linux pas Windows

# 3 - Recuperer les Base Images Microsoft

Lancer PowerShell

```
docker pull microsoft/windowsservercore
docker pull microsoft/nanoserver
```

# Have Fun with Windows Containers & Docker ! :metal:

![Dojocat](http://octodex.github.com/images/dojocat.jpg)