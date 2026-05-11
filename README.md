# Launch WSL directly into the Linux home directory

## Context

By default, when launching WSL from PowerShell with:

```powershell
wsl
```

WSL may open inside the current Windows directory, for example:

```bash
/mnt/c/Users/<windows-user>
```

This behavior can be annoying when you want to start directly inside your Linux user home directory.

The goal is to make the command:

```powershell
wsl
```

open directly in:

```bash
/home/[YOUR_LINUX_USERNAME]
```

instead of a mounted Windows path such as:

```bash
/mnt/c/Users/<windows-user>
```

## Goal

Configure PowerShell so that running:

```powershell
wsl
```

automatically launches a specific WSL distribution and starts directly inside the Linux home directory:

```bash
/home/[YOUR_LINUX_USERNAME]
```

Example placeholders used in this guide:

```text
[YOUR_DISTRO_NAME]
[YOUR_LINUX_USERNAME]
```

You must replace them with your own values.

Example:

```text
[YOUR_DISTRO_NAME]       -> FedoraLinux-44
[YOUR_LINUX_USERNAME]   -> your Linux username
```

## Check the WSL distribution name

First, check the exact name of the installed WSL distribution:

```powershell
wsl -l -v
```

Example output:

```text
  NAME                  STATE           VERSION
* [YOUR_DISTRO_NAME]    Running         2
```

Use the exact distribution name shown in the `NAME` column.

## Check the Linux username

Open your WSL distribution, then run:

```bash
whoami
```

The result is your Linux username.

Your Linux home directory is usually:

```bash
/home/[YOUR_LINUX_USERNAME]
```

You can confirm it with:

```bash
echo $HOME
```

## Manual test

Before making the change permanent, test the command manually from PowerShell:

```powershell
wsl -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
```

Then, inside WSL, verify the current directory:

```bash
pwd
```

Expected result:

```bash
/home/[YOUR_LINUX_USERNAME]
```

If this works, you can make the behavior permanent in PowerShell.

## Make `wsl` start in the Linux home directory by default

Open your PowerShell profile:

```powershell
notepad $PROFILE
```

If the file does not exist, create it first:

```powershell
New-Item -ItemType File -Path $PROFILE -Force
notepad $PROFILE
```

Add this function to the PowerShell profile:

```powershell
function wsl {
    if ($args.Count -eq 0) {
        & "$env:WINDIR\System32\wsl.exe" -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
    } else {
        & "$env:WINDIR\System32\wsl.exe" @args
    }
}
```

Save the file, close PowerShell, and open it again.

## Final test

Run:

```powershell
wsl
```

Then check the current directory inside WSL:

```bash
pwd
```

Expected result:

```bash
/home/[YOUR_LINUX_USERNAME]
```

## Why this works

This PowerShell function changes the behavior of `wsl` only when no argument is provided.

When running:

```powershell
wsl
```

PowerShell executes:

```powershell
wsl.exe -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
```

But when running a WSL command with arguments, such as:

```powershell
wsl -l -v
```

the function forwards the arguments to the real Windows binary:

```powershell
C:\Windows\System32\wsl.exe
```

This means normal WSL commands still work as expected.

## Summary

Before:

```powershell
wsl
```

could open WSL inside a Windows-mounted path:

```bash
/mnt/c/Users/<windows-user>
```

After:

```powershell
wsl
```

opens directly inside:

```bash
/home/[YOUR_LINUX_USERNAME]
```

This avoids having to manually run:

```bash
cd /home/[YOUR_LINUX_USERNAME]
```

every time WSL starts.

## Notes

If you do not want to override the `wsl` command, you can create a separate function instead:

```powershell
function wslhome {
    & "$env:WINDIR\System32\wsl.exe" -d [YOUR_DISTRO_NAME] --cd /home/[YOUR_LINUX_USERNAME]
}
```

Then launch WSL with:

```powershell
wslhome
```

This alternative is useful if you want to keep the default `wsl` behavior unchanged. 
