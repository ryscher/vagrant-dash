vagrant-dryad
=============

Vagrant and Ansible config for building a Dryad VM

Uses Ubuntu 12.04 64-bit, Ansible, Postgres, Java6, Tomcat6

See [How to install Dryad](http://wiki.datadryad.org/How_To_Install_Dryad#Building_a_Virtual_Machine_with_Vagrant) for background/details

## Requirements

These applications must be installed on your host machine.  They will be used to build and run a virtual machine containing Dryad.

1. [Vagrant](http://vagrantup.com)
2. [Ansible](http://ansible.com)
3. [VirtualBox](http://virtualbox.org)

Vagrant and VirtualBox installation packages can be downloaded from their respective websites.  Ansible is a Python package with [many ways to install](http://docs.ansible.com/intro_installation.html).  I use [Homebrew](http://brew.sh) (which, in turn, requires ruby) and simply `brew install ansible`.

These packages are available on many platforms, but I've only tested this on Mac OS X 10.9.

## Getting started

First, You will need to clone the repository:

    git clone git@github.com:datadryad/vagrant-dryad.git
    cd vagrant-dryad

## Customizing

### Vagrantfile (Port forwarding)

There are two files to examine before continuing. The Vagrantfile controls VM parameters such as IP addresses and ports.

Ports 8000 and 9999 are configured by default.  8000 is for JPDA debugging if you wish to connect a remote debugger.  9999 is the default port for Dryad development hosts. You can change 9999 from the default, or forward a different host port, but you'll also need to change the corresponding port in the `ansible-dryad/group_vars/all` file. Unless you know what you're doing with Dryad's internal and external ports, it's best to leave this at 9999.

### Ansible Vars

Software versions, passwords, file paths, __Git Repo addresses__, Administrator email addresses, and more are configured in `ansible-dryad/group_vars/all`.  This is a YAML file that specifies configuration for your VM that you may change.

__Do not proceed without generating a new db password__

## Building the VM

With Virtualbox, vagrant, and ansible installed, building the virtual machine is done with

    vagrant up

This command takes a while - it's downloading a base virtual machine, installing software packages, and loading Dryad.

Sometimes provisioning fails with `fatal: [x.x.x.x] => SSH encountered an unknown error during the connection.`.  In this case simply retry with `vagrant provision`

## Accessing the Virtual Machine

After the machine has been created/provisioned successfully, you can log in with

    vagrant ssh
    
Within the virtual machine, the __vagrant__ user owns the Dryad code and installed directory.

To shut down the virtual machine, use

    vagrant halt

If you wish to destroy the virtual machine

    vagrant destroy


## Developing, Building, Deploying

By default, the Dryad repo is checked out to `/home/vagrant/dryad-repo`. This and other defaults can be changed before provisioning by editing the `ansible-dryad/group_vars/all` file.

When you log in with ssh, the VM will show some information about file locations and next steps.  In order to get Dryad up and running, these scripts need to be run in order.

```
1. Build dryad          /home/vagrant/bin/build_dryad.sh
2. Deploy dryad         /home/vagrant/bin/deploy_dryad.sh
3. Install database     /home/vagrant/bin/install_dryad_database.sh
4. Start tomcat         /home/vagrant/dryad-tomcat/bin/startup.sh
5. Rebuild SOLR indexes /home/vagrant/bin/build_indexes.sh
```

After the first build/install process, you'll only need to run build, deploy, and startup.

## Debugging

If you'd like to use an external tool that supports JPDA debugging (e.g. NetBeans, Eclipse), the default JPDA port (8000) is already configured for forwarding. To start tomcat with debugging enabled, use the `/home/vagrant/dryad-tomcat/bin/startup-debug.sh` script

## Communication with the VM

In addition to port forwarding, the contents of this directory (The one containing the Vagrantfile) are synchronized from your host computer to the virtual machine's `/vagrant` directory. Additional synchronized directories can be added to the Vagrantfile. For example, the `dryad-bootstrap.sql` file that installs necessary content into the database is stored here, and used by the VM during installation.

## Hacking on this repo

If you change the Vagrantfile, issue the `vagrant reload` command. Vagrant will re-read the configuration and restart your VM. It will not rebuild it from scratch. This is useful for changing ports or adding shared directories

If you change the ansible playbooks or variables, issue the `vagrant provision` command. Vagrant will run ansible and pick up your changes.

In either case, it's best to make sure the change works from start to finish. You can do this by cloning the repo and issuing `vagrant up` again. 

