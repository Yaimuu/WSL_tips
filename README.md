# WSL_tips
A repository designed to give tips about the WSL (Windows Subsystem for Linux)

## 1. Install WSL 2

To install WSL 2 and Ubuntu 20.04, Open PowerShell as Administrator and run:

``` powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Download the following package for your system and run it : 

> **x64 :**
>
> https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

> **arm64 :**
>
> https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_arm64.msi

Configure your WSL version 2 by default :

``` powershell
wsl --set-default-version 2
```

Download Ubuntu 20.04 :

<span style="color:red">If you already had an old version of ubuntu, uninstall it!</span>

``` powershell
Invoke-WebRequest -Uri https://aka.ms/wslubuntu2004 -OutFile Ubuntu.appx -UseBasicParsing
Add-AppxPackage .\Ubuntu.appx
```

If the installation above is complete, left click on Windows icon. Select Ubuntu 20.04 LTS and continue with the configuration. Provide the username and password when prompted.

When ubuntu is running, check you have the correct version using the following command in your powershell :

``` powershell
wsl -l -v
```
For more information, see [Ubuntu 20.04 installation](https://ripon-banik.medium.com/how-to-install-wsl2-offline-b470ab6eaf0e)


*NOTE : Your WSL environment may have been setup by Docker Desktop as it's used to run Docker. In that case, the password of the sudo user on WSL is the same as your account's password to Docker Desktop.*

To ensure having no problem afterwards, open ubuntu as administrator.

## 2. Configure WSL

### Override $PATH variable

To prevent any mismatch between Linux and Windows depedencies, add thes lines in your ```/etc/wsl.conf``` (if it doesn't exist, create it) :

```sh
[interop]
enabled = false # enable launch of Windows binaries; default is true
appendWindowsPath = false # append Windows path to $PATH variable; default is true
```

> **OR**

If you want to be sure the ```$PATH``` has no Windows depencies, run the following.

**Run the following only once !**

The command will add this line ``` echo $PATH | tr ':' '\n' | grep -v /mnt/ | tr '\n' ':'` ``` to ```~/.bashrc``` file. This line removes all the Windows references from the ```$PATH```.
``` bash
$ echo "export PATH=`echo $PATH | tr ':' '\n' | grep -v /mnt/ | tr '\n' ':'`" >> ~/.bashrc
```

If you want it to be effective, restart your WSL or run the following
``` bash
$ export PATH=`echo $PATH | tr ':' '\n' | grep -v /mnt/ | tr '\n' ':'`
```

Check that your PATH does not contain any Windows references. They always sart with ```/mnt/```
``` bash
$ echo $PATH
```

Then, restart your WSL.

### Setup the environment

> #### GUI for WSL

To run ```az acr``` commands afterwards, you have to log in thanks to a web browser. To do so, you need to install an external software for Windows : [XLaunch](https://sourceforge.net/projects/vcxsrv/).

Everytime you want to use your WSL, you should also launch this software and follow these screenshots :

**SCREENSHOTS OF THE APP**

![Xsrv screen 1](./Ressources/Xserver_1.png)
![Xsrv screen 2](./Ressources/Xserver_2.png)
![Xsrv screen 3](./Ressources/Xserver_3.png)
![Xsrv screen 4](./Ressources/Xserver_4.png)

You can now download a web browser like Firefox, Google Chrome, Chromium or another if you want to.

You can also install the text editor for Linux, gedit :
``` bash
sudo apt-get install gedit
```

### Setup your WSL to work with the VPN

1. During your VPN session, run in **PowerShell** to get your VPN's DNS adress
``` powershell
nslookup
```
You'll get the IPv4 adress of your corporate nameserver Copy this address.

2. Disable ```resolv.conf``` generation in **WSL**:
``` sh
sudo nano /etc/wsl.conf
```
copy this text to the file (to disable resolve.conf generation, when wsl starts up)
```conf
[network]   
generateResolvConf = false
```
3. **In WSL** Add your corporate nameserver to ```resolv.conf```
``` sh
sudo nano /etc/resolv.conf
```
``` sh
nameserver <VPN DNS adress>
```
4. Set your VPN adapter (if you have Cisco AnyConnect) **open a admin PowerShell**

Set adapter metric (Replace -Match with your name), **run this after ever reboot or VPN reconnect**:
``` powershell
Get-NetAdapter | Where-Object {$_.InterfaceDescription -Match "Cisco AnyConnect"} | Set-NetIPInterface -InterfaceMetric 6000
```
5. Restart your wsl **in PowerShell**
``` powershell 
wsl.exe --shutdown
```
6. Test it **in wsl** run
```sh
wget google.com
```
[More info here](https://stackoverflow.com/questions/66444822/no-internet-connection-ubuntu-wsl-while-vpn)

### Run VS Code in WSL

With VS Code, you are able to connect to your WSL via SSH

[Follow this tutorial](https://code.visualstudio.com/docs/remote/wsl)

### Useful tips

*TEMPLATE*
> #### Application
``` bash
# Comment
command 
```

> #### Git
If you want to know in real time in which branch you are when working on a git repository, add the following code at the end of your ```~/.bashrc``` file.
```sh
# Show git branch
show_git_branch() {
   git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
 }
 # \u : user - \h : host - \w : path
export PS1="${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w \[\033[31m\]\$(show_git_branch)\[\033[00m\]$\[\033[00m\] "
```

> #### Kube
If you want to know in real time in which context you are when working with kube, add the following code at the end of your ```~/.bashrc``` file.
```sh
# prompt
CONTEXT=$(cat ~/.kube/config | grep "current-context:" | sed "s/current-context: //")
export PS1="${GREEN}\u${YELLOW}(k8s: $CONTEXT)${NORMAL} \$ "
```
Some useful kube aliases :
```sh
# some kube aliases
alias k='kubectl'
alias klog='kubectl get events --sort-by=.metadata.creationTimestam'
alias kap='kubectl apply -f'
alias kget='kubectl get'
alias kdes='kubectl describe'
alias klogs='kubectl logs'
alias kdel='kubectl delete'
alias kcns='kubectl config set-context --current'
# kube stuff
source <(kubectl completion bash) # setup autocomplete in bash
```

After editing the ```~/.bashrc``` file, the coloration may disapear, to fix this you can use :
```sh
bash
# OR
source ~/.bashrc
```
### Useful documentation

[WSL2 Hacks](https://github.com/shayne/wsl2-hacks)

[Dev Containers](https://code.visualstudio.com/docs/remote/containers)
