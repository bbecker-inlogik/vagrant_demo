#Vagrant Demo

## Windows installation
 
Download and install the following:

* [VirtualBox]
* [Vagrant]
* [Cygwin]

In the spirit of automation use [chocolatey]. 
To install chocolatey run the commands from website in admin cmd or admin Powershell.

Restart cmd or powershell (path needs to be refreshed)
From cmd or powershell run 
```bash
choco /?
cinst vagrant virtualbox cyg-get
```
> choco /? returns a list of functions

> cinst is shorthand for choco install

Virtual Box, Vagrant and 'Cygwin64 Terminal' are now installed as a program on windows.

Cygwin installs to c:\ProgramData\Chocolatey\lib\Cygwin\tools\cygwin

Home directory is emulated, but real location is:
C:\ProgramData\Chocolatey\lib\Cygwin\tools\cygwin\home\<current_user>

To access C drive use /cygdrive/c

To use home type ~


Install some needed cygwin packages.
Open new command prompt with updated path from cygwin installation.
```bash
cyg-get openssh
cyg-get rsync
cyg-get ncurses
cyg-get git
```

Open 'Cygwin64 Terminal' and test if aspects are installed: 
```bash
$ ssh
$ rsync
$ clear
```

Avoid mixing cygwin and standard command since home directories are different can lead to confusion.

###Mac installation
Use [homebrew](https://www.brew.sh)
```bash
$ brew tap caskroom/cask
$ brew install brew-cask 
$ brew cask install vagrant virtualbox
```

###Ubuntu installation

run:
```bash
wget https://dl.bintray.com/mitchellh/vagrant/vagrant_1.6.3_x86_64.deb
sudo dpkg -i vagrant_1.6.3_x86_64.deb
```

Follow instructions for installation on virtualbox.org
```bash
sudo add-apt-repository -y "deb http://download.virtualbox.org/virtualbox/debian trusty contrib"
wget http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc
gpg --with-fingerprint oracle_vbox.asc
	check that fingerprint matches that found on the virtualbox website
sudo apt-key add oracle_vbox.asc
sudo apt-get update
	install virtualbox and dynamic kernel module support
sudo apt-get -y install virtualbox-4.3 dkms
```

##Getting started

check from command or cygwin that vagrant, virtualbox and ssh is working by typing:
```bash
vagrant -v
vboxmanage -v
ssh
```
> if not found check path and restart cygwin to update path

Open Cygwin.
```bash
mkdir ubuntu
cd ubuntu/
vagrant init hashicorp/precise32
vagrant up
vagrant ssh
```
> /home/&lt;current_user&gt;/ubuntu ==> C:\ProgramData\Chocolatey\lib\Cygwin\tools\cygwin\home\&lt;current_user&gt;\ubuntu

Now we are at the command line of Ubuntu.
```bash
uname
exit
```
> exit ends the ssh session

```bash
vboxmanage list runningvms
vboxmanage list vms
```

Vagrant by default runs headless.

To run with a GUI uncomment the following lines in the vagrantfile:
```bash
	config.vm.provider "virtualbox" do |vb|
     # Display the VirtualBox GUI when booting the machine
     vb.gui = true
  
     # Customize the amount of memory on the VM:
     vb.memory = "1024"
   end
```

```bash
vagrant reload
```

Vagrant operates on the vm located in the current directory so make sure you are in the correct directory

##Stopping and starting VMs
Hibernate is fast to resume but uses more memory
```bash
vagrant suspend
vagrant resume
vagrant ssh
```

Shutdown and restart
```bash
vagrant halt
vagrant up
```
or
```bash
vagrant reload
```
> vagrant reload is the same as running vagrant halt and vagrant up

To remove a vm:
---------------
```bash
vboxmanage list vms
vagrant destroy
vboxmanage list vms
vagrant status
```

Create a VM
```bash
vagrant up
```

For Help
```bash
vagrant -h
vagrant status --debug
```

###Note on line endings:
If working on linux guest on windows host end of line feeds can be an issue.

add a .gitattributes file to the root git directory

for linux projects:
```bash
* text eol=lf
```

for windows projects:
```bash
* text eol=crlf
```

Configure popular text editors and IDEs use editorconfig.org

.editorconfig file for linux
```bash
	# top-most EditorConfig file
	root = true
	
	# Unix-style newlines with a newline ending every line
	[*]
	end_of_line = lf
	insert_final_newline = true
	indent_style = space
	indent_size = 2
```

####Setup SublimeText and editorconfig
```bash
choco install SublimeText3
```

install package control , https://packagecontrol.io/installation

SublimeText3 > tools > command palette

or

ctrl+shift+P and type package Control: install package search for editorconfig and install


DEMO
----------
```bash
mkdir nginx
cd nginx
vagrant init hashicorp/precise32 --minimal
```

Provisioning:
open Vagrantfile and add
```bash
  config.vm.hostname = "web-dev"
  config.vm.provision "shell", path: "provision.sh"
```

add provision.sh and modify
```bash
	apt-get -y update
	apt-get -y install nginx
	service nginx start
```

run
```bash
vagrant up
vagrant ssh
service nginx status
wget -qO- localhost
```

We want to view the website from the host.

open Vagrantfile and add
```bash
  config.vm.network "forwarded_port", guest: 80, host: 8080, id: "nginx"
```

```bash
vagrant reload
```

On host browser visit localhost:8080

We want our source code in the shared folder so we can edit on the host.
```bash
ls /usr/share/nginx/www
ls /vagrant
cp -r /user/share/nginx/www /vagrant/

rm -rf /usr/share/nginx/www
ln -s /vagrant/www /usr/share/nginx/www
```
> cp copy www folder to vagrant  
> rm remove www folder  
> ln create link  
> default shared folder is located in /vagrant on guest 

edit html file on host and reload website

```bash
vagrant reload
```
reload localhost:8080

Our link creation has been lost so add it to the provision.sh
```bash
apt-get -y update
apt-get -y install nginx

rm -rf /usr/share/nginx/www
ln -s /vagrant/www /usr/share/nginx/www

service nginx start
```

```bash
vagrant reload
```

> this will not rerun the provisioning since it is only run on the first vagrant up

To re-provision run
```bash
vagrant provision
```
 or 
```bash
vagrant destroy
vagrant up
```

###Setup Git
Each project will have a Vagrantfile committed to version control
```bash
git init
```

before you can connect to github you need to generate ssh keys
```bash
ssh-keygen -t rsa -b 4096 -C "name@email.com"
```
follow instructions and enter a strong password

add contents of generated ssh public key to github, settings > ssh keys > add new key
```bash
$ clip < ~/.ssh/id_rsa.pub
```
> Copies the contents of the id_rsa.pub file to your clipboard

add new ssh key to ssh-agent
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

Now you may to use git and github from cygwin.

##Provisioning Benefits:
* Knowledge of environment setup is retained
* Easily recreate environment
* Test environment same as production
* Can commit setup to version control
* Can rollback to last known working configuration if something goes wrong

If you share your vm publicly make sure to overwrite the default ssh key that comes with the box as part of your provisioning.

I hope you find this guide useful.


[VirtualBox]:https://www.virtualbox.org
[Vagrant]:https://www.vagrantup.com
[Cygwin]:https://www.cygwin.com
[chocolatey]:https://www.chocolatey.org