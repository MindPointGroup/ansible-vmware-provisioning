# ansible-vmware-provisioning

## Background
This repo has been developed in support of Matt Shepherd's (matts-mpg) presentation at AnsibleFest San Francisco 2017- Improving Provisioning in VMware with Ansible. Fundamentally, the goal of the talk and this repo is to help teams "do VMware" better by leveraging Ansible for provisioning. The reasons Matt felt this talk & repo are "necessary" are:
* Looking through Ansible Galaxy, there are almost no roles for VMware (like less than a 10);
* Looking through GitHub there are only 153 YAML repos referencing "vmware_guest;"
* Some of the core Ansible modules related to VMware have undergone significant change in the last ~12 months;
* Personal experience has shown that because VMware has been around forever (in technology years) many teams operating/managing VMware are using legacy processes and tools to do so; and
* VMware is widely adopted, and in some environments there are requirements to use internal virtualization farms versus migrating to public cloud.

So, given the dearth of Ansible content available for VMware, the fact that VMware has a large deployment footprint, and the fact that most VMware farms are managed using legacy tools and practices, it seemed that this is an area where showing people a better way to do things can have a big impact and is sorely needed. This is the impetus behind Matt's talk and this repo.

## Plays
### provisioning.yml
This play does just what it says- it creates a new VM from a template in vCenter. However, that single task doesn't get us to a complete, fully deployed VM that we can start applying Ansible roles to so it does a couple things after creating the VM to put a nice bow on it, and to allow you to go straight into your further configuration without requiring a bunch of different actions on the part of the ops team. These "clean-up items" include:
* Setting the desired IP address for the host;
* Applying your common role (we do this in particular so that we can get a specific python library installed); and
* Taking the actions necessary to get the disk resized properly.

### disk_resize.yml
One thing that can really hinder VMware deployments is that unlike AWS where resizing an EBS volume automagically includes doing all of the things necessary for the full, expanded disk to be accessible immediately by the OS, you need to do a bit more in a VMware VM. So, when deploying one or many VMs you can be doing those tasks manually a number of times, and it is time-consuming and error-prone. In addition, for those adhering to a secure hardening guideline that requires establishing discrete partitions for your mount points, you need to resize each of those individually.

The same issue applies to existing VMs where you want to grow a disk, and this play can be used in those cases as well so while it's almost a requirement to use at deploy time it can be useful later to give your VMware environment that same automagic goodness that comes with resizing a volume in AWS.

NOTE: Above we note that our common role is applied prior to running this disk resize play. Part of the reason for that is that we need to use fdisk here and we've learned the hard way that it's not possible to use fdisk non-interactively. So, we need to install Python's pexpect library on all our hosts before being able to use Ansible's expect module in the first task of this play.

## Variables
The following sample of an abridged host_vars file for a host we would install Confluence on with the relevant variables for our two plays shown. The name var just gives our VM a name in the VMware host inventory (as well as being useful for setting the hostname in the OS). Then there are a couple vars that define the hardware we want to provision the VM with, and then the IP we need to assign. Below that you can see vars that are specific to being able to resize our partitions to use the expanded space we will have available. Doing the math from the comments in the mount size definitions you can see the size of the disk in the VM template is 17 GB, and we are provisioning this host with a disk that is 231 GB.
```yaml
---
# Confluence

name: p-atl-rhel7-02
cpu: 8
mem_mb: 24576
disk: 231
server_ip: 192.168.1.22
....
next_part: '3'
mount:
  - part: /dev/rhel/root
    size: 10G    # +8G
  - part: /dev/rhel/opt
    size: 30G    # +28G
  - part: /dev/rhel/home
    size: 20G    # +15G
  - part: /dev/rhel/tmp
    size: 10G    # +8G
  - part: /dev/rhel/var
    size: 100G   # +98G
  - part: /dev/rhel/var_log
    size: 30G    # +28G
  - part: /dev/rhel/var_log_audit
    size: 30G    # +28G
```

## License
MIT

## Author Information
Matt Shepherd aka matts-mpg<br>
Vice President at MindPoint Group
