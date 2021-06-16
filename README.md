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

You can execute arbitrary commands with the command module (default module for ansible). When invoking this module, you also need to pass an argument
to the module with the _-a_ flag, which is the command to run `ansible testserver -m command -a uptime` or
`ansible testserver -a "tail /var/log/dmesg"`. Use the _-b_ flag to execute the command as root.

## Chapter 2: Playbooks: A Beginning<a name="Chapter2"></a>

A playbook is the term that Ansible uses for a configuration management script.

### Playbooks Are YAML

Ansible playbooks are written in YAML syntax. YAML is a file format similar in intent to JSON, but generally easier for humans to read and write. YAML
files are supposed to start with three dashes (---) to indicate the beginning of the document but it is not required. Comments start with '#' and in
general, YAML strings don’t have to be quoted, although you can quote them if you prefer. YAML lists are like arrays in JSON and Ruby, or lists in
Python, they are delimited with hyphens. YAML dictionaries are like objects in JSON, dictionaries in Python, or hashes in Ruby.

When writing playbooks, you might want to break this up across multiple lines in your file, but you want Ansible to treat the string as if it were a
single line. You can do this with YAML by using line folding with the greater than `>` character. The YAML parser will replace line breaks with
spaces.

### Anatomy of a Playbook

A playbook is a list of dictionaries. Specifically, a playbook is a list of plays. Every play must contain the following:

    * A set of hosts to configure
    * A list of tasks to be executed on those hosts

Along with this, a play commonly has a `name` (describes what the plays does), `become` (If true, Ansible will run every task by becoming root)
and `vars` (list of variables to use).

A play is formed by tasks, and you can use the `--start-at-task <task name>` flag to tell ansible-playbook to start a playbook in the middle of a
play. Every task must contain a key with the name of a module and a value with the arguments to that module. Modules are scripts that come packaged
with Ansible and perform some kind of action on a host (like `apt`, `copy`, `file` or `service`). Ansible ships with the `ansible-doc` command-line
tool, which shows documentation about modules, it can be run like `ansible-doc <module>`.

#### Did Anything Change? Tracking Host State

Ansible modules will first check to see whether the state of the host needs to be changed before taking any action. If the state of the host matches
the arguments of the module, Ansible takes no action on the host and responds with a state of ok.

#### Variables

Any valid YAML can be used as the value of a variable, including lists and dictionaries. You reference variables by using the {{ braces }} notation.
Ansible replaces these braces with the value of the variable. If you reference a variable right after specifying the module, the YAML parser will
misinterpret the variable reference as the beginning of an inline dictionary and you need to quote it.

### Templating

Ansible uses the Jinja2 template engine to implement templating. We use the .j2 extension to indicate that the file is a Jinja2 template. Ansible also
uses the Jinja2 template engine to evaluate variables in playbooks.

### Handlers

Handlers are one of the conditional forms that Ansible supports. A handler is similar to a task, but it runs only if it has been notified by a task. A
task will fire the notification if Ansible recognizes that the task has changed the state of the system. Handlers usually run after all of the tasks
are run at the end of the play. They run only once, even if they are notified multiple times. If a play contains multiple handlers, the handlers
always run in the order that they are defined in the handlers section, not the notification order.