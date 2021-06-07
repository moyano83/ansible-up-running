# Ansible - Up & Running

## Table of contents:

1. [Chapter 1: Introduction](#Chapter1)
2. [Chapter 2: Playbooks: A Beginning](#Chapter2)
3. [Chapter 3: Inventory: Describing Your Servers](#Chapter3)
4. [Chapter 4: Variables and Facts](#Chapter4)
5. [Chapter 5: Introducing Mezzanine: Our Test Application](#Chapter5)
6. [Chapter 6: Deploying Mezzanine with Ansible](#Chapter6)
7. [Chapter 7: Roles: Scaling Up Your Playbooks](#Chapter7)
8. [Chapter 8: Complex Playbooks](#Chapter8)
9. [Chapter 9: Customizing Hosts, Runs, and Handlers](#Chapter9)
10. [Chapter 10: Callback Plugins](#Chapter10)
11. [Chapter 11: Making Ansible Go Even Faster](#Chapter11)
12. [Chapter 12: Custom Modules](#Chapter12)
13. [Chapter 13: Vagrant](#Chapter13)
14. [Chapter 14: Amazon EC2](#Chapter14)
15. [Chapter 15: Docker](#Chapter15)
16. [Chapter 16: Debugging Ansible Playbooks](#Chapter16)
17. [Chapter 17: Managing Windows Hosts](#Chapter17)
18. [Chapter 18: Ansible for Network Devices](#Chapter18)
19. [Chapter 19: Ansible Tower: Ansible for the Enterprise](#Chapter19)

## Chapter 1: Introduction<a name="Chapter1"></a>

### Ansible: What Is It Good For?

Ansible is often described as a configuration management tool, and is typically mentioned in the same breath as Chef, Puppet, and Salt. Ansible
exposes a domain-specific language (DSL) that you use to describe the state of your servers.

### How Ansible Works

In Ansible, a script is called a playbook. A playbook describes which hosts (what Ansible calls remote servers) to configure, and an ordered list of
tasks to perform on those hosts. It’s important to note the following:

    * Ansible runs each task in parallel across all hosts
    * Ansible waits until all hosts have completed a task before moving to the next task
    * Ansible runs the tasks in the order that you specify them

### What’s So Great About Ansible?

    * Ansible’s playbook syntax is built on top of YAML, which is a human friendly data format language
    * There’s no need to preinstall an agent or any other software on the host
    * Ansible is push based by default, so you control when the changes happen to the servers, althought ansible has official support for pull 
      mode, using a tool it ships with called ansible-pull
    * Ansible Scales Down, it is easy to configure a single node
    * Ansible modules are declarative; you use them to describe the state you want the server to be in. Ansible comes with a large collection of 
      modules it ships with, and modules are idempotent
    * Ansible does not provide a layer of abstraction so that you can use the same configuration management scripts to manage servers running 
      different operating systems, ansible playbooks aren’t really intended to be reused across different contexts

### Installing Ansible

All of the major Linux distributions package Ansible these days, on macOS you can use `sudo pip install ansible`

### Setting Up a Server for Testing

#### Using Vagrant to Set Up a Test Server

Vagrant has built-in support for provisioning virtual machines with Ansible, Vagrant needs the VirtualBox virtualizer to be installed on your machine.

#### Telling Ansible About Your Test Server

Ansible can manage only the servers it explicitly knows about. You provide Ansible with information about servers by specifying them in an inventory
file and add the information about your servers there. You can test the connectivity to a server using the ping module 
`ansible testserver -i hosts -m ping`

### Simplifying with the ansible.cfg File

Ansible looks for an ansible.cfg file in the following places, in this order:

    1. File specified by the ANSIBLE_CONFIG environment variable 
    2. ./ansible.cfg (ansible.cfg in the current directory)
    3. ~/.ansible.cfg (.ansible.cfg in your home directory)
    4. /etc/ansible/ansible.cfg

In this file, you can define common configuration options such as:

```text
[defaults]
inventory = hosts
remote_user = vagrant
private_key_file = .vagrant/machines/default/virtualbox/private_key
host_key_checking = False
```

You can execute arbitrary commands with the command module (default module for ansible). When invoking this module, you also need to pass an 
argument to the module with the _-a_ flag, which is the command to run `ansible testserver -m command -a uptime` or 
`ansible testserver -a "tail /var/log/dmesg"`. Use the _-b_ flag to execute the command as root.