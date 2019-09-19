Virtual Redfish BMC
Redfish Simulation Emulator 
After reviewing a few Redfish simulation tools, our choice, moving forward, was a tool called sushy-tools developed by the Openstack community (https://docs.openstack.org/sushy-tools/latest/).  This tool simulates the Redfish protocols and provides the development community with independent access and testing of the Redfish protocol implementations.  This tool is actively being enhanced and provides support for uefi boot.  As such, one might encounter temporary hiccups with the code if one tries to use the latest code, thus we provide the git commit sha1 for the code we tested in the prerequisites sections that follows, 
Redfish Simulation Emulator Installation 
Installation Prerequisites
Before we begin the installation there are a few prerequisites that should be considered, such as: 
•	Are you installing the sushy-tools directly on an existing node or host or hosting the tool inside a virtual machine or instance? 
•	While is possible to host the tools on Windows, our assumption and favored choice was to install on a Linux (CentOS or RHEL) with qemu/libvirtd support
•	Hardware is a matter of choice as longs as the hardware supports virtual systems.
Tested Hardware and Software
For our development purposes and based on what was available to us at the time, we selected and tested the following hardware and software.
•	Dell R640 PowerEdge servers with the following accessories: 
•	R640 Server with dual Intel Xeon 6126 2.6GHz CPUs
•	192GB - 2666Mhz RAM
•	800GB  RAID10 storage
•	10GB bonded Intel NIC
•	CentOS 7.6 Base Operating System
•	Major CentOS group packages included the core, standard, development-tools
•	Other rpm packages installed included: net-tools, zip, unzip, git, qemu and libvirtd with requisites
Installation
As indicated above in the installation prerequisites, one has the option of installing sushy-tools directly on the host system or withing a virtual machine hosted by the host system.  We went with the later, simply because it was easier to tear down and build up experimental redfish simulation engine without polluting our host node.  Below our the steps we followed. 
Building the Redfish VM on a host system
Get the files to create the VM.
1.	Download the Fedora 30 Server image from https://getfedora.org/en/server/download/
2.	Download the Redfish_tools.zip file and build the redfish emulator VM (see attached file) Redfish_tools.zip
3.	Scp the redfish_tools zip to the host machine.   Note: The host machine must be capable of running a qemu libvirt Instance/VM.
4.	Extract the files from the zip image (unzip redfish_tools.zip)
5.	Modify the redfish.cfg file.
a.	Make the appropriate changes for your domain / network ( values to change listed below )
rootpassword calvin
timezone UTC
hostname redfish.oss.labs
gateway 100.82.32.129
nameserver 100.82.32.10
ntpserver 0.centos.pool.ntp.org
# CHANGEME: Change the IP and netmask below to the IP address and netmask for
# the Redfish Admin VM on the Public API network
# Iface IP NETMASK MTU
ens3 100.82.32.164 255.255.255.192 1500
6.	Run the deployment script to deploy the VM using the cfg file and the path to fedora image that you downloaded earlier.
a.	./deploy-redfish-vm.py redfish.cfg /tmp/Fedora-Server-dvd-x86_64-30-1.2.iso
7.	 Optional -- You can watch the VM deploy using virt-viewer (if you previously installed the virt-viewer yum package on your hosts system and have X windows installed)
a.	virt-viewer redfish
8.	 When the VM has finished installing, start the VM 
a.	virsh start redfish
9.	 SSH into the VM using the IP address assigned in step 5.
a.	ssh root@100.82.32.164  
Configure and Install the Sushy-emulator to the host VM 
1.	git clone https://opendev.org/openstack/sushy-tools.git
2.	cd sushy-tools/
3.	python3 setup.py build
4.	python3 setup.py install
Upload the redfish.d and emulator.conf files and add them to the VM
1.	scp localsystem//redfishd.service root@redfish_vm_ip://tmp
2.	scp localsystem://emulator.conf root@redfish_vm_ip://tmp
3.	vi /tmp/redfishd.service #adjust file for the redfish_vm_ip
4.	vi /tmp/emulator.conf #adjust file for the redfish_vm_ip
5.	mkdir -p /etc/redfish
6.	cp /tmp/emulator.conf /etc/redfish/
7.	cp /tmp/redfishd.service /etc/systemd/system
8.	systemctl start redfishd
9.	systemctl status redfishd
10.	systemctl enable redfishd
Build the vbmc-node
1.	tmpfile=$(mktemp /tmp/sushy-domain.XXXXXX)
2.	virt-install --name vbmc-node --ram 1024 --boot uefi --disk size=1 --vcpus 2 --os-type linux --os-variant fedora28 --graphics vnc --print-xml > $tmpfile
3.	virsh define --file $tmpfile
4.	rm $tmpfile
Verify the sushy emulator is working and the vbmc-node was added
1.	curl -L http://100.82.39.164:8000/redfish/v1/Systems
2.	curl -L http://100.82.39.164:8000/redfish/v1/Systems/b5588a3b-f489-4760-b49b-ca04db062713
Some helpful links
•	Openstack Sushy Stories (suppport) (https://storyboard.openstack.org/#!/story/list)
•	Installing Sushy-tools (redfish) (https://docs.openstack.org/sushy-tools/latest/)

