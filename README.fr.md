# Lancer WSL directement dans le répertoire utilisateur Linux

[English version](./README.md)

## Contexte

Par défaut, lorsqu’on lance WSL depuis PowerShell avec :

```powershell
wsl
```

WSL peut s’ouvrir dans le répertoire Windows courant, par exemple :

```bash
/mnt/c/Users/<windows-user>
```

Ce comportement peut être gênant lorsqu’on souhaite démarrer directement dans le répertoire utilisateur Linux de sa distribution WSL.

L’objectif est de faire en sorte que la commande :

```powershell
wsl
```

ouvre directement :

```bash
/home/[YOUR_LINUX_USERNAME]
```

au lieu d’un chemin Windows monté dans WSL, comme :

```bash
/mnt/c/Users/<windows-user>
```

## Objectif

Configurer PowerShell pour que l’exécution de :

```powershell
wsl
```

lance automatiquement une distribution WSL précise et démarre directement dans le répertoire utilisateur Linux :

```bash
/home/[YOUR_LINUX_USERNAME]
```

Les exemples de ce guide utilisent les placeholders suivants :

```text
[YOUR_DISTRO_NAME]
[YOUR_LINUX_USERNAME]
```

Il faut les remplacer par vos propres valeurs.

Exemple :

```text
[YOUR_DISTRO_NAME]       -> FedoraLinux-44
[YOUR_LINUX_USERNAME]    -> votre nom d’utilisateur Linux
```

## Vérifier le nom de la distribution WSL

Commencez par vérifier le nom exact de votre distribution WSL :

```powershell
wsl -l -v
```

Exemple de sortie :

```text
  NAME                  STATE           VERSION
* [YOUR_DISTRO_NAME]    Running         2
```

Utilisez exactement le nom affiché dans la colonne `NAME`.

## Vérifier le nom d’utilisateur Linux

Ouvrez votre distribution WSL, puis exécutez :

```bash
whoami
```

Le résultat correspond à votre nom d’utilisateur Linux.

Le répertoire personnel Linux est généralement :

```bash
/home/[YOUR_LINUX_USERNAME]
```

Vous pouvez le confirmer avec :

```bash
echo $HOME
```

## Test manuel

Avant de rendre la modification permanente, testez la commande manuellement depuis PowerShell :

```powershell
wsl -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
```

Une fois dans WSL, vérifiez le répertoire courant :

```bash
pwd
```

Résultat attendu :

```bash
/home/[YOUR_LINUX_USERNAME]
```

Si ce test fonctionne, vous pouvez rendre le comportement permanent dans PowerShell.

## Faire démarrer `wsl` dans le répertoire utilisateur Linux par défaut

Ouvrez votre profil PowerShell :

```powershell
notepad $PROFILE
```

Si le fichier n’existe pas encore, créez-le avec :

```powershell
New-Item -ItemType File -Path $PROFILE -Force
notepad $PROFILE
```

Ajoutez ensuite cette fonction dans le profil PowerShell :

```powershell
function wsl {
    if ($args.Count -eq 0) {
        & "$env:WINDIR\System32\wsl.exe" -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
    } else {
        & "$env:WINDIR\System32\wsl.exe" @args
    }
}
```

Enregistrez le fichier, fermez PowerShell, puis ouvrez-le à nouveau.

## Test final

Exécutez :

```powershell
wsl
```

Puis vérifiez le répertoire courant dans WSL :

```bash
pwd
```

Résultat attendu :

```bash
/home/[YOUR_LINUX_USERNAME]
```

## Pourquoi cette méthode fonctionne

Cette fonction PowerShell modifie le comportement de `wsl` uniquement lorsqu’aucun argument n’est fourni.

Lorsque vous lancez :

```powershell
wsl
```

PowerShell exécute en réalité :

```powershell
wsl.exe -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
```

En revanche, lorsqu’une commande WSL avec arguments est utilisée, par exemple :

```powershell
wsl -l -v
```

les arguments sont transmis au véritable binaire Windows :

```powershell
C:\Windows\System32\wsl.exe
```

Cela permet de conserver le fonctionnement normal des commandes WSL classiques.

## Résumé

Avant :

```powershell
wsl
```

pouvait ouvrir WSL dans un chemin Windows monté :

```bash
/mnt/c/Users/<windows-user>
```

Après :

```powershell
wsl
```

ouvre directement :

```bash
/home/[YOUR_LINUX_USERNAME]
```

Cela évite d’avoir à exécuter manuellement :

```bash
cd /home/[YOUR_LINUX_USERNAME]
```

à chaque lancement de WSL.

## Notes

Si vous ne souhaitez pas remplacer le comportement de la commande `wsl`, vous pouvez créer une fonction séparée :

```powershell
function wslhome {
    & "$env:WINDIR\System32\wsl.exe" -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
}
```

Vous pourrez ensuite lancer WSL avec :

```powershell
wslhome
```

Cette alternative est utile si vous souhaitez conserver le comportement par défaut de `wsl`.
