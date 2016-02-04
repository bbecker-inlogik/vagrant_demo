Vagrant Demo
============

### Windows installation
 
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

Cygwin
```bash
mkdir ubuntu
cd ubuntu/
vagrant init hashicorp/precise32
vagrant up
vagrant ssh
```
> /home/&lt;current_user&gt;/ubuntu ==> C:\ProgramData\Chocolatey\lib\Cygwin\tools\cygwin\home\&lt;current_user&gt;\ubuntu

now we are at the command line of Ubuntu
```bash
uname
exit
```
> exit -to end ssh session

vboxmanage list runningvms
vboxmanage list vms

vagrant by default runs headless
to run with a GUI uncomment the following lines in the vagrantfile:
	config.vm.provider "virtualbox" do |vb|
     # Display the VirtualBox GUI when booting the machine
     vb.gui = true
  
     # Customize the amount of memory on the VM:
     vb.memory = "1024"
   end

vagrant reload

vagrant operates on the vm located in the current directory so make sure you are in the correct directory

Stopping and starting vms
-------------------------
to hibernate use: 	fast to resume but uses more memory
vagrant suspend
vagrant resume
vagrant ssh

to shutdown use:
vagrant halt

to restart use:
vagrant up
or
vagrant reload //same as vagrant halt and vagrant up


To remove a vm:
---------------
vboxmanage list vms
vagrant destroy
vboxmanage list vms
vagrant status

to recreate vm use
vagrant up

for Help use
vagrant -h
vagrant status --debug

Note on line endings:
--------------------
If working on linux guest on windows host end of line feeds can be an issue.
add a .gitattributes file to the root git directory
for linux projects:
1)	* text eol=lf

for windows projects:
1)	* text eol=crlf

to configure popular text editors and IDEs use
editorconfig.org

.editorconfig file for linux
	# top-most EditorConfig file
	root = true
	
	# Unix-style newlines with a newline ending every line
	[*]
	end_of_line = lf
	insert_final_newline = true
	indent_style = space
	indent_size = 2
	
choco install SublimeText3
install package control , https://packagecontrol.io/installation
tools > command palette
ctrl+shift+P and type package Control: install package search for editorconfig and install

----------
DEMO
----------
mkdir nginx
cd nginx

vagrant init hashicorp/precise32 --minimal

Provisioning:
open Vagrantfile and add
  config.vm.hostname = "web-dev"

  config.vm.provision "shell", path: "provision.sh"

add provision.sh and modify
	apt-get -y update

	apt-get -y install nginx

	service nginx start
 
vagrant up
vagrant ssh
service nginx status
wget -qO- localhost

We want to view website from the host.
open Vagrantfile and add
  config.vm.network "forwarded_port", guest: 80, host: 8080, id: "nginx"

vagrant reload //same as vagrant halt and vagrant up

On host visit localhost:8080

We want out source code in the shared folder so we can edit on the host.
ls /usr/share/nginx/www
ls /vagrant
 //default shared folder is located in /vagrant on guest 
cp -r /user/share/nginx/www /vagrant/

rm -rf /usr/share/nginx/www
ln -s /vagrant/www /usr/share/nginx/www

edit html file on host and reload website

vagrant reload
reload website

Our link creation has been lost so add it to the provision.sh
	apt-get -y update

	apt-get -y install nginx

	rm -rf /usr/share/nginx/www
	ln -s /vagrant/www /usr/share/nginx/www

	service nginx start

vagrant reload
  //this will not rerun the provisioning since it is only run on the first vagrant up
vagrant provision
 or 
vagrant destroy
vagrant up

Provisioning Benefits:
knowledge of environment setup is retained
Easily recreate environment
Test environment same as production
Can commit setup to version control
Can rollback to last known working configuration if something goes wrong

If you share your vm publicly make sure to overwrite the default ssh keys that come with the box as part of your provisioning.


[VirtualBox]:https://www.virtualbox.org
[Vagrant]:https://www.vagrantup.com
[Cygwin]:https://www.cygwin.com
[chocolatey]:https://www.chocolatey.org