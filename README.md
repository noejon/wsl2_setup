G'day Internet,

Here we are with the first article I am releasing on the web. And today we will talk about Windows Subsystem for Linux, a feature released as part of the insider build 
18917. We will attempt to install WSL2 and setup a node.js development environment.

[Windows has a Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about) since 2016. It enabled us to use Linux distributions on our Windows 10 systems. It now comes with [Windows Subsystem for Linux 2](https://docs.microsoft.com/en-us/windows/wsl/wsl2-index), with a shift in their architecture, improving both the performance and the compatibility between windows and the subsystem. Further information can be found in [Microsoft's developer blog](https://devblogs.microsoft.com/commandline/)

# Node.js development environment

The development environment we will set up is composed of:
- `git` as our version control system.
- `zsh` and `oh-my-zsh` to replace bash (optional).
- `visual studio code` as our text editor.
- `node.js` and `npm`.
- `docker` and `docker-compose` to containerise our projects .

But first we need to install WSL2.

# Installing WSL2
 
## Windows insider

The first thing we are going to do is enable the `Windows Insider Program`

It can be enabled in the `Settings` > `Update and Security` > `Windows Insider Program`
- Login to your `Windows Insider account` or create one.
- At the time of writing, WSL2 is in the `Fast` builds, so your `insider settings` should be set to `Fast`.
- Then go to `settings` > `Update and Security` > `Windows Update` and check for updates. Install the latest build, then restart your machine. This should take some time.
- You should now have the latest updates, and with that comes WSL2. Next we will see how we can activate WSL2.

## WSL 1

In order to install WSL2, WSL first of the name needs to be installed. This feature needs to be enabled:

Open a Powershell (as an Administrator) and type:

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
When prompted to restart our machine, we just answer with [Y]es.

After the computer restarted, we head to `the Microsoft Store` and search the term `linux`. After clicking on `Run Linux on Windows` we choose to get `ubuntu` as our linux distribution. Once ubuntu is installed, we launch. After a first initialization we are prompted for the `Unix username` and the `Unix password`. We then update ubuntu's package manager (this might take some time and you will be prompted for information twice, normally).

```zsh
sudo apt update && sudo apt upgrade
```

We are now one step away from enabling WSL2.

## WSL 

Let's open a powershell again, and enable another optional feature

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```
Again, we are prompted to restart the system. [Y]es again.

After the machine rebooted, we open a powershell as Administrator for the last time, to make sure that we make use of WSL2!

```powershell
wsl --set-version ubuntu 2
```
This process might take a few minutes, says Powershell.
__Note__: I have ran this installation on two different machines during the same week, and once I had to type `ubuntu` and the other one `ubuntu-18.04`. There are several ubuntus in the store, I might have picked different ones.

Next we set WSL2 as our default choice for WSL

```powershell
wsl --set-default-version 2
```

And this is it, we have now successfuly installed WSL2. Time to set up our development environment.

# Linux files

[The release announcement blogpost](https://devblogs.microsoft.com/commandline/wsl-2-is-now-available-in-windows-insiders/) asks us "To make sure to put the files that we will be accessing frequently inside of our Linux root file system to enjoy the file performances benefits".

We can now access files from the windows `explorer`. It is as easy as typing `\\wsl$\Ubuntu\home` in the explorer navigation bar. We are now in our home folder. 
The home folder can be pinned to `Quick access`


Most of the installation will happen in the ubuntu console. As good practice, before installing anything on linux run 

```zsh
sudo apt update && sudo apt upgrade
```

We can now start setting up our development environment.

# Changing bash to zsh

In this section we are replacing the default `bash` terminal with `zsh` and `oh-my-zsh`. You can skip to the [development tools section](#development-tools) if you plan to keep use `bash` 

To install `zsh` we open a `terminal` (the ubuntu app) and run

```zsh
sudo apt install zsh
```

`zsh` can now be launched just by typing `zsh` in the terminal. A `.zshrc` file needs to be created, and we are prompted for a choice. He we pick (0) as the `.zshrc` file will be replaced when we install `oh-my-zsh`.
We don't want to type zsh each and everu time we start the ubuntu app, so the next thing we want to is change the default shell to use `zsh`. To do so, we'll use the [chsh](http://man7.org/linux/man-pages/man1/chsh.1.html) command as per [this example](https://askubuntu.com/questions/131823/how-to-make-zsh-the-default-shell). Simply run

```zsh
chsh -s $(which zsh)
```

After this is done, we will change the theme of our `zsh` and to do so, we will leverage the power of [oh-my-zsh](https://ohmyz.sh). A simple command will install it:

```zsh 
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
I changed my theme to [the lambda theme](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes#lambda) but feel free to choose amongst all [other existing themes](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes) and follow the instructions there to change.

`zsh` also comes with a set of plugins that could be useful to increase your development speed.  Completion plugins are also available for `npm`. Please refer to the [plugin page](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins) to find anything that suits you. I enjoy working with the [git plugin](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/git/). Talking about git, it is the next tool we are going to install.

# Development tools

## git

### Install

To install git, in a wsl terminal, we run 

```zsh
sudo apt update && sudo apt upgrade
sudo apt install git
```

### End of line

The next step is to deal with cross plateform problems, where sometimes git recognise changes in files, when there are none. This is due to windows using CRLF and linux LF to signify an end of line. To fix that, the following line can be run:

```zsh
git config --global core.autocrlf input
```

### Configuring SSH

First we create a SSH key on your new linux subsystem. Instructions can be found (here)[https://help.github.com/en/enterprise/2.15/user/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent]

To generate a private and a public key for git, we run:

```zsh
ssh-keygen -t rsa -b 4096 -C "email@email.com" #replace with your email
```

This will generate a new ssh key. When prompted to choose a file where the key will be saved, we can use either the default location or input your preferred location for the file. For this example we will consider that we used the default location `~/.ssh/id_rsa`

Start the ssh-agent:

```zsh
eval "$(ssh-agent -s)"
```

We then add the ssh private key to the ssh-agent

```zsh
ssh-add ~/.ssh/id_rsa
```

After that we can add the key to a [github](https://help.github.com/en/enterprise/2.15/user/articles/adding-a-new-ssh-key-to-your-github-account) or a (gitlab)[https://docs.gitlab.com/ee/ssh/#adding-an-ssh-key-to-your-gitlab-account] account.

Now that we installed git, we are sure that the code we write does not get lost. Now to write our code, let's install Visual Studio Code.

### Visual Studio Code

We browse (the Visual Studio Code website)[https://code.visualstudio.com/], download visual studio code for **Windows**, and install it.

During the installation process, we make sure to check the box to add Visual Studio Code to the PATH, it is recommended for an extension we'll install later.

Screenshot

 Next we open a terminal within VS Code. By default the terminal should be configured to be a `powershell`. We change it to `wsl`, which should then use `zsh` (or `bash` if you skipped the `zsh` section). 

Code comes with heaps of extensions, but the one we are interested in is [VS Code Remote Development](https://code.visualstudio.com/docs/remote/remote-overview). It bundles a few extensions that are useful for remote development, including `Remote - WSL` which will do some magic for us.

In VS Code, in the extension tab we look for `Remote Development` and install it.

Screenshot

In a `zsh` ternminal we browse to our home folder and create a `dev` folder:

```zsh
cd ~ && mkdir dev && cd dev
```

Now we just start code from a ubuntu terminal

```zsh
code .
```

It should open a new VS Code window, and install a VS Code server on our WSL. Once this is done, we can now create files in our editor and they will be created in the linux file system.
This article is written using exactly this setup \o/

Now that we have installed Visual Studio Code, let's install node.js

### Node.js

#### Node Version Manager (nvm)

To install [node.js](https://nodejs.org/en/) we will leverage [nvm](https://github.com/nvm-sh/nvm). The [installation](https://github.com/nvm-sh/nvm#install--update-script) is once again quite straight forward. In a ubuntu terminal we run:

```zsh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | zsh
```
__Note__: If you did not install `zsh`, the previous command should be run with `bash` instead of `zsh`.  

If the installation is successful the following command should return `nvm`. If it does not, a simple restart of the terminal should fix it.

```zsh
command -v nvm
```

#### Installing node and npm

Let's have a look at the commands available in nvm
```zsh
nvm --help
```
The command we are interested in in the `nvm install` command. Let's install node's latest LTS (Long-term support) version:

```zsh
nvm install --lts
```

That's it, we installed `node` and `npm`. But let's check that I am not lying by checking the versions.

```zsh
node --version && npm --version
```

Our journey to set up our development environment is almost complete. The last thing we need to do is to take one last (whale-)boat trip. Here comes Docker.

### Docker

#### Docker CE

To install [docker](https://www.docker.com/), we will follow the [steps from the official documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

First we update the `apt` package

```zsh
sudo apt-get update
```

After what we install a few required dependencies

```zsh
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

We then need to add Docker's official GPG key

```zsh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

We need to make sure that we have the key with fingerprint `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`

```zsh
sudo apt-key fingerprint 0EBFCD88
```

Docker needs to be added to the list of repositories

```zsh
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

__Note__: My processor is an amd64 process, if you have a different type of processor, replace `[arch=amd64]` accordingly (other possible values: `armhf`, `arm64`, `ppc64e1` and `390x`).

We are now ready to install (Docker Community Edition)[https://docs.docker.com/install/]

```zsh
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Now let's make sure that docker is installed.

```zsh
sudo docker run hello-world
```

This should download a test image and run it in a container. If the docker service is not started, it needs a (manual kickstart)[https://docs.docker.com/config/daemon/systemd/] and in our case, with ubuntu we will use:

```zsh
sudo service docker start
sudo docker run hello-world
```

#### Docker Compose

In this world full of microservices, a tool such as [docker compose](https://docs.docker.com/compose/) is a time saver. It allows us to run several application containers from one single command. [The installation steps](https://docs.docker.com/compose/install/) are as follows:

First download the stable release of Docker Compose (`1.24.1` at time of writing)

```zsh
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Once the binary is downloaded, it needs to have execution permissions:

```zsh
sudo chmod +x /usr/local/bin/docker-compose
```

Finally we check the installation by running

```zsh
docker-compose --version
```

which should return a docker-compose version and a build id (`docker-compose version 1.24.1, build 1110ad01` at time of writing)

We now have a machine ready to create the craziest node apps from our windows machine. 

I hope that you enjoyed this article, the first one I ever wrote. Feel free to leave some feedback.

That's all folks!

Jonathan.