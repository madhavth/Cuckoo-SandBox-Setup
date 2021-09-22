# Cuckoo-SandBox-Setup

# **Cuckoo Sandbox Installation Guide**

# 1. Host system minimum requirements

The following describes the requirements for installing Cuckoo on the system.

- VMWare ESXi
- Debian 10.0 Buster
- 8GB Ram
- At least 100GB memory
- Virtualbox 6.0
- Cuckoo Sandbox v2.0.7

More processing power and ram is better, at least 16 GB Ram on the host machine is recommended from what I experienced building it.

Make sure that virtualization has been enabled and nested virtualization is supported before moving forward.

# 2. Cuckoo and VMCloak installation

**Bring the system up to date**:

- sudo apt-get update 
- sudo apt-get upgrade -y


**Installation of Python libraries which are necessary for VMCloak and Cuckoo:**

- sudo apt-get install python3 python3-pip python-dev libffi-dev libssl-dev -y 
- sudo apt-get install libjpeg-dev zlib1g-dev swig -y
- sudo apt-get install python3-virtualenv python-virtualenv -y


**Adding the user Cuckoo:**

For security reasons, malware analysis is performed on a low privileged user (cuckoo).

- sudo adduser cuckoo


**Installation of the required database:**

- sudo apt-get install mongodb -y


**Installation and configuration of TCP-Dump as a tool for network analysis:**

- sudo apt-get install tcpdump apparmor-utils -y
- sudo aa-disable /usr/sbin/tcpdump


**Installation of Virtualbox:**
- sudo apt-get install virtualbox -y


**Add the Cuckoo user to groups:**

- sudo usermod -a -G vboxusers cuckoo
- sudo groupadd pcap
- sudo usermod -a -G pcap cuckoo
- sudo chgrp pcap /usr/sbin/tcpdump
- sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

**Create a virtual environment:**

- sudo su cuckoo
- virtualenv -p python2.7 ~/cuckoo
- source ~/cuckoo/bin/activate

*The virtual environment can be left with the command exit.*


**Install Cuckoo and VMCloak within the virtual cuckoo environment:**

- pip install -U cuckoo vmcloak

# **2.1. pip installation error**
If you encounter an error stating that python 2.7 is required then do the following, install python2, get pip module for python2.

- sudo apt install python2

Check if pip module is installed for python2, by running:
- python2 -m pip

If it returns with an error, then check the following guide for installing python2 pip module, check

https://stackoverflow.com/questions/21305524/how-to-install-pip-for-python-2

- python2 -m pip install -U cuckoo vmcloak

If the issue is now resolved and packages are installed properly, then proceed with the following commands.

- cuckoo init
- cuckoo community --force
 


# **3. Creation of the Base-Virtual Machine**

**Download Windows ISO file**

- sudo wget https://cuckoo.sh/win7ultimate.iso


**Directory Mounting**

- sudo mkdir /mnt/win7
- sudo chown cuckoo:cuckoo /mnt/win7
- sudo mount -o ro,loop win7ultimate.iso /mnt/win7


*Make sure you are in the virtual environment and are in session with user cuckoo in the terminal. If not then switch to the virtual environment.*

- . ~/cuckoo/bin/activate


**Configuring VirtualBox Host-Only Network Adapter**

- vmcloak-vboxnet0

we will need this later.


**Create the default virtual machine**

Creating the default virtual machine
The Default VM serves as a basis for the creation of future virtual analysis machines. Thus, future VMs do not have to be created first, but can be simply cloned from the base VM quickly.
The syntax is as follows:

- vmcloak init --verbose --win7x64 win7x64Base --cpus 2 --ramsize 2048

*change the settings as per required, if you want faster system put ram to around 8096 and cpus to at least 4.*

# **4. Creation of analysis systems**

The preferred way to add a new analysis machine to the Cuckoo Sandbox is cloning. This ensures that the default configurations such as the agent and network parameters are applied appropriately. This allows a fast and uncomplicated start of a new analysis system.

Cloning of a machine using Vmcloak
The Vmcloack tool is used to clone a VM. The syntax for cloning a virtual disk image (VDI) from an existing one is as follows:

- vmcloak clone presentVDI newVDI
- vmcloak clone win7x64 win7x64cuckoo

# **5. Software Installation on analysis systems**
The installation of additional software on the analysis systems is not necessary. However, in order to simulate the system environment of a user as realistically as possible, it is highly recommended. Malware often works/spreads in dependence of third party software. For this reason, standard software applications such as office products, readers and browsers are installed. Additionally, parameters such as activation of macros in Office products can be used to make the system even more vulnerable to malware.

A software package can be installed using the following syntax:
- vmcloak install \<image name> \<package>

Install these basic packages for now:-

- vmcloak install win10x64Cuckoo1 adobe9 wic pillow java7

# **6. Creation Of Snapshot**
After creating the analysis system, a snapshot of the analysis system has to be created. The syntax for creating snapshots is as follows:
- vmcloak snapshot win7x64cuckoo cuckoo1 192.168.56.101 --debug --ramsize 2048 --cpus 2

# **6.1 Workaround for "Cannot Multi-attach media"**
If the following error occurs Cannot change type for medium: the media type “Multiattach”, Apply this workaround. If the snapshot was created without causing problems, this section can be skipped.

<img src= "https://t0xicity.com/img/cuckoo_sandbox/errorMultiattatch.png" alt ="cuckoo error message">

The following steps are necessary for the workaround:
Create a temporary “Throw-Away” machine

<img src ="https://t0xicity.com/img/cuckoo_sandbox/throw_away.png" alt="image throw_away machine setup">

When selecting the disk image, select the Analysis VM.

<img src="https://t0xicity.com/img/cuckoo_sandbox/selectVM.png" alt= "hard disk seelction">

Open the virtual media manager in VirtualBox:

<img src="https://t0xicity.com/img/cuckoo_sandbox/openMediaM.png">

Select disk image of the previously created Throw_away VM in Media Manager:

<img src="https://t0xicity.com/img/cuckoo_sandbox/selectThrowAway.png">


Set Disk-Type to Multi-attach: Under the Properties tab in the Type -> Select Multi-attach selection, click Apply.

<img src="https://t0xicity.com/img/cuckoo_sandbox/vm_properties.png">

Since the virtual disk image now has the multi-attach flag set, we can add it to VirtualBox as a snapshot using Vmcloak as described in the previous section. To do so execute the following command:

- vmcloak snapshot win7x64cuckoo cuckoo1 192.168.56.101 --debug --ramsize 2048 --cpus 2 

*use more cpu cores and ram size to avoid potential issues, optimal ramsize would be 8 gb and cpu core at least 4.*

**Afterwards the throw_away VM can be deleted.**

# **6.2 Workaround for snapshot creation error**
If no error occurred during snapshot creation, then this sectioon can be skipped.

If there is an error while creating snapshot, and the errors falls under one of the two categories then it can be handled as shown below:

-------------------

**1. timeout error waiting for connection**
If there is a timeout error, it means that the newly created cuckoo1 machine didn't boot in time, open virtualbox delete the machine named cuckoo1 and try again with increased ramsize and more cpu cores.

---------------------------------------------
**2. any other exception returning in failure, this part can be skipped if vmcloak snapshot creation went smoothly.**

create snapshot manually using vboxmanage
- vboxmanage snapshot "cuckoo1" take "cuckoo1" --pause
- vboxmanage controlvm "cuckoo1" poweroff
- vboxmanage snapshot "cuckoo1" restorecurrent


**add the machine to cuckoo virtualbox config file by using the command:**

- cuckoo machine --add cuckoo1 192.168.56.101

**To install the Cuckoo signatures and latest monitor, we run the following command:**

- cuckoo community --force
-----------------------

# **7. Network configuration**

for network configuration, follow the Network Configuration guide from https://hatching.io/blog/cuckoo-sandbox-setup/, the "Using uWSGI and nginx - Advanced" section can be ignored and any error when running "cuckoo rooter" can be ignored for now.


# **8. Starting Cuckoo**

If everything has been configured properly, then cuckoo should be up and running now.

run
- cuckoo --debug


If it doesn't return with any error, then cuckoo is working properly. Now, keep this terminal session running and open up another terminal.

Enter the virtual environment by using the following command:
- source ~/cuckoo/bin/activate

After entering the virtual environment, now run cuckoo web interface or submit files using the cuckoo submit command. Check the documentation for more info regarding submitting with through the cuckoo cli.

For running the web interface, use the following command.
- cuckoo web --host 127.0.0.1 --port 8080

Now, if there isn't any errors then cuckoo web interface should load up now enter the url in the browser to access the cuckoo web interface.

- 127.0.0.1:8000
