EnvZ
====

This is an opinionated shared environment that augments the zshell.  It'll set up a bunch of stuff that nobody wants to bother with.

PREFERRED usage is with ZSH but it is portable to work with bash

Things like

- Installing a default editor ([atom](https://atom.io/)) and mapping it to `e` in the terminal
- useful aliases
- useful autocompletes
- adding cutom user modules
- simplifying installing via brew
- easy way to update your environment

Clone the repo wherever you want and just add the following to the bottom of your .zshrc file
```
. <path>/EnvZ/bootstrap
```

The bootstrapper may prompt or install required items (such as python) and give you the option to install a default editor or other items

When updates are pushed just execute `reload-env`

## Source directory

A key part of the environment is knowing where you store your source code. When the environment first starts it may ask you to put in where you source code exists.  Environment variables will work as well as `~`. All stored settings are put into the `.config` file. If you mess up, blow the file out.

This is nice because you can go to a coworkers machine and just type `src` in the shell and go to their source folder. Now everyone works in the same logical directory structure even though the physical directory structure is different

## Updating

If you have linked modules, you can easily update all of them by doing `update-env` (assuming they are all git folders) and envz will do a git pull on all of them.

## Sensitive information

Add any non shareable keys to a file called `keys` in the modules folder. `modules/keys` will be sourced if it exists.  For example, to add your git
oauth token do

```
export GIT_TOKEN=....
```

In the keys folder.  

## Custom plugins

To add a folder to your set of shell loads, go to your path and do

```
add-user-env <folder>
```

This will add the folders script contents to be sourced on shell start. Only files that end with `.sh` will get sourced and only the first directory level (this lets you build your own custom folders that contain whatever you want that wont get sourced)

To remove a custom folder do

```
remove-user-env <folder>
```

## Setup

1. Homebrew
Install Homebrew via the following command

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

2. Iterm2
If you don't have iterm2, install it with brew:


```
brew cask install iterm2

```

Now run the following command to add zsh to default shells and restart the terminal after the command.


```
brew install zsh
sudo dscl . -create /Users/$USER UserShell /usr/local/bin/zsh
```

Remember to restart the terminal after the command

3. Curl
In order to call APIs using client certs you need to have a version of curl with openssl support


```
brew install curl --with-openssl && brew link curl --force

```

4. Kubernetes
Install it using these commands


```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.12.7/bin/darwin/amd64/kubectl
 
chmod +x kubectl
 
sudo mv kubectl /usr/local/bin/kubectl

```

5. Git
Install git to allow you to configure the rest of the environment and add a SSH key to github


```
brew install git
 
ssh-keygen -t rsa -b 4096 -C "your_email@godaddy.com"
 
ssh-add ~/.ssh/id_rsa

```

Add the SSH key to GitHub using this guide.
Configure your git user name and password using the following commands:


```
git config --global user.name "your name"
 
git config --global user.email your_email@godaddy.com
 
```

##(Optional) Configure Git to always set new branches to rebase when doing a pull:

```
git config --global branch.autosetuprebase always

```

In addition, optionally install kubectx with the following command. Kubectx allows us to easily switch between namespaces and contexts

```
brew install kubectx

```


6. EnvZ setup
Clone the following repo to your machine to ~ folder 

If you get permission error such as "fatal: could not create work tree dir 'EnvZ': Permission denied", make sure you are in ~ folder. 

Run the setup script. This will install Yadr, gradle and Atom in addition to setting up your shell. At several points through the script, you will be prompted for your administrator password.


```
git clone https://github.secureserver.net/platapi/EnvZ
 
./EnvZ/setup.sh

```
After the script has run, restart your shell. When the shell restarts, Homebrew will run to install some packages and then you will be prompted to enter the path to the directory you use to store your code (~/Source or ~/code for example). Enter your directory path and press "Enter". Please remember this directory for team setup step.



The script populates your .zshrc file. Once that has been done, add this line at the very end, right after for config_file ($HOME/.yadr/zsh/*.zsh) source $config_file:

```
. /Users/$USER/EnvZ/bootstrap

```
where $USER is your username. To be safe, replace $USER with your actual username. If you didn't put EnvZ in your ~ directory, then be sure to change the path.

Next, the shell will ask you for your password and install more components. You may optionally install the Atom editor during this phase. Next, you will receive a warning saying that the "hub config is missing". To suppress this warning, execute the following command and enter your GitHub credentials when prompted:


```
cd {path/to/EnvZ/repo}/EnvZ && init-hub

```
This will generate a new GitHub access token that is stored in ~/.config/hub.





To suppress the final warning produced by the shell saying that "No 'GIT_TOKEN' env variable was found.", copy the access token from ~/.config/hub then run the following command (filling in the appropriate values):
```
echo "export GIT_TOKEN={your access token}" > {path/to/EnvZ/repo}/EnvZ/modules/keys
```

7. Team setup
The following block will set up team specific modules for your machine

Running this command will install Prezto (a configuration framework for Zsh) and will override any shell configurations you have previously made. If you have a current setup you would like to keep, feel free to skip this step.

git clone git@github.secureserver.net:platapi/platapi-team-modules.git
 
 
add-user-env platapi-team-modules
Create all of your kubernetes contexts for the team using this command:

create-kube-contexts


Here is a list of useful aliases found in the `platapi-team-modules/02_kubernetes.sh` script

### General k8s commands
kube_cleanup_deployment : Delete's Ingress, SVC, Deployments, RS, and verifies pods
kgc : get contexts
kgp : get pods
kl : get logs
ke : exec bash
ks : exec sh
add-secret : ''
get-secret : ''
...


You can add custom validation to your modules that will run after all other modules are run by creating a `validate.sh` file in your module.  This file WILL NOT get run during the initial load, and only run after the fact.

## Zsh autocompletion detection

If your user module has a folder called `zsh_completions` it will automatically get added to the `fpath` for loading

## Validation

If your user module has a file called `validate.sh` it will be executed after your module is loaded as a verification phase

And it will give you autocomplete on your installed plugins

For more information about use cases see this [blog post](http://onoffswitch.net/shareable-zsh-environment-envz/)
