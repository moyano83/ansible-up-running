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

## Chapter 3: Inventory: Describing Your Servers<a name="Chapter3"></a>

### The Inventory File

The collection of hosts that Ansible knows about is called the inventory. The default way to describe your hosts in Ansible is to list them in text
files, called inventory files. Ansible automatically adds one host to the inventory by default: localhost. You have to have at least one other host in
your inventory file or ansible will fail but if you don't have other host, you can configure your localhost like `localhost ansible_connection=local`.

### Behavioral Inventory Parameters

Ansible calls variables related to connection properties behavioral inventory parameters, and there are several of them you can use when you need to
override the Ansible defaults for a host:

| Name | Default | Description |
|:---  | :---    | :---        |
| ansible_host | Name of host | Hostname or IP address to SSH to |
| ansible_port | 22 | Port to SSH to |
| ansible_user | Root | User to SSH as | 
| ansible_password | (None) | Password to use for SSH authentication | 
| ansible_connection | smart | How Ansible will connect to host (see the following section) | 
| ansible_private_key_file | (None) | SSH private key to use for SSH authentication | 
| ansible_shell_type | sh | Shell to use for commands (see the following section) | 
| ansible_python_interpreter | /usr/bin/ python | Python interpreter on host (see the following section) |
| ansible_*_interpreter | (None) | Like ansible_python_interpreter for other languages |

The ones that requires further explanation are:

#### ansible_connection

Ansible supports multiple transports, which are mechanisms that Ansible uses to connect to the host. The default transport, smart, will check whether
the locally installed SSH client supports a feature called ControlPersist. If the SSH client supports ControlPersist, Ansible will use the local SSH
client. If the SSH client doesn't support ControlPersist, the smart transport will fall back to using a Python-based SSH client library called
Paramiko.

#### ansible_shell_type

Ansible assumes that the remote shell is /bin/sh, and will generate the appropriate command-line parameters that work with this shell. Ansible also
accepts csh, fish, and (on Windows) powershell as valid values for this parameter.

#### ansible_python_interpreter

The modules that ship with Ansible are implemented in Python 2, Ansible needs to know the location of the Python interpreter on the remote machine.
You might need to change this if your remote host does not have a Python 2 interpreter at /usr/bin/python.

#### ansible_*_interpreter

If you are using a custom module that is not written in Python, you can use this parameter to specify the location of the interpreter.

### Changing Behavioral Parameter Defaults

You can override some of the behavioral parameter default values in the defaults section of the _ansible.cfg_ file:

| Behavioral inventory parameter | ansible.cfg option | 
|:---  | :---    | 
| ansible_port | remote_port |
| ansible_user | remote_user |
| ansible_private_key_file | private_key_file |
| ansible_shell_type | executable (see the following paragraph) |

### Groups and Groups and Groups

When performing configuration tasks, we typically want to perform actions on groups of hosts, rather than on an individual host. Ansible automatically
defines a group called _all_ (or *), which includes all of the hosts in the inventory. We can define our own groups in the inventory file. Ansible
uses the .ini file format for inventory files. In the .ini format, configuration values are grouped together into sections.

#### Aliases and Ports

Ansible supports using <hostname>:<port> syntax when specifying hosts, but you can associate only a single host with an IP.

#### Numbered Hosts (Pets versus Cattle)

Ansible can work with numeric patterns to define servers, like in:

```ini
[web]
web[1:20].example.com # or web[01:20].example.com if you prefer leading with 0
```

#### Hosts and Group Variables: Inside the Inventory

We can define arbitrary variable names and associated values on hosts, for example, we could define a variable named `color` and set it to a value for
each server like in `newhampshire.example.com color=red`. This variable can then be used in a playbook, just like any other variable. You can also
define group variables like in:

```ini
[all:vars]
ntp_server = ntp.ubuntu.com

[production:vars]
db_primary_host = rhodeisland.example.com
```

### Host and Group Variables: In Their Own Files

As your inventory gets larger, it gets more difficult to manage variables as seen in the previous section. Ansible offers a more scalable approach to
keep track of host and group variables: you can create a separate variable file for each host and each group. Ansible looks for host variable files in
a directory called `host_vars` and group variable files in a directory called `group_vars`. Ansible expects these directories to be either in the
directory that contains your playbooks or in the directory adjacent to your inventory file. In the example above we can put these variables in
_./group\_vars/production_. Here, _production_ can be a YAML file with the variables, or a directory containing multiple files for the variables.

### Dynamic Inventory

You might have a system external to Ansible that keeps track of your hosts (Amazon EC2, tracks information about your hosts for you, and you can
retrieve this information through EC2’s web interface). If your inventory file is an executable, Ansible will assume it is a dynamic inventory script
and will execute the file instead of reading it.

#### The Interface for a Dynamic Inventory Script

An Ansible dynamic inventory script must support two command-line flags:

    * --host=<hostname> for showing host details
    * --list for listing groups

The output of any of the two commands must be a single JSON object: the names are variable names, and the values are the variable values. As an
optimization, the `--list` command can contain the values of the host variables for all of the hosts, which saves Ansible the trouble of making a
separate `--host` invocation to retrieve the variables for the individual hosts. To take advantage of this optimization, the `--list` command should
return a key named _\_meta_ that contains the variables for each host, in this form:

```text
"_meta" :
    { "hostvars" :
        "vagrant1" : { "ansible_host": "127.0.0.1", "ansible_port": 2222, "ansible_user": "vagrant"}, ... }
```

#### Breaking the Inventory into Multiple Files

If you want to have both a regular inventory file and a dynamic inventory script (or, really, any combination of static and dynamic inventory files),
just put them all in the same directory and configure Ansible to use that directory as the inventory. You can do this either via the inventory
parameter in ansible.cfg or by using the -i flag on the command line.

#### Adding Entries at Runtime with add_host and group_by

Ansible will let you add hosts and groups to the inventory during the execution of a playbook.

##### add_host

The _add_host_ module adds a host to the inventory. This module is useful if you’re using Ansible to provision new virtual machine instances inside an
infrastructure-as-a-service cloud (useful for scenarios where you start up new virtual machine instances and configure them in the same playbook).

##### group_by

Ansible also allows you to create new groups during execution of a playbook, using the _group_by_ module. This lets you create a group based on the
value of a variable that has been set on each host, which Ansible refers to as a fact. For example, we use _group_by_ to create separate groups for
our Ubuntu hosts, and then we use the apt module to install packages:

```yaml
- name: group hosts by distribution
  hosts: myhosts
  gather_facts: True
  tasks:
    - name: create groups based on distro
      group_by: key={{ ansible_distribution }}
- name: do something to Ubuntu hosts
  hosts: Ubuntu
  tasks:
    - name: install htop
      apt: name=htop
```

## Chapter 4: Variables and Facts<a name="Chapter4"></a>