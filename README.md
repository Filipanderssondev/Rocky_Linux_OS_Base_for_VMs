# Rocky Linux OS base for VMs
<img alt="" src="https://github.com/Filipanderssondev/Rocky_Linux_OS_Base_for_VMs/blob/main/Images/rocky-linux-os-vm-base.png" allign=left><br>

**Rocky Linux OS base for VMs** <br>
**Authors:** _<a href="https://github.com/Filipanderssondev">Filip Andersson</a> and <a href="https://github.com/JonatanHogild">Jonatan Högild</a>_
01-13-2026<br>


## Abstract
Creating and configuring a Rocky Linux golden image OS base for virtual machines on a Proxmox server. <br>

## Table of Contents

1. [Introduction](#introduction)
2. [Goals and Objectives](#goals-and-objectives)
3. [Method](#method) <br>
4. [Target Audience](#target-audience)
5. [Document Status](#document-status)
6. [Disclaimer](#disclaimer)
7. [Scope and Limitations](#scope-and-limitations)
8. [Environment](#environment)
9. [Acknowledgments](#acknowledgments)
10. [Implementation](#implementation) <br>
    	10.1 [Download and verify Rocky Linux](#download-and-verify-rocky-linux) <br>
		10.2 [Create a Rocky Linux VM](#create-a-rocky-linux-vm)<br>
		10.3 [Configure Rocky Linux](#configure-rocky-linux)<br>
		10.4 [Firewall Configuration](#firewall-configuration)<br>
		10.5 [Cleaning up and finishing](#cleaning-up-and-finishing)<br>
11. [Conclusion](#conclusion)
12. [References](#references) <br>
    12.1 [Other projects in our virtual IT-enviroment](#other-projects-in-our-virtual-it-enviroment)

## Introduction
**Welcome!** <br> 
This project is about how to configure a Rocky Linux golden image to serve as a minimal operating system base for virtual machines. This project is the second <a href="https://github.com/rafaelurrutiasilva/Proxmox_on_Nuc/blob/main/Extra/Mermaid/Projects.md">in a series of projects</a>, with the goal of setting up a complete virtualized, automated, and monitored IT-Enviroment as a part of our internship at [The Swedish Meteorological and Hydrological Institute (SMHI)](https://www.smhi.se/en/about-smhi). Previously, <a href=https://github.com/rafaelurrutiasilva/Proxmox_on_Nuc>Proxmox was installed and configured</a> on a server. Here, we will prepare a Rocky Linux golden image for cloning. A <a href=https://www.redhat.com/en/topics/linux/what-is-a-golden-image>golden image</a> serves as a baseline template for replication, reducing repetitive setup and ensuring consistency. When ready, a template will be created from the Rocky Linux image, and then cloned. <br>

_[Other projects in our virtual IT-enviroment](#other-projects-in-our-virtual-it-enviroment)_

## Goals and Objectives
The goal of this project is to create a golden image that is suitable for replication across a production environment. The image should be lightweight, secure and ready for replication. 

## Method
An official Rocky Linux cloud image was downloaded and configured as a VM for Proxmox. On the VM, basic system settings were configured, package sources were updated, users were added, and utilities were downloaded. Firewall rules were set up to allow only certain traffic. Finally, machine-specific data was cleaned, and the VM was converted into a template used to clone the remaining virtual machines in the environment.

## Target Audience
This repo is for anyone who wants a step-by-step guide on preparing a Rocky Linux golden image for Proxmox. 
This repo is also part of a larger project aimed at people interested in learning about IT-infrastructure, and building such an environment from scratch. 

## Document Status
This repository is considered complete and officially published.<br>
Future improvements, refinements, or corrections may be introduced through controlled updates. Any changes will be versioned and documented in the commit history.

## Disclaimer
> [!CAUTION]
> This is intended for learning, testing, and experimentation. The emphasis is not on security or creating an operational environment suitable for production.

## Scope and Limitations
* Instructions for installing and configuring Rocky Linux as a golden image.
* Instructions for how to work within Proxmox VE (9.1.1), create and manage VMs.
  
* This guide is not intended for production-grade, multi-node clusters or advanced HA setups.
* Hardware compatibility varies; If unsure, check <a href=https://docs.rockylinux.org/10/guides/minimum_hardware_requirements>hardware requirements</a> before proceeding. 
* Instructions may become outdated as software updates; always verify with the official documentation.
* Sensetive information will be withheld. This will not hinder participation in the guide.

## Environment
 - Asus PN64 ax210NGW
   - Intel® Core™ i7-12700H 
   - 1TB disk
   - 64 GB memory
 - Proxmox VE (9.1.1)
 - Rocky Linux (10.1)

## Acknowledgments
We would like to thank <a href=https://github.com/rafaelurrutiasilva>Rafael Urrutia</a> for his continuous support and guidance.

## Implementation

### Download and verify Rocky Linux

Instead of a regular .ISO file, we chose the generic cloud image file (.qcow2), since this is optimal for environments like Proxmox, and ideal to make templates from. 
The current version of Rocky Linux is v10.1 and it's downloaded from the <a href=https://rockylinux.org/download>official site</a>.
Also download the CHECKSUM file.

Open a terminal and go to the download location: 
```
cd ./Downloads
```

#### Compute the hash

This can be done with: 
```
sha256sum Rocky-10-GenericCloud-Base.latest.x86_64.qcow2
```

Confirm with: 
```
sha256sum -c CHECKSUM | grep OK
```

### Create a Rocky Linux VM

#### Add the cloud image file

In the Proxmox web-GUI, go to Datacenter > Storage > Local <br>
Add *Import* to Content list.

Then go to Node > Local > Import <br>
Upload the cloud image. 

#### Create a new VM

Now create a VM by clicking the blue button *Create VM* in the topright corner. 

settings we used:<br>
<pre>
General:
	Name: rocky-base
	Add to HA: No
	Start at boot: No
	
OS:
	Type: Linux
	Version: 6.x - 2.6 Kernel
	Do not use any media
	
System:
	Graphic Card: Default
	Machine: q35
	BIOS: OVMF (UEFI)
	Add EFI Disk: Yes
	EFI Storage: local-lvm
	Pre-Enroll keys: yes
	SCSI ontroller: VirtiO SCSI single
	
Disks:
	No Disks

CPU:
	Sockets: 1
	Cores: 14
	Type: host

Memory:
	Memory (MiB): 4096
	Ballooning Device: No
	Allow KSM: Yes

Network:
	Bridge: vmbr0
	Model: VirtIO (paravirtualized)
	Vlan Tag: No
	MAC address: Use the autogenerated address
	Firewall: Yes
</pre>

These settings are preliminary and may be changed. Our server has an Intel® Core™ i7-12700H Processor with 14 cores, 64GB of RAM and 1TB of storage. Assigning all 14 cores to every VM may cause contention, so we'll monitor this. The lightweight version of Rocky Linux <a href=https://docs.rockylinux.org/10/guides/minimum_hardware_requirements>requires about 1GB of RAM</a>, so 4GB should be enough for a single VM. Hard disk size is set to 10GB by default. The size may be increased later on, and is evaluated on a per-VM basis.


#### Import Hard Disk

Go to the new VM > Hardware > Add > Import Hard Disk <br>
Important Storage: local <br>
Select Image: Rocky-10-GenericCloud-Base.latest.x86_64.qcow2 <br>
Target Storage: local-lvm

Also Add > CloudInit Drive

#### Cloud-init settings

Go to the Cloud-Init menu for the VM <br>
Choose a username and password

In IP Config (net0): <br>
We set static ip addresses for our VM, but go with whatever works best for you.

Go to the Options menu for the VM <br>
Change the boot order: <br>
1. scsi0 <br>
2. ide2 <br>
3. net0 <br>

Start the VM and log in.


### Configure Rocky Linux

#### Keyboard and Timezone

Swedish keyboard layout: 
```
sudo localectl set-keymap se
```

Change Timezone: 
```
sudo timedatectl set-timezone Europe/Stockholm
```

#### Update sources

Before updating, we change our repo sources to use a mirror provided by NSC: *mirror.nsc.liu.se* (https/433).
This step is not necessary if you can run *dnf update* directly. For our project, we must request which resoruces we want to access over the Internet, and this is our prefered source. 

In */etc/yum.sources.d*, there are a couple of repos that can be changed. We change them with: 
```
vi /etc/yum.repos.d/rocky.repo
```

Comment out the lines beginning with mirrorlist, and replace *http://dl.rockylinux.org* with *https://mirror.nsc.liu.se* in baseurl. Also remove the comment from the baseurl lines. Save and update. 

#### Install additional programs

The Rocky Linux cloud image is purposefully minimal. Though there are some programs we'll want available for every clone, and will install here. After running an update, we installed the following: <br>
- bind-utils <br>
- bash-completion <br>
- tcpdump <br>
- traceroute <br>
- python3-pip <br>
- unzip <br>
- zip <br>
- nmap <br>
- tmux <br>

#### Change console font and size

We felt that the default console font size and color could use some improvements. on Rocky Linux, fonts can be changed in the */etc/vconsole.conf* file:
```
sudo vi /etc/vconsole.conf
```

We added the following line: 
```
FONT=sun12x22.psfu.gz
```

Colors can be added in a profile.d script, which will execute on each login:
```
#!/bin/sh
if [[ "$(tty | grep -c tty)" -gt 0 ]]; then
 setterm -foreground green
 setterm -background black
fi
```

#### Add users

We added new users for each of us and placed these in the wheel group:
```bash
useradd jonatan
useradd Filip
usermod -aG wheel jonatan
usermod -aG wheel Filip
```

### Firewall Configuration

There is a firewall at every layer in Proxmox (datacenter > node > virtual machine). At the datacenter level, security groups, aliases and IPsets can be created. A security group is a grouping of rules, which can then be quickly applied to nodes and virtual machines. An IP set groups networks and hosts, which can then be added as source and destination properties for firewall rules.

This firewall will be designed to be only as permissive as it needs to be. Initially, we'll identify which protocols we need to use, and make rules for these. As the project evolves, so will the firewall, and new rules will be added later on. 

#### SSH

Go to Datacenter > Firewall > Security Group
Create a new security group, call it something like *allow-ssh* with the following configuration:<pre>
Direction: in
Action: ACCEPT
Enable: Yes
Macro: SSH
Log level: info</pre>

Set logging to whichever level you prefer, or keep it turned off if you like. *Debug* may be a bit too verbose, so we chose *info* as our baseline. 

We'll also make a copy of the rule, and set its direction to *out*.

Specifications for macros can be found <a href=https://github.com/proxmox/pve-docs/blob/master/pve-firewall-macros.adoc>here</a>.

#### ICMP

Next, we'll make a new security group for ICMP. There is no macro for ICMP, so it must be selected in the protocol field:<pre>
Direction: in
Action: ACCEPT
Enable: Yes
Protocol: icmp
Log level: info</pre>

ICMP type can be specified. We'll allow the following types: 
- echo-reply
- destination-unreachable
- echo-request
- time-exceeded <br>

These rules allow for troubleshooting, without being too permissive. We'll also add the same rules for IPv6-ICMP. Next, make a copy of each rule, and change its direction to *out*. In total, there will be 16 ICMP rules. 

Create a new security group, and call it something like *allow-ipv6*. This group will contain rules for IPv6-ICMP types that enable basic IPv6 functionality. The following types will be allowed: 
- packet too big (2) for both directions 
- router solicitation (133) out
- router advertisment (134) in
- neighbour solicitation (135) out
- neighbour advertisment (136) in

#### DNS

Go to Datacenter > Firewall > IPSet
Create a new IP set and call it dns. Add the IP-address of your DNS-server. 

Create a security group for DNS with the following rule:<pre>
Direction: out
Action: ACCEPT
Enable: Yes
Macro: DNS
Destination +dns
Log level: info</pre>

#### Web 

We create a new security group for web, and add a new rule:<pre>
Direction: in
Action: ACCEPT
Enable: Yes
Macro: Web
Log level: info</pre>

Note that this macro allows both HTTP and HTTPS. Consider if a second rule for outbound web traffic will be necessary. For our lab, it will be, so we'll add it.

#### NTP

Security group for NTP, with the rule:<pre>
Direction: out
Action: ACCEPT
Enable: Yes
Macro: NTP
Log level: info</pre>

#### Block all other traffic

The last security group will block everything else, call it something like *drop-everything* and make two new rules:<pre>
Direction: in
Action: Drop
Enable: Yes
Log level: info</pre>

<pre>
Direction: out
Action: Drop
Enable: Yes
Log level: info</pre>

These rules works as a catch-all, and must be set as the last in the rule-matching order. It might be worth considering wheter to use *drop* or *reject*. 
Reject usually gives instant feedback (connection refused instead of timeout) and is more convenient for a lab environment. Drop, however, is less prone to leak information.

#### Set up Firewall 

Add the security-groups to the rocky-base VM. The order we selected is: <br>
1) allow-icmp <br>
2) allow-ipv6 <br>
3) allow-ssh <br>
4) allow-dns <br>
5) allow-ntp <br>
6) allow-web <br>
7) drop-everything <br>

For further hardening, configure the firewall options on the VM level: <pre>
Firewall - on
DHCP - off (we're not using DHCP in this lab)
NDP - on (necessary for IPv6 functionality)
Router Advertisment - on (also used by IPv6)
MAC filter - on (prevents MAC-address spoofing)
IP filter - off (prevents IP-address spoofing, causes issues so it's turned off)
log_level_in - info
log_level_out - info
Input Policy - DROP (drops traffic when no rule matches, does what the drop-everything-rule does)
Output Policy - DROP </pre>

Double-check that the firewall is enabled. If it's disabled at one level, it will also be disabled at every lower level.

Go into the VM to confirm that the rules work. Try commands like ping, ssh, curl, dig, nc and nmap.

### Cleaning up and finishing

The VM is almost ready to be copied. One final thing to do is cleaning up temporary and machine-specific files.

#### Clear DNF/YUN cache, metadata and tmp files

Package-manager leftovers can be cleared with this command: 
```
sudo dnf clean all
```

#### Temporary files

Remove any files left in temporary folders:
```bash
sudo rm -rf /tmp/*
sudo rm -rf /var/tmp/*
```

#### Machine ID

Machine-id is autogenerated on installation/boot. It's used by systemd, d-bus, networkmanager, and sometimes licenses and UUID-based apps:
```bash
sudo truncate -s 0 /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
```

#### Shell history

Not strictly necessary to remove, but command history could reveal sensetive information. 
```bash
history -c  
rm -f ~/.bash_history
```

#### Create Template and clones 

Shut down the VM, then convert it to a template. This template can now be easily cloned. We'll make 3 clones: 
- mgmt-01
- metrics-01
- app-01

We recommend using the *Linked Clone* mode, for potential performance gain. 

The clones will be given new VM IDs, and new static IP addresses. That's all the setup needed in the cloning process. 

## Conclusion
The aim of this project was to prepare a Rocky Linux install to use as a golden image. We strived to keep it minimal, yet include a set of binaries that will be useful throughout the project. This project also helped us further explore Proxmox and virtualization, and we have become more familiar with these as a result. 

## References
- [SMHI](https://www.smhi.se/en/about-smhi)
- [What is a golden image](https://www.redhat.com/en/topics/linux/what-is-a-golden-image )
- [Rocky Linux Download Page](https://rockylinux.org/download)
- [Rocky Linux hardware requirements](https://docs.rockylinux.org/10/guides/minimum_hardware_requirements)
- [Proxmox Firewall Macros](https://github.com/proxmox/pve-docs/blob/master/pve-firewall-macros.adoc)
<br>

### Other projects in our virtual IT-enviroment:
- Project 1 - [Proxmox on Nuc](https://github.com/rafaelurrutiasilva/Proxmox_on_Nuc/)
- Project 3 - [Ansible on management VM](https://github.com/JonatanHogild/Ansible_on_management_vm)
- Project 4 - [Container stack deployment and monitoring with ansible](https://github.com/Filipanderssondev/Container_Stack_Deployment_With_Ansible)
- Project 5 - [FreeIPA for Virtual Enviroment](https://github.com/JonatanHogild/FreeIPA_for_virtual_environment/)

