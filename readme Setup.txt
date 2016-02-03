====================
Windows installation
====================
download and install
VirtualBox	virtualbox.org
Vagrant		vagrantup.com

or use chocolatey.org
------------------------
Run commands from website in admin cmd or admin Powershell to install chocolatey

restart cmd or powershell
from cmd or powershell run choco /? for a list of functions
run cinst vagrant virtualbox tortoisegit
 - cinst is shorthand for choco install

Also need ssh and rsync 
cygwin.com has both

choco install cyg-get

installs to c:\ProgramData\Chocolatey\lib\Cygwin\tools\cygwin

need to also install some cygwin packages 
open new command prompt with updated path from cygwin installation

cyg-get openssh
cyg-get rsync
cyg-get ncurses
cyg-get git

'cygwin64 terminal' is installed as a program on windows
test if aspects are installed type: ssh, rsync, clear

Home directory is emulated, but real location is:
C:\ProgramData\Chocolatey\lib\Cygwin\tools\cygwin\home\<current_user>

to access C drive use /cygdrive/c
to use home type ~

Avoid mixing cygwin and standard command since home directories are different can lead to confusion

======
mac install use homebrew, brew.sh
brew tap caskroom/cask
brew install brew-cask 
brew cask install vagrant virtualbox

===================
Ubuntu installation
===================
run:
wget https://dl.bintray.com/mitchellh/vagrant/vagrant_1.6.3_x86_64.deb
sudo dpkg -i vagrant_1.6.3_x86_64.deb

follow instructions for installation on virtualbox.org
sudo add-apt-repository -y "deb http://download.virtualbox.org/virtualbox/debian trusty contrib"
wget http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc
gpg --with-fingerprint oracle_vbox.asc
	check that fingerprint matches that found on the virtualbox website
sudo apt-key add oracle_vbox.asc
sudo apt-get update
	install virtualbox and dynamic kernel module support
sudo apt-get -y install virtualbox-4.3 dkms

+++++++++++++++++++
check from command or cygwin that vagrant, virtualbox and ssh is working by typing:
vagrant -v
vboxmanage -v
ssh
	if not found check path and restart cygwin to update path

mkdir ubuntu
	C:\ProgramData\Chocolatey\lib\Cygwin\tools\cygwin\home\<current_user>\ubuntu
cd ubuntu/
vagrant init hashicorp/precise32
vagrant up

vagrant ssh
	now we are at the command line of Ubuntu
uname
exit  -to end ssh session

vboxmanage list runningvms
vboxmanage list vms

vagrant be default runs headless
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
to hibernate use: 
vagrant suspend
vagrant resume
vagrant ssh
	fast to resume but uses more memory

to shutdown use:
vagrant halt

to restart use:
vagrant up
vagrant ssh

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

mkdir nginx
cd nginx

vagrant init hashicorp/precise32 --minimal

open Vagrantfile
 can edit config
 
vagrant up
ssh
exit

Each project will have a Vagrantfile committed to version control
git init

Base image is called a box

to install or configure use
Provisioning:
add to Vagrantfile and save:
  congig.vm.provision "shell", path: "provision.sh"

add commands to the sh file and save.

vagrant reload //same as vagrant halt and vagrant up
  //this will not rerun the provisioning since it is only run on the first vagrant up

vagrant provision

vagrant ssh
service nginx status
wget -qO- localhost

default shared folder is located in /vagrant on guest 

Provisioning Benefits:
knowledge of environment setup is retained
Easily recreate environment
Test environment same as production
Can commit setup to version control
Can rollback to last known working configuration if something goes wrong

if you share your vm publicly make sure to overwrite the default ssh that comes with the box in your provisioning

