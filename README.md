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
tasks to perform on those hosts. It's important to note the following:

    * Ansible runs each task in parallel across all hosts
    * Ansible waits until all hosts have completed a task before moving to the next task
    * Ansible runs the tasks in the order that you specify them

### What's So Great About Ansible?

    * Ansible's playbook syntax is built on top of YAML, which is a human friendly data format language
    * There's no need to preinstall an agent or any other software on the host
    * Ansible is push based by default, so you control when the changes happen to the servers, althought ansible has official support for pull 
      mode, using a tool it ships with called ansible-pull
    * Ansible Scales Down, it is easy to configure a single node
    * Ansible modules are declarative; you use them to describe the state you want the server to be in. Ansible comes with a large collection of 
      modules it ships with, and modules are idempotent
    * Ansible does not provide a layer of abstraction so that you can't use the same configuration management scripts to manage servers running 
      different operating systems, ansible playbooks aren't really intended to be reused across different contexts

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
general, YAML strings don't have to be quoted, although you can quote them if you prefer. YAML lists are like arrays in JSON and Ruby, or lists in
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
retrieve this information through EC2's web interface). If your inventory file is an executable, Ansible will assume it is a dynamic inventory script
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

The _add_host_ module adds a host to the inventory. This module is useful if you're using Ansible to provision new virtual machine instances inside an
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

### Defining Variables in Playbooks

The simplest way to define variables is to put a vars section in your playbook with the names and values of variables:

```yaml
vars:
  key_file: /etc/nginx/ssl/nginx.key
  cert_file: /etc/nginx/ssl/nginx.crt
  conf_file: /etc/nginx/sites-available/default
  server_name: localhost
```

Ansible also allows you to put variables into one or more files, using a section called _vars\_files_:

```yaml
vars_files:
  - file_in_the_same_folder.yml
```

For debugging, it's often handy to be able to view the output of a variable. You can use the following: `- debug: var=myvarname`.

### Registering Variables

Often, you'll find that you need to set the value of a variable based on the result of a task. To do so, we create a registered variable using the
register clause when invoking a module:

```yaml
- name: capture output of whoami command
  command: whoami
  register: login
```

The value of a variable set using the register clause is always a dictionary, but the specific keys of the dictionary are different depending on the
module that was invoked. If a variable contains a dictionary, you can access the keys of the dictionary by using either a dot (.) or a subscript ([]).
The simplest way to find out what a module returns is to register a variable and then output that variable with the debug module. If the task fails,
Ansible will stop executing tasks for the failed host. We can use the _ignore\_errors_ clause to change this behaviour.

### Facts

When Ansible gathers facts, it connects to the host and queries it for all kinds of details about the host: CPU architecture, operating system, IP
addresses, memory info, disk info, and more. This information is stored in variables that are called facts, and they behave just like any other
variable. You can get those like in the example below:

```yaml
 - name: print out operating system
     hosts: all
     gather_facts: True
     tasks:
       - debug: var=ansible_distribution
```

Ansible implements fact collecting through the use of a special module called the setup module. You don't need to call this module in your playbooks
because Ansible does that automatically when it gathers facts. Because Ansible collects many facts, the setup module supports a filter parameter that
lets you filter by fact name by specifying a glob. For example: `ansible web -m setup -a 'filter=ansible_eth*'`. The output is a dictionary whose key
is _ansible\_facts_.

#### Local Facts

Ansible provides an additional mechanism for associating facts with a host. You can place one or more files on the remote host machine in the
_/etc/ansible/facts.d_ directory. Ansible will recognize the file if it's any of the following:

    * In .ini format 
    * In JSON format
    * An executable that takes no arguments and outputs JSON on standard out 

These facts are available as keys of a special variable named _ansible\_local_. If we name the file _example.fact_, then the dictionary will contain a
key _ansible\_local_ with another key _example_ containing all the variables set in the _example.fact_ file.

#### Using set_fact to Define a New Variable

Ansible also allows you to set a fact in a task by using the _set\_fact_ module. Below Example demonstrates how to use set_fact so that a variable can
be referred to as snap instead of _snap\_result.stdout_:

```yaml
- name: get snapshot id
  shell: >
    aws ec2 describe-snapshots --filters
    Name=tag:Name,Values=my-snapshot
    | jq --raw-output ".Snapshots[].SnapshotId"
  register: snap_result
- set_fact: snap={{ snap_result.stdout }}
```

### Built-in Variables

Ansible defines several variables that are always available in a playbook:

| Parameter | Description | 
|:--- | :--- | 
| hostvars | A dict whose keys are Ansible hostnames and values are dicts that map variable names to values | 
| inventory_hostname | Fully qualified domain name of the current host as known by Ansible (e.g., myhost.example.com) |
| inventory_hostname_short | Name of the current host as known by Ansible, without the domain name (e.g., myhost) | 
| group_names | A list of all groups that the current host is a member of |
| groups | A dict whose keys are Ansible group names and values are a list of hostnames that are members of the group. Includes all and ungrouped groups:{"all": [...], "web": [...], "ungrouped": [...]} |
| ansible_check_mode | A boolean that is true when running in check mode | 
| ansible_play_batch | A list of the inventory hostnames that are active in the current batch |
| ansible_play_hosts | A list of all of the inventory hostnames that are active in the current play | 
| ansible_version | A dict with Ansible version info: {"full": 2.3.1.0", "major": 2, "minor": 3, "revision": 1, "string": "2.3.1.0"} |

#### hostvars

In Ansible, variables are scoped per host. It only makes sense to talk about the value of a variable relative to a given host. Sometimes, a task
that's running on one host needs the value of a variable defined on another host. Imagine you need to retrieve the IP of a particular server, if our
server is db.example.com, then we could put the following in a configuration template: `{{ hostvars['db.example.com'].ansible_eth1.ipv4.address }}`.

#### inventory_hostname

The inventory_hostname is the hostname of the current host, as known by Ansible. If your inventory contains a line like
`server1 ansible_host=192.168.4.10` then _inventory\_hostname_ would be _server1_.

#### Groups

The groups variable can be useful when you need to access variables for a group of hosts. You can use it like in the example below:

```jinja2
{% for host in groups.web %}
  server {{ hostvars[host].inventory_hostname }} {{ hostvars[host].ansible_default_ipv4.address }}:80
{% endfor %}
```

#### Setting Variables on the Command Line

Variables set by passing `-e var=value` to ansible-playbook have the highest precedence, which means you can use this to override variables that are
already defined. Ansible also allows you to pass a file containing the variables instead of passing them directly on the command line by passing
_@filename.yml_ as the argument to -e.

#### Precedence

You can define the same variable multiple times for a host, using different values (avoid this when you can). When the same variable is defined in
multiple ways, the precedence rules determine which value wins. The basic rules of precedence are as follows (Highest to lowest):

    1. ansible-playbook -e var=value
    2. Task variables
    3. Block variables
    4. Role and include variables
    5. set_fact
    6. Registered variables
    7. vars_files
    8. vars_prompt
    9. Play variables
    10. Host facts
    11. host_vars set on a playbook
    12. group_vars set on a playbook
    13. host_vars set in the inventory 
    14. group_vars set in the inventory
    15. Inventory variables
    16. In defaults/main.yml of a role

## Chapter 5: Introducing Mezzanine: Our Test Application<a name="Chapter5"></a>

In contrast with the deployment to a development environment, deploying to a production environment involves the use of several independent pieces of
software such as a web server, database with specific user's application permissions, a process supervisor and much more. This is to be developed in
the next chapter.

## Chapter 6: Deploying Mezzanine with Ansible<a name="Chapter6"></a>

The ansible-playbook command-line tool supports a flag called `--list-tasks`. This flag prints out the names of all the tasks in a playbook:
`ansible-playbook --list-task <filename>.yaml`.

### Using Iteration (with_items) to Install Multiple Packages

To install several system packages at the same time, we can use the `with_items` property like this:

```yaml
- name: install apt packages
  apt: pkg={{ item }} update_cache=yes cache_valid_time=3600
  become: True
  with_items:
    - git
    - libjpeg-dev
    - libpq-dev
```

The `{{ item }}` is a placeholder variable that will be populated by each of the elements in the list of the `with_items` clause. The apt module
contains an optimization making it more efficient to install multiple packages by using the with_items clause. Ansible will pass the entire list of
packages to the apt module, and the module will invoke the apt program only once, passing it the entire list of packages to be installed. By
adding `become: True` to the tasks we effectively run them as root.

### Updating the Apt Cache

Ubuntu maintains a cache with the names of all of the apt packages that are available in the Ubuntu package archive. In some cases, when the Ubuntu
project releases a new version of a package, it removes the old version from the package archive. If the local apt cache of an Ubuntu server hasn't
been updated, then it will attempt to install a package that doesn't exist in the package archive. Run `apt-get update` to update the cache.

### Complex Arguments in Tasks: A Brief Digression

A complex argument is an argument to a module that is a list or a dictionary. If we don't like long lines in our files, we could break up the argument
string across multiple lines by using YAML's line folding (>), or you can mix it up by passing some arguments as a string and others as a dictionary.

### Running Custom Python Scripts in the Context of the Application

You can set environment variables with an environment clause on a task, passing it a dictionary that contains the environment variable names and
values. You can add an environment clause to any task; it doesn't have to be a script. i.e.

```yaml
- name: set the site id
  script: scripts/setsite.py
  environment:
    PATH: "{{ venv_path }}/bin"
    PROJECT_DIR: "{{ proj_path }}"
    PROJECT_APP: "{{ proj_app }}"
    WEBSITE_DOMAIN: "{{ live_hostname }}"
```

## Chapter 7: Roles: Scaling Up Your Playbooks<a name="Chapter7"></a>

In Ansible, the role is the primary mechanism for breaking a playbook into multiple files.

### Basic Structure of a Role

An Ansible role has a name, such as database. Files associated with the database role go in the roles/database directory, which contains the following
files and directories (each individual file is optional, no need to add an empty file if no configuration is given):

    * roles/database/tasks/main.yml: Tasks
    * roles/database/files/: Holds files to be uploaded to hosts
    * roles/database/templates/: Holds Jinja2 template files
    * roles/database/handlers/main.yml: Handlers
    * roles/database/vars/main.yml: Variables that shouldn't be overridden
    * roles/database/defaults/main.yml: Default variables that can be overridden
    * roles/database/meta/main.yml: Dependency information about a role

Ansible looks for roles in the _roles_ directory alongside your playbooks. It also looks for systemwide roles in _/etc/ansible/roles_
(customizable in ansible.cfg or by providing ANSIBLE_ROLES_PATH).

### Using Roles in Your Playbooks

When we use roles, we have a _roles_ section in our playbook, we can pass in variables when invoking the roles:

```yaml
- role: my_role
  name: "{{ some_value }}"
  user: "{{ some_value }}"
```

### Pre-Tasks and Post-Tasks

Ansible allows you to define a list of tasks that execute before the roles with a _pre\_tasks_ section, and a list of tasks that execute after the
roles with a _post\_tasks_ section:

```yaml
- name: My Ansible script
  hosts: web
  vars_files:
    - secrets.yml
  pre_tasks:
    - name: some task to be executed before my role
      ...
  roles:
    - role: my_role
      ...
  post_tasks:
    -name: some task to be executed after my role
  ...
```

#### Defining variables in roles

There are two places available to define variables in roles, the _vars/main.yml_ file and the _defaults/main.yml_. If you think you might want to
change the value of a variable in a role, use a default variable. If you don't want it to change, use a regular variable. It is a good practice to
prepend your variable names with the role name, as ansible doesn't have the concept of namespace for variables (This means that variables that are
defined in other roles, or elsewhere in a playbook, will be accessible everywhere).

There's one important difference between tasks defined in a role and tasks defined in a regular playbook, and that's when using the copy or template
modules. When invoking copy in a task defined in a role, Ansible will first check the rolename/ files/ directory for the location of the file to copy.
Similarly, when invoking template in a task defined in a role, Ansible will first check the rolename/templates directory for the location of the
template to use.

### Creating Role Files and Directories with ansible-galaxy

Ansible has a command line tool named ansible-galaxy, which primary purpose is to download roles that have been shared by the Ansible community, but
it can also be used to generate scaffolding, an initial set of files and directories involved in a role:
`ansible-galaxy init -p <role directory> <role name>`

### Dependent Roles

Ansible supports a feature called dependent roles, when you define a role, you can specify that it depends on one or more other roles. Ansible will
ensure that roles that are specified as dependencies are executed first. We specify that the web role depends on the ntp role by creating a
_roles/web/meta/main.yml_ file and listing the dependant roles there

```yaml
dependencies:
  - { role1: some_role1, var_name1=var_value1 }
  - { role2: some_role2 }
```

### Ansible Galaxy

Ansible Galaxy is an open source repository of Ansible roles contributed by the Ansible community. The roles themselves are stored on GitHub. The
ansible-galaxy command-line tool allows you to download roles from Ansible Galaxy. You can install a role (i.e. _test_ developed by _dev1_) with
`ansible-galaxy install -p ./roles dev1.test`. The ansible-galaxy program will install roles to your systemwide location by default (configurable in
ansible.cfg). You can list installed roles with `ansible-galaxy list` and remove a role with `ansible-galaxy remove dev1.test`.

## Chapter 8: Complex Playbooks<a name="Chapter8"></a>

### Dealing with Badly Behaved Commands: changed_when and failed_when

If we don't have a module that could invoke idempotent commands, we can use _changed\_when_ and _failed\_when_ clauses to change how Ansible
identifies that a task has changed state or failed. In the example below, if the database is created it print something like "Creating tables", but
when we try to run the second time it prints something like "CommandError: Database already created".

```yaml
- name: initialize the database
  django_manage:
    command: createdb --noinput --nodata
    app_path: "{{ proj_path }}"
    virtualenv: "{{ venv_path }}"
  register: result
  changed_when: '"Creating tables" in result.out|default("")'
```

### Filters

Filters are a feature of the Jinja2 templating engine, you can use filters inside `{{ braces }}` in your playbooks, as well as inside your template
files. Using filters resembles using Unix pipes, whereby a variable is piped through a filter.

#### The Default Filter

The default filter is used to define default values in variables, in case they are not defined: `{{ database_host | default('localhost') }}`.

#### Filters for Registered Variables

Let's say we want to run a task and print out its output, even if the task fails (and fail after printing the variable in this case):

```yaml
- name: Run myprog
  command: bash ./myprog.sh
  register: result
  ignore_errors: True

- debug: var=result

- debug: msg="Stop running the playbook if myprog failed"
  failed_when: result|failed
```

Task return value filters:

| Name | Description |
| :---  | :--- |
| failed | True if a registered value is a task that failed |
| changed | True if a registered value is a task that changed |
| success | True if a registered value is a task that succeeded |
| skipped | True if a registered value is a task that was skipped |

#### Filters That Apply to File Paths

File path filters:

| Name | Description |
| :---  | :--- |
| basename | Base name of file path |
| dirname | Directory of file path |
| expanduser | File path with~replaced by home directory |
| realpath | Canonical path of file path, resolves symbolic links |

In an example, the playbook below avoids repetition of 'index.html' by using the basename filter.

```yaml
vars:
  homepage: /usr/share/nginx/html/index.html
tasks:
  - name: copy home page
    copy: src=files/{{ homepage | basename }} dest={{ homepage }}
```

#### Writing Your Own Filter

Ansible will look for custom filters in the filter_plugins directory, relative to the directory containing your playbooks. You can place a python
script in this directory with the following signature to add your own scripts:

```python
# MyFilter.py
def my_function(parameters_to_my_function):
    return "Something with parameters_to_my_function"


class FilterModule(object):
    def filters(self):
        return {'my_function_name': my_function}
```

#### Lookups

Sometimes a piece of configuration data you need lives somewhere else, like a text file or a csv. Ansible has a feature called lookups that allows you
to read in configuration data from various sources and then use that data in your playbooks and template.

| Name | Description |
| :---  | :--- |
| file | Contents of a file |
| password | Randomly generate a password |
| pipe | Output of locally executed command |
| env | Environment variable |
| template | Jinja2 template after evaluation |
| csvfile | Entry in a .csv file |
| dnstxt | DNS TXT record |
| redis_kv | Redis key lookup |
| etcd | etcd key lookup |

You invoke lookups by calling the lookup function with two arguments. The first is a string with the name of the lookup, and the second is a string
that contains one or more arguments to pass to the lookup. You can invoke lookups in your playbooks between `{{ braces }}`, or you can put them in
templates.

##### File

Read the contents of a file and pass that as a parameter to a module.

```yaml
- name: Add my public key as an EC2 key
  ec2_key: name=mykey key_material="{{ lookup('file', '/Users/jmoyano/.ssh/id_rsa.pub') }}"
```

##### pipe

The pipe lookup invokes an external program on the control machine and evaluates to the program's output on standard out.

````yaml
- name: get SHA of most recent commit
  debug: msg="{{ lookup('pipe', 'git rev-parse HEAD') }}"
````

##### env

The env lookup retrieves the value of an environment variable set on the control machine:

```yaml
 - name: get the current shell
   debug: msg="{{ lookup('env', 'SHELL') }}"
```

##### password

The password lookup evaluates to a random password, and it will also write the password to a file specified in the argument.

```yaml
- name: create deploy postgres user
  postgresql_user:
    name: deploy
    password: "{{ lookup('password', 'deploy-password.txt') }}"
```

##### template

The template lookup lets you specify a Jinja2 template file, and then returns the result of evaluating the template.

```yaml
- name: output message from template
  debug: msg="{{ lookup('template', 'message.j2') }}"
```

##### csvfile

The csvfile lookup reads an entry from a csv file. An example of utilization is `lookup('csvfile', 'sue file=users.csv delimiter=, col=1')`. The
second parameter to lookup is a list of parameters, in this case you don't specify a name for the first argument to a lookup plugin, but you do
specify names for the additional arguments. The first argument is an entry that must appear exactly once in column 0 (the first column, 0-indexed)
of the table. The other arguments specify the name of the csv file, the delimiter, and which column should be returned.

##### dnstxt

A TXT record is just an arbitrary string that you can attach to a hostname in the DNS protocol. Once you've associated a TXT record with a hostname,
anybody can retrieve the text by using a DNS client, like in: `dig +short some_domain.com TXT`. The dnstxt lookup queries the DNS server for the TXT
record associated with the host. We can do the same in a playbook:

```yaml
- name: look up TXT record
  debug: msg="{{ lookup('dnstxt', 'ansiblebook.com') }}"
```

If multiple TXT records are associated with a host, the module will concatenate them together, and it might do this in a different order each time it
is called.

##### redis_kv

Redis is a popular key-value store, commonly used as a cache, as well as a data store for job queue services. You can use the redis_kv lookup to
retrieve the value of a key.

```yaml
- name: look up value in Redis
  debug: msg="{{ lookup('redis_kv', 'redis://localhost:6379,key_to_look_for') }}"
```

The module will default to _redis://localhost:6379_ if the URL isn't specified, so we could invoke the module like:
`lookup('redis_kv', ',key_to_look_for')`.

##### etcd

Etcd is a distributed key-value store, commonly used for keeping configuration data and for implementing service discovery. If we define a task in our
playbook that invokes the etcd plugin:

```yaml
- name: look up value in etcd
  debug: msg="{{ lookup('etcd', 'key_to_look_for') }}"
```

By default, the etcd lookup looks for the etcd server at _http://127.0.0.1:4001_, but you can change this by setting the _ANSIBLE\_ETCD\_URL_
environment variable before invoking ansible-playbook.

### More Complicated Loops

Summary of the iteration constructs available:

| Name | Input | Looping Strategy |
| :--- | :---- | :--------------- |
| with_items | List | Loop over list elements |
| with_lines | Command to execute | Loop over lines in command output |
| with_fileglob | Glob | Loop over filenames | 
| with_first_found | List of paths | First file in input that exists |
| with_dict | Dictionary | Loop over dictionary elements |
| with_flattened | List of lists | Loop over flattened list | 
| with_indexed_items | List | Single iteration | 
| with_nested | List | Nested loop | 
| with_random_choice | List | Single iteration | 
| with_sequence | Sequence of integers | Loop over sequence | 
| with_subelements | List of dictionaries | Nested loop | 
| with_together | List of lists | Loop over zipped list | 
| with_inventory_hostnames | Host pattern | Loop over matching hosts |

#### with_lines

The with_lines looping construct lets you run an arbitrary command on your control machine and iterate over the output, one line at a time.

```yaml
- name: Send out a slack message
  slack:
    domain: example.slack.com
    token: "{{ slack_token }}"
    msg: "{{ item }} was in the list"
  with_lines:
    - cat files/turing.txt
```

#### with_fileglob

The with_fileglob construct is useful for iterating over a set of files on the control machine.

```yaml
- name: add public keys to account
  authorized_key: user=deploy key="{{ lookup('file', item) }}"
  with_fileglob:
    - /var/keys/*.pub
```

#### with_dict

The with_dict construct lets you iterate over a dictionary instead of a list. The item loop variable is a dictionary with two keys: _key_ and _value_.

```yaml
- name: iterate over ansible_eth0
    debug: msg={{ item.key }}={{ item.value }}
    with_dict: "{{ ansible_eth0.ipv4 }}"
```

#### Looping Constructs as Lookup Plugins

Ansible implements looping constructs as lookup plugins. You just slap a with at the beginning of a lookup plugin to use it in its loop form.

```yaml
- name: Add my public key as an EC2 key using the file lookup as a loop
  ec2_key: name=mykey key_material="{{ item }}"
  with_file: /Users/lorin/.ssh/id_rsa.pub
```

### Loop Controls

Ansible provides users with more control over loop handling.

#### Setting the Variable Name

The loop_var control allows us to give the iteration variable a different name than the default name, item. In the example below, we would like to
loop over multiple tasks at once. One way to achieve that is to use include with `with_items`. However, the _vhosts.yml_ file that is going to be
included may also contain `with_items` in some tasks. This would produce a conflict, as the default `loop_var` item is used for both loops at the same
time. To prevent a naming collision, we specify a different name for `loop_var` in the outer loop (inside the _vhosts.yml_ file we can use the
`item` var inside the loops as well as `vhost.domain`).

```yaml
- name: run a set of tasks in one loop
  include: vhosts.yml
  with_items:
    - { domain: www1.example.com }
    - { domain: www2.example.com }
  loop_control:
    loop_var: vhost
```

#### Labeling the Output

The label control provides some control over how the loop output will be shown to the user during execution.

```yaml
- name: create nginx vhost configs
  template:
    src: "{{ item.domain }}.conf.j2"
    dest: "/etc/nginx/conf.d/{{ item.domain }}.conf"
  with_items:
    - { domain: www1.example.com, ssl_enabled: yes }
    - { domain: www2.example.com, aliases: [ edge2.www.example.com, eu.www.example.com ] }
  loop_control:
    label: "for domain {{ item.domain }}"
```

This results in much more readable output:

```text
TASK [create nginx vhost configs] **********************************************
ok: [localhost] => (item=for domain www1.example.com)
ok: [localhost] => (item=for domain www2.example.com)
```

### Includes

The include feature allows you to include tasks or even whole playbooks, depending on where you define an include. It is often used in roles to
separate or even group tasks and task arguments to each task in the included file. If we have a file named _nginx_include.yml_ with contents:

```yaml
- name: install nginx
  package:
    name: nginx

- name: ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

We can create a task like the one below to which would add the `tag`, `become` and `when` condition to both of them.

```yaml
- include: nginx_include.yml
  tags: nginx
  become: yes
  when: ansible_os_family == 'RedHat'
```

#### Dynamic Includes

A common pattern in roles is to define tasks specific to a particular operating system into separate task files. Ansible allows us to dynamically
include a file by using variable substitution (but `ansible-playbook --list-tasks` might not list the tasks from a dynamic `include` if it does not
have enough information to populate the variables that determine which file will be included):

```yaml
- include: "{{ ansible_os_family }}.yml"
  static: no
```

#### Role Includes

A special include is the include_role clause. In contrast with the role clause, which will use all parts of the role, the include_role not only allows
us to selectively choose what parts of a role will be included and used, but also where in the play.

```yaml
- name: install php
  include_role:
    name: php
    tasks_from: install
```

### Blocks

Like the _include_ clause, the _block_ clause provides a mechanism for grouping tasks. The _block_ clause allows you to set conditions or arguments
for all tasks within a block at once:

```yaml
- block:
    - name: install nginx
      package:
        name: nginx
    - name: ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
  become: yes
  when: "ansible_os_family == 'RedHat'"
```

### Error Handling with Blocks

Ansible's default error-handling behavior is to take a host out of the play if a task fails and continue as long as there are hosts remaining that
haven't encountered errors. In combination with the _serial_ and _max\_fail\_percentage_ clause, Ansible gives you some control over when a play has
to be declared as failed. The way error handling is implemented reminds the _try-catch-finally_ paradigm, and it works much the same way.

```yaml
- block:
    - debug: msg="You will see a failed tasks right after this"
    - command: /bin/false
    - debug: "You won't see this message"
      rescue:
    - debug: "You only see this message in case of an failure in the block"
      always:
    - debug: "This will be always executed"
```

### Encrypting Sensitive Data with Vault

In case our playbooks needs access to sensitive information, Ansible allows us to encrypt files using a command line tool named _ansible-vault_. This
tool allows you to create and edit an encrypted file that ansible-playbook will recognize and decrypt automatically, given the password. We can
encrypt an existing file with `ansible-vault encrypt my_secret_file.yml` or even create it from scratch with `ansible-vault create my_secret_file.yml`
(you will be prompted for a password, and then ansible-vault will launch a text editor so that you can populate the file). To use this file, we need
to tell ansible-playbook to prompt us for the password of the encrypted file, or it will simply error out.
Use `ansible-playbook mezzanine.yml --ask-vault-pass` or put the password in a file and use
`ansible-playbook mezzanine --vault-password-file ./my_password_file.txt`, if the argument to `--vault-password-file` has the executable bit set,
Ansible will execute it and use the contents of standard out as the vault password. Other ansible-vault commands are:

| Command | Description |
| :------ | :---------- |
| encrypt | Encrypt the plain-text file.yml file |
| decrypt | Decrypt the encrypted file.yml file |
| view    | Print the contents of the encrypted file.yml file |
| create  | Create a new encrypted file.yml file |
| edit    | Edit an encrypted file.yml file |
| rekey   | Change the password on an encrypted file.yml file |

## Chapter 9: Customizing Hosts, Runs, and Handlers<a name="Chapter9"></a>

### Patterns for Specifying Hosts

Instead of specifying a single host or group, you can specify a pattern. From all hosts `hosts: all` to a union of groups `hosts: dev:staging` or a
group intersection `hosts: staging:&database` (all database servers in staging). The supported patterns are:

| Action | Example usage |
| :------ | :---------- |
| All hosts | all | 
| All hosts | * | 
| Union | dev:staging | 
| Intersection | staging:&database | 
| Exclusion | dev:!queue | 
| Wildcard | *.example.com | 
| Range of numbered servers | web[5:10] |  
| Regular expression | ~web\d+\.example\.(com\|org) | 

### Limiting Which Hosts Run

Use the `-l` hosts or `--limit` hosts flag to tell Ansible to limit the hosts to run the playbook against the specified list of hosts:
`ansible-playbook -l hosts playbook.yml`.

### Running a Task on the Control Machine

Sometimes you want to run a particular task on the control machine instead of on the remote host. Ansible provides the _local\_action_ clause for
tasks to support this. If your play involves multiple hosts, and you use _local\_action_, the task will be executed multiple times, one for each host.
You can restrict this by using _run\_once_.

### Running a Task on a Machine Other Than the Host

Sometimes you want to run a task that's associated with a host, but you want to execute the task on a different server. Use the _delegate\_to_
clause to run the task on a different host.

### Running on One Host at a Time

By default, Ansible runs each task in parallel across all hosts. Sometimes you want to run your task on one host at a time. You can use the _serial_
clause on a play (outside the task, at the play level) to tell Ansible to restrict the number of hosts that a play runs on. Normally, when a task
fails, Ansible stops running tasks against the host that fails, but continues to run against other hosts. You can use a _max_fail\_percentage_
clause along with the _serial_ clause to specify the maximum percentage of failed hosts before Ansible fails the entire play.

### Running on a Batch of Hosts at a Time

You can also pass serial a percentage value instead of a fixed number, like in `serial:50%`, or you can determine the numer of percentage of hosts to
run in every iteration by passing a list, like in:

```yaml
serial:
  -1
  - 30%
```

### Running Only Once

Sometimes you might want a task to run only once, even if there are multiple hosts. You can use the run_once clause to tell Ansible to run the command
only once (particularly useful when using local_action if your playbook involves multiple hosts, and you want to run the local task only once):

```yaml
- name: run the database migrations
  command: /opt/run_migrations
  run_once: true
```

### Running Strategies

The _strategy_ clause on a play level gives you additional control over how Ansible behaves per task for all hosts. Available strategies are:

    * linear (default): This is the strategy in which Ansible executes one task on all hosts and waits until the task has completed or failed on 
      all hosts before it executes the next task on all hosts.
    * Free: Ansible will not wait for results of the task to execute on all hosts. Instead, if a host com‐ pletes one task, Ansible will execute 
      the next task on that host.

### Advanced Handlers

#### Handlers in Pre and Post Tasks

Handlers are usually executed after all tasks, once, and only when they get notified. But keep in mind there are not only _tasks_, but _pre\_tasks_,
_tasks_, and _post\_tasks_. Each tasks section in a playbook is handled separately, any handler notified in _pre\_tasks_, _tasks_, or
_post\_tasks_ is executed at the end of each section. A handler can be executed several times in one play.

#### Flush Handlers

Ansible lets us control the execution point of the handlers with the help of a special module, _meta_. i.e. if we want to execute the handlers
'my_handler' in between task1 and task2, we can configure it like this:

```yaml
- name: task1
  ...

- name: My Handler execution
  meta: my_handler

- name: task2
  ...
```

#### Handlers Listen

The listen clause defines what we'll call an event, on which one or more handlers can listen. This decouples the task notification key from the
handlers name. To notify more handlers to the same event, we just let these additional handlers listen on the same event, and they will also get
notified.

```yaml
---
- hosts: mailservers
  tasks:
    - copy:
        src: main.conf
        dest: /etc/postfix/main.cnf
      notify: postfix config changed
  handlers:
    - name: restart postfix
      service: name=postfix state=restarted
      listen: postfix config changed
```

### Manually Gathering Facts

Ansible will try to SSH to the host to gather facts before running the first tasks. We can dissable this behaviour using _gather\_facts_ set to false
and then explicitly invoke the _setup_ module to get Ansible to gather our facts:

```yaml
 tasks:
   - name: wait for ssh server to be running
     local_action: wait_for port=22 host="{{ inventory_hostname }}" search_regex=OpenSSH

   - name: gather facts
     setup:
   # The rest of the tasks go here
```

### Retrieving the IP Address from the Host

Ansible retrieves the IP address of each host and stores it as a fact. Each network interface has an associated Ansible fact. Using the name of the
web interfaces, we can define our variables like this:

```yaml
live_hostname: "{{ ansible_eth1.ipv4.address }}.com"
domains:
  - "{{ ansible_eth1.ipv4.address }}.com"
  - "www.{{ ansible_eth1.ipv4.address }}.com"
```

## Chapter 10: Callback Plugins<a name="Chapter10"></a>

Ansible supports a feature called callback plugins that can perform custom actions in response to Ansible events such as a play starting or a task
completing on a host (the output you see in your terminal when you execute an Ansible playbook is implemented as a callback plugin).

### Stdout Plugins

A stdout plugin controls the format of the output displayed to the terminal. Only a single stdout plugin can be active at a time. You specify a stdout
callback by setting the _stdout\_callback_ parameter in the defaults section of _ansible.cfg_, like in:

```ini
[defaults]
stdout_callback = actionable
```

| Name | Description |
| :--- | :---------- |
| actionable | Show only changed or failed (makes the output less noisy) | 
| debug | Human-readable stderr and stdout | 
| default | Show default output | 
| dense | Overwrite output instead of scrolling (always shows only two lines of output. It overwrites the existing lines rather than scrolling) | 
| json | JSON output (will not generate output until the entire playbook has finished executing) | 
| minimal | Show task results with minimal formatting | 
| oneline | Like minimal, but on a single line | 
| selective | Show only output for tagged tasks (shows output only for successful tasks that have the print_action tag) | 
| skippy | Suppress output for skipped hosts |

### Other Plugins

The other plugins perform a variety of actions, unlike with stdout plugins, you can have multiple other plugins enabled at the same time. Set
_callback\_whitelist_ in _ansible.cfg_ with a comma-separated list of other plugins:

```ini
[defaults]
callback_whitelist = mail, slack
```

| Name | Description |
| :--- | :---------- |
| foreman | end notifications to Foreman | 
| hipchat | Send notifications to HipChat | 
| jabber | Send notifications to Jabber |
| junit | Write JUnit-formatted XML file |
| log_plays | Log playbook results per hosts in /var/log/ansible/hosts |
| logentries | Send notifications to Logentries |
| logstash | Send results to Logstash |
| mail | Send email when tasks fail |
| osx_say | Speak notifications on macOS |
| profile_tasks | Report execution time of individual tasks and total execution time for the playbook |
| slack | Send notifications to Slack |
| time | Report total execution time (better use the profile_task plugin) |

#### foreman plugin environment variables

| Environment var | Description | Default |
| :-------------- | :---------- | :------ |
| FOREMAN_URL | URL to the Foreman server | http://localhost:3000 |
| FOREMAN_SSL_CERT | X509 certificate to authenticate to Foreman if HTTPS is used | /etc/foreman/client_cert.pem |
| FOREMAN_SSL_KEY | The corresponding private key | /etc/foreman/client_key.pem |
| FOREMAN_SSL_VERIFY | To verify the Foreman certificate. 1 to verify using the installed CAs or to a path pointing to a CA bundle. 0 to disable | 1 |

#### hipchat plugin environment variables

| Environment var | Description | Default |
| :-------------- | :---------- | :------ |
| HIPCHAT_TOKEN | HipChat API token | (None) |
| HIPCHAT_ROOM | HipChat room to post in | ansible |
| HIPCHAT_NAME | HipChat name to post as | ansible |
| HIPCHAT_NOTIFY | Add notify flag to important messages | true |

#### jabber plugin environment variables

| Environment var | Description |
| :-------------- | :---------- |
| JABBER_SERV | Hostname of Jabber server |
| JABBER_USER| Jabber username for auth |
| JABBER_PASS | Jabber password auth |
| JABBER_TO | Jabber user to send the notification to |

#### junit plugin environment variables

| Environment var | Description | Default |
| :-------------- | :---------- | :------ |
| JUNIT_OUTPUT_DIR | Destination directory for files | ~/.ansible.log |
| JUNIT_TASK_CLASS | Configure output: one class per YAML file | false |

#### logentries plugin environment variables

| Environment var | Description | Default |
| :-------------- | :---------- | :------ |
| LOGENTRIES_ANSIBLE_TOKEN | Logentries token | (None) |
| LOGENTRIES_API | Hostname of Logentries endpoint | data.logentries.com |
| LOGENTRIES_PORT | Logentries port | 80 |
| LOGENTRIES_TLS_PORT | Logentries TLS port | 443 |
| LOGENTRIES_USE_TLS | Use TLS with Logentries | false |
| LOGENTRIES_FLATTEN | Flatten results | false |

#### logstash plugin environment variables

| Environment var | Description | Default |
| :-------------- | :---------- | :------ |
| LOGSTASH_SERVER | Logstash server hostname | localhost |
| LOGSTASH_PORT | Logstash server port | 5000 |
| LOGSTASH_TYPE | Message type | ansible |

#### Mail plugin environment variables

| Environment var | Description | Default |
| :-------------- | :---------- | :------ |
| SMTPHOST | SMTP server hostname | localhost |

#### slack plugin environment variables

| Environment var | Description | Default |
| :-------------- | :---------- | :------ |
| SLACK_WEBHOOK_URL | Slack webhook URL | (None) |
| SLACK_CHANNEL | Slack room to post in | #ansible |
| SLACK_USERNAME | Username to post as | ansible |
| SLACK_INVOCATION | Show command-line invocation details | false |

## Chapter 11: Making Ansible Go Even Faster<a name="Chapter11"></a>

### SSH Multiplexing and ControlPersist

Because the SSH protocol runs on top of the TCP protocol, when you make a connection to a remote machine with SSH, you need to make a new TCP
connection. The client and server have to negotiate this connection before you can actually start doing useful work. OpenSSH (most common ssh client)
supports an optimization called SSH multiplexing, which is also referred to as _ControlPersist_. With this optimization, SSH sessions to the same host
will share the same TCP connection, so TCP connection negotiation happens only the first time. This is possible because OpenSSH uses a live socket
connection named _control socket_, which is user-configurable for an amount of time, usually 60 seconds.

### SSH Multiplexing Options in Ansible

The ControlPath /tmp/%r@%h:%p line tells SSH where to put the control Unix domain socket file on the filesystem. %h is the target hostname, %r is the
remote login username, and %p is the port. You can check whether a master connection is open by using the -O check
flag: `ssh -O check user@myserver.com`. Ansible's SSH multiplexing options

| Option | Value |
| :----- | :---- |
| ControlMaster | auto |
| ControlPath | $HOME/.ansible/cp/ansible-ssh-%h-%p-%r | 
| ControlPersist | 60s |

The operating system sets a maximum length on the path of a Unix domain socket, and if the ControlPath string is too long, then multiplexing won't
work. This is a common occurrence when connecting to Amazon EC2 instances, because EC2 uses long hostnames. You can change this path on the
_ansible.cfg_ config file:

```ini
[ssh_connection]
control_path = %(directory)s/%%h-%%r
```

Ansible sets %(directory)s to $HOME/.ansible/cp, and the double percent signs (%%) are needed to escape these characters because percent signs are
special characters for files in _.ini_ format.

### Pipelining

Ansible steps to execute a task are:

    1. Generate a Python script based on the module being invoked
    2. Copy the Python script to the host
    3. Execute the Python script

Ansible supports an optimization called *pipelining*, whereby it executes the Python script by piping it to the SSH session instead of copying it.

#### Enabling Pipelining

Pipelining is off by default, to enable it you have to change the _ansible.cfg_ config file:

```ini
[defaults]
pipelining = True
```

#### Configuring Hosts for Pipelining

Make sure that _requiretty_ is not enabled in your _/etc/sudoers_ file on your hosts. If sudo on your hosts is configured to read from
_/etc/sudoers.d_, add a sudoers config file that disables the _requiretty_ restriction for the user you use SSH with. For example for the
_ansible_ user create the file _disable-requiretty_ with the line `Defaults:ansible !requiretty`.

### Fact Caching

If your play doesn't reference any Ansible facts, you can turn off fact gathering for that play. You can disable fact gathering by default by adding
the following to your _ansible.cfg_ file:

```ini
[defaults]
gathering = explicit
```

If you write plays that do reference facts, you can use fact caching so that Ansible gathers facts for a host only once. If fact caching is enabled,
Ansible will store facts in a cache the first time it connects to hosts (If you want to clear the fact cache before running a playbook, pass the
`--flush-cache` flag to ansible-playbook). Modify the _ansible.cfg_ with:

```ini
[defaults]
# Smart gathering tells Ansible to gather facts only if they are not present in the cache or if the cache has expired
gathering = smart
# 24-hour timeout, adjust if needed 
fact_caching_timeout = 86400
# You must specify a fact caching implementation available implementations are: JSON files, Redis and Memcached
fact_caching = ...
```

#### JSON File Fact-Caching Backend

With the JSON file fact-caching backend, Ansible will write the facts it gathers to files on your control machine. If the files are present on your
system, it will use those files instead of connecting to the host and gathering facts. Use the _fact\_caching\_connection_ configuration option to
specify a directory where Ansible should write the JSON files .

#### Redis Fact-Caching Backend

To enable fact caching by using the Redis backend, you need to install and run Redis on your control machine, install the Python Redis package and
modify _ansible.cfg_ to enable fact caching using Redis.

#### Memcached Fact-Caching Backend

To use Memcached, you need to install and run Memcached on your control machine, install the Python Memcached package and modify _ansible.cfg_ to
enable fact caching using Memcached.

### Parallelism

The level of parallelism that ansible uses to execute tasks is controlled by a parameter, which defaults to 5. This can be changed by setting the
environment variable _ANSIBLE\_FORKS_ before running the playbook or by adding the following to the _ansible.cfg_ file:

```ini
[defaults]
forks = 20
```

### Concurrent Tasks with Async

Ansible introduced support for asynchronous actions with the async clause to work around the problem of SSH timeouts. Tasks marked with async are not
blocking ansible from executing the next task defined:

```yaml
- name: clone Linus's git repo
  git:
    repo: git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
    dest: /home/vagrant/linux
  async: 3600
  poll: 0
  register: linux_clone
```

_poll_ set to 0 is used to tell Ansible that it should immediately move on to the next task.

## Chapter 12: Custom Modules<a name="Chapter12"></a>

### Where to Put Custom Modules

Ansible looks in the _library_ directory relative to the playbook.

### How Ansible Invokes Modules

Ansible will do the following:

    1. Generate a standalone Python script with the arguments (Python modules only): Ansible will generate a self-contained Python script that 
       injects helper code and the arguments
    2. Copy the module to the host: Usually under ~/.ansible/<random_id>/<your_module>
    3. Create an arguments file on the host (non-Python modules only): Usually under ~/.ansible/<random_id>/arguments
    4. Invoke the module on the host, passing the arguments file as an argument
    5. Parse the standard output of the module: Ansible expects modules to output JSON

#### Output Variables that Ansible Expects

Your module can return whatever variables you like, but Ansible has special treatment for certain returned variables.

    * changed (Boolean): Should be returned by all ansible modules. a changed variable. Indicates whether the module execution caused the host to 
      change the state. If a task has a notify clause to notify a handler, the notification will fire only if changed is true
    * failed (Boolean): Ansible will treat a task execution as a failure if this variable is set to true and will not run any further tasks against 
      the host that failed, unless the task has an ignore_errors or failed_when clause
    * msg: Use the msg variable to add a descriptive message that describes the reason that a module failed

### Implementing Modules in Python

Ansible provides the AnsibleMod ule Python class that makes it easier to parse the inputs, return outputs in JSON format and invoke external programs.
When writing a Python module, Ansible will inject the arguments directly into the generated Python file rather than require you to parse a separate
arguments file.

```python
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule


def my_module(arg0, arg1):
    echo_path = module.get_bin_path('echo', required=True)
    args = ["Output is:", arg0, str(arg1)]
    (rc, stdout, stderr) = module.run_command(args)
    return rc == 0


def main():
    module = AnsibleModule(
        argument_spec=dict(
            arg0=dict(required=True),
            arg1=dict(required=True, type='int')
        ),
        supports_check_mode=True
    )
    # In check mode, we take no action. Since this module never changes system state, we just return changed=False
    if module.check_mode:
        module.exit_json(changed=False)
    arg0 = module.params['arg0']
    arg1 = module.params['arg1']

    if my_module(arg0, arg1):
        module.exit_json(changed=False)

    else:
        msg = "Could not parse %s:%s" % (arg0, arg1)
        module.fail_json(msg=msg)


if __name__ == "__main__":
    main()
```

In the example above, you can see that Ansible instantiates an _AnsibleModule_ object by passing it an _argument_spec_, which is a dictionary in which
the keys are parameter names and the values are dictionaries that contain information about the parameters. Once you’ve declared an
_AnsibleModule_ object, you can access the values of the arguments through the params dictionary, like:

```python
module = AnsibleModule(...)
arg0 = module.params["arg0"]
arg1 = module.params["arg1"]
```

#### Importing the AnsibleModule Helper Class

Starting with Ansible 2.1.0, Ansible deploys a module to the host by sending a ZIP file containing the module file along with the imported helper
files. One consequence of this it that you can now explicitly import classes with `from ansible.module_utils.basic import AnsibleModule`.

#### Argument Options

For each argument to an Ansible module, you can specify several options:

    * required: The required option is the only option that you should always specify
    * Default: For arguments that have required=False set, you should generally specify a default value for that option
    * choices: The choices option allows you to restrict the allowed arguments to a predefined list
    * aliases: The aliases option allows you to use different names to refer to the same argument
    * type: The type option enables you to specify the type of an argument (Types supported are str, list, dict, bool, int and float). Lists are 
      comma-delimited and dictionaries can either be key=value pairs delimited by commas, or you can use JSON inline

#### AnsibleModule Initializer Parameters

The AnsibleModule initializer method takes various arguments:

    * argument_spec: This is a dictionary that contains the descriptions of the allowed arguments for the module
    * no_log: When Ansible executes a module on a host, the module will log output to the syslog which can be disabled with this parameter
    * check_invalid_arguments: Ansible will verify that all of the arguments that a user passed to a module unless this option is set to False
    * mutually_exclusive: The mutually_exclusive parameter is a list of arguments that cannot be specified during the same module invocation
    * required_one_of: The required_one_of parameter expects a list of arguments with at least one that must be passed to the module
    * bypass_checks: Before a module executes, Ansible first checks that all of the argument constraints are satisfied, dissable it with this option
    * add_file_common_args: Many modules create or modify a file and users might want to set some attributes on it, like owner, group or permissions

#### Returning Success or Failure

Use the _exit_json_ method to return success `module.exit_json(changed=False, msg="meaningful message")` and Use the _fail_json_ method to indicate
failure `module.fail_json(msg="Some error")`.

#### Invoking External Commands

The AnsibleModule class provides the run_command convenience method for calling an external program, which wraps the native Python subprocess module.
It accepts the following arguments:

| Argument | Type | Default |  Description |
| :------- | :--- | :------ | :----------- |
| args (default) | String or list of strings | (None) | The command to be executed (see the following section) |
| check_rc | Boolean | False | If true, will call fail_json if command returns a nonzero value | 
| close_fds | Boolean | True | Passes as close_fds argument to subprocess.Popen | 
| executable | String (path to program) | (None) | Passes as executable argument to subprocess.Popen |
| data | String | (None) | Send to stdin if child process | 
| binary_data | Boolean | False | If false and data is present, Ansible will send a newline to stdin after sending data |
| path_prefix | String (list of paths) | (None) | Colon-delimited list of paths to prepend toPATHenvironment variable |
| cwd | String (directory path) | (None) | If specified, Ansible will change to this directory before executing |
| use_unsafe_shell | Boolean | False | When set to true, the args is passed as a single string even if there is multiple tokens |

### Check Mode (Dry Run)

Ansible supports something called check mode, which is enabled when passing the `-C` or `--check` flag to ansible-playbook. When Ansible runs a
playbook in check mode, it will not make any changes to the hosts when it runs, but reports whether each task would have changed the host. To tell
Ansible that your module supports check mode, set _supports\_check\_mode_ to true in the AnsibleModule initializer method.

### Documenting Your Module

You should document your modules according to the Ansible project standards so that HTML documentation for your module will be correctly generated and
the ansible-doc program will display documentation for your module. Near the top of your module, define a string variable called _DOCUMENTATION_
that contains the documentation, and a string variable called _EXAMPLES_ that contains example usage.

### Debugging Your Module

The Ansible repository in GitHub contains a couple of scripts that allow you to invoke your module directly on your local machine, without having to
run it by using the ansible or ansible-playbook commands. Clone the Ansible repo: `git clone https://github.com/ansible/ansible.git --recursive`
and set up your environment variables `source ansible/hacking/env-setup`. After this you can invoke your module:
`ansible/hacking/test-module -m /path/to/my_module -a "arg0=Test arg1=12"`.

### Implementing the Module in Bash

You can write modules in other languages as well. For example in Bash:

```bash
#!/bin/bash
# WANT_JSON 
# The above 'WANT_JSON' tells Ansible that we want the input to be in JSON syntax

echo $1 $2
```

### Specifying an Alternative Location for Bash

Not all systems will have the Bash executable at _/bin/bash_. You can tell Ansible to look elsewhere for the Bash interpreter by setting the
_ansible\_bash\_interpreter_ variable on hosts that install it elsewhere. You can create a host variable (_server.example.com_ in this example) by
creating the file `host_vars/server.example.com` that contains the following `ansible_bash_interpreter: /custom_path/bash`

## Chapter 13: Vagrant<a name="Chapter13"></a>
