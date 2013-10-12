---
layout: post
title: VBoxManage - Command line options to handle Oracle VMs
last_changed: 2013-06-18
tags: VBoxManage shellscript headless
---

Ahhh.. It's been almost one year since my last post.  Time to add one :)  
<br/>
In this post I share a command line script to control virtual machines using VBoxManage.  
<br/>
VBoxManage is a Oracle VM VirtualBox command line management interface. Using VBoxManage you can achieve all functionality using command line. Simple command `VBoxManage list vms` shows the existing VMs. Basic command structure is the VBoxManage command options. You can create a new machines, add hard disk, change the memory settings, get in formations using different VBoxManage commands. For more help type `VBoxManage -h`.  
<br/>
### Q. Okay that's cool. But Why do you need a script to control VMs?

I am having 4 or 5 VMs in my systems with minimal Linux installation. Each one is dedicated to specific development like one is for kernel development, another one is for egroupware testing. I usually start my VMs, create a ssh connection and do my work. I don't use DMs. So what I need is a running VM in background in which I can login. Virtual Box provides VBoxHeadless frond-end to achieve this. VBoxHeadless produces no visible output on the host at all.  

There are 3 ways to start a VM with VBoxHeadless frond-end:

1. Hold the shift key while starting a VM.
2. `VBoxManage startvm "VM name" --type headless`
3. `VBoxHeadless --startvm "VM name"`  

I wrote a simple shell script to start/stop VMs.
You can find it [here](http://www.github.com/sanketsparmar/) (still under development).  

### How to use:
1. Change the execution permission using following command

		chmod u+x vbox

2. To list all VMs and its status

		lucky@rico:~/bin$ vbox
		1. XP ---- "poweroff"
		2. Lubuntu ---- "running"
		3. kernel development ---- "poweroff"
		4. webfilter_dev ---- "saved"
		5. egroupware testing ---- "poweroff"
		6. egroupware final ubunut 11.10 ---- "poweroff"

3. To start VM, Just provide the index number in the script.(Default mode is headless)

		lucky@rico:~/bin$ vbox 1 on
		VBoxManage startvm  ecaf2c9c-75af-42ca-b02f-8329d1da9d45 --type headless
		Waiting for VM "ecaf2c9c-75af-42ca-b02f-8329d1da9d45" to power on...
		VM "ecaf2c9c-75af-42ca-b02f-8329d1da9d45" has been successfully started.

4. To start VM with GUI, Add provide 1 as third argument

		lucky@rico:~/bin$ vbox 1 on 1
		VBoxManage startvm  ecaf2c9c-75af-42ca-b02f-8329d1da9d45 
		Waiting for VM "ecaf2c9c-75af-42ca-b02f-8329d1da9d45" to power on...
		VM "ecaf2c9c-75af-42ca-b02f-8329d1da9d45" has been successfully started.

5. To stop VM

		lucky@rico:~/bin$ vbox 2 off
		VBoxManage controlvm  c4c83e22-07fd-456e-815d-eef1088d581e poweroff
		0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%

6. To start all VMs

		lucky@rico:~/bin$ vbox all on

7. To stp all VMs

		lucky@rico:~/bin$ vbox all off

This script also fetch the VM's description. Generally I write machine's IP, username and password in description.

	lucky@rico:~/bin$ vbox 3 on
	Description:
	username: user1 password: XYZ ip: 192.168.4.232 For kernel development; Ubuntu 11.10 32bit
	>VBoxManage startvm  14a23ab2-d34b-41b1-a816-5cc08858712a --type headless<
	Waiting for VM "14a23ab2-d34b-41b1-a816-5cc08858712a" to power on...
	VM "14a23ab2-d34b-41b1-a816-5cc08858712a" has been successfully started.

Now create ssh connection and start working :)
