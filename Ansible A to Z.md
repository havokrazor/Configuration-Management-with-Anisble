# Ansible Practical

This session focuses on the **practical side of Ansible**. We will start from the basics and gradually move toward:

1. Installing Ansible
2. Setting up passwordless SSH authentication
3. Understanding Ansible ad hoc commands
4. Working with inventory files
5. Grouping servers
6. Writing an Ansible playbook
7. Running a playbook
8. Understanding verbose/debug output
9. Understanding Ansible roles
10. Creating roles using Ansible Galaxy
11. Understanding the structure of an Ansible role

---

# 1. Setting Up the Environment

For the practical exercises, a Linux machine is recommended.

Ubuntu is a good choice for beginners because the examples can be followed easily using its package manager.

The practical setup uses two servers:

### Ansible Server

This machine has Ansible installed and is used to execute Ansible commands and playbooks.

### Target Server

This is the machine that Ansible will configure.

The basic architecture is:

```text
+-------------------+          +-------------------+
|   Ansible Server  |  SSH --->|   Target Server   |
|                   |          |                   |
| Ansible installed |          | Being configured  |
+-------------------+          +-------------------+
```

Ansible can manage different types of target machines. The important requirement is that Ansible must be able to authenticate with the target machine without repeatedly asking for a password.

---

# 2. Installing Ansible

The first step when working with any tool is to install it. The official Ansible documentation provides installation instructions for different operating systems and distributions.

For Ubuntu, the easiest approach is to use the package manager.

First update the package information:

```bash
sudo apt update
```

`sudo` is required because package-management operations normally require root privileges.

After updating the package information, install Ansible:

```bash
sudo apt install ansible
```

Once the installation is complete, verify it:

```bash
ansible --version
```

If Ansible is installed correctly, the command displays information about the installed Ansible version.

## Why use the package manager?

Ansible can also be installed through Python and `pip`.

The generic installation process is useful because Ansible is based on Python, but for a beginner, the operating-system package manager is often simpler.

For example:

* Ubuntu → `apt`
* macOS → `brew`
* Windows → a suitable Windows package-management method

Using the system package manager also makes it easier for the executable to be available through the system `PATH`.

---

# 3. Setting Up Passwordless SSH Authentication

The next important step is configuring communication between the Ansible server and the target server. Ansible needs to be able to connect to the target machine without manually entering a password every time.

This is generally accomplished using **SSH key-based authentication**.

Suppose we have:

```text
Ansible Server
       |
       | SSH
       v
Target Server
```

The Ansible server should be able to connect to the target server without a password prompt.

---

# 4. Understanding SSH Keys

On the Ansible server, generate an SSH key pair:

```bash
ssh-keygen
```

This generates a pair of keys:

* Private key
* Public key

A common location is:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

The important distinction is:

### Private Key

The private key must be kept secret. Do **not** share it with other people.

### Public Key

The public key is the key that can be placed on servers that you want to access.

The basic idea is:

```text
Ansible Server
    |
    | Public Key
    v
Target Server
    |
    | authorized_keys
```

---

# 5. Adding the Public Key to the Target Server

First display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the output. Now log in to the target server.

The target server has an SSH directory:

```bash
ls ~/.ssh
```

Among the files in this directory, you should find:

```text
authorized_keys
```

Open the file:

```bash
vi ~/.ssh/authorized_keys
```

Paste the public key from the Ansible server into this file and save it.

The public key from the Ansible server is now authorized on the target server.

---

# 6. Testing Passwordless Authentication

From the Ansible server, connect to the target server:

```bash
ssh <target-server-private-ip>
```

If everything is configured correctly, the connection should succeed without asking for the target server's password.

This is the prerequisite required for the practical Ansible exercises.

In simple terms:

> Copy the Ansible server's public key into the target server's `authorized_keys` file.

If you have multiple target servers, the same public key can be added to each target server.

For example:

```text
                    +--> Target Server 1
                    |
Ansible Server -----+--> Target Server 2
                    |
                    +--> Target Server 3
```

The Ansible host can then communicate with each of those machines using SSH key-based authentication.

---

# 7. What Is an Ansible Playbook?

Ansible playbooks are files that define the tasks Ansible should perform. The idea is similar to other scripting environments:

| Technology | Script/File  |
| ---------- | ------------ |
| Shell      | Shell script |
| Python     | Python file  |
| Ansible    | Playbook     |

However, you do **not** always need to create a playbook.

Just as you can execute a simple shell command directly without creating a shell script, Ansible also allows you to execute individual commands directly from the command line.

These are called **Ansible ad hoc commands**.

---

# 8. Ansible Ad Hoc Commands

Ad hoc commands are useful when you need to perform a small number of tasks. For example, if the requirement is simply:

> Create a file on the target server.

It would be unnecessary to create an entire playbook for such a simple operation. Instead, an Ansible ad hoc command can be used.

---

# 9. Inventory File

Before executing Ansible commands, Ansible needs to know which machines it should manage. This information is stored in an **inventory file**.

For example:

```text
172.31.62.28
```

The inventory contains the target server's IP address.

Ansible has a default inventory location, commonly:

```text
/etc/ansible/hosts
```

However, it is often convenient to maintain an inventory file alongside your project.

For example:

```text
inventory
```

An inventory containing multiple servers could look like:

```text
172.31.62.28
172.31.62.29
172.31.62.30
```

---

# 10. Running an Ad Hoc Command

The general structure is:

```bash
ansible -i <inventory-file> <target> -m <module> -a "<arguments>"
```

For example:

```bash
ansible -i inventory all -m shell -a "touch devops-class"
```

Here:

* `ansible` → Ansible command
* `-i` → Specifies the inventory
* `inventory` → Inventory file
* `all` → Target all hosts in the inventory
* `-m` → Specifies the module
* `shell` → Shell module
* `-a` → Provides arguments to the module
* `touch devops-class` → Command executed on the target

After running the command, Ansible reports whether the operation succeeded.

The created file can then be verified on the target server:

```bash
ls -ltr
```

---

# 11. Ad Hoc Commands vs Playbooks

This is an important interview question.

### Ad Hoc Commands

Ad hoc commands are appropriate for simple, one-off tasks.

For example:

```bash
ansible -i inventory all -m shell -a "df -h"
```

or:

```bash
ansible -i inventory all -m shell -a "nproc"
```

### Playbooks

Playbooks are useful when you have multiple tasks that need to be performed in a structured and repeatable manner.

A simple way to remember the distinction:

```text
Simple / one-off operation
        ↓
Ad hoc command

Multiple structured tasks
        ↓
Playbook
```

---

# 12. Finding Ansible Modules

Ansible contains a large number of modules. **You do not need to memorize every module.**

The recommended approach is to use the Ansible documentation. For example, if you want to perform a file-copy operation, search the Ansible modules documentation for the `copy` module.

The documentation provides:

* Module description
* Available parameters
* Examples
* Expected syntax

This makes it easier to construct commands correctly.

For example, the `copy` module can be used to copy files from the Ansible machine to target machines.

---

# 13. Grouping Servers in the Inventory

In real environments, you usually have many servers.

You might have:

* Database servers
* Web servers
* Application servers
* Monitoring servers

You don't always want to execute every task on every machine. Ansible allows you to group machines in the inventory.

For example:

```ini
[db_servers]
172.31.62.101
172.31.62.102

[web_servers]
172.31.62.100
172.31.62.103
```

The names inside square brackets define groups.

---

# 14. Executing Commands Against a Group

Instead of using `all` in the ansible command you can use the group name where you want the command to be executed, you can specify a group:

```bash
web_servers
```

For example:

```bash
ansible -i inventory web_servers -m shell -a "nproc"
```

Ansible will:

1. Read the inventory
2. Find the `web_servers` group
3. Identify the machines belonging to that group
4. Execute the command against those machines

This is especially useful when different types of servers require different configurations.

For example:

```text
Inventory
│
├── db_servers
│   ├── DB Server 1
│   └── DB Server 2
│
└── web_servers
    ├── Web Server 1
    └── Web Server 2
```

---

# 15. Important Interview Question: How Do You Group Servers?

> Servers can be grouped inside the Ansible inventory file. A playbook or ad hoc command can then target a specific group instead of targeting all hosts.

This allows tasks to be executed only against the required machines.

---

# 16. Writing the First Ansible Playbook

Now we can move from ad hoc commands to an actual playbook.

The scenario is:

1. Install Nginx 
2. Start Nginx 
on the target server

Since this contains multiple tasks, a playbook is appropriate.

Ansible playbooks use **YAML**.

Create a file such as:

```text
first-playbook.yaml
```

A playbook begins with:

```yaml
---
```

The three hyphens indicate the beginning of a YAML document.

---

# 17. Basic Playbook Structure

The basic structure is:

```yaml
---
- name: Install and start nginx
  hosts: all
  become: true

  tasks:
    - name: Install nginx
      ...

    - name: Start nginx
      ...
```

Let's understand each section.

---

**`name`** - The `name` field describes what the play or task is doing.

For example:

```yaml
- name: Install and start nginx
```

Task names should clearly describe the operation. Using meaningful names makes playbooks easier to understand.

---

**`hosts`** - The `hosts` field determines which machines should execute the play.

For example:

```yaml
hosts: all
```

means that the play should run against all hosts in the inventory. You can also specify an inventory group:

```yaml
hosts: web_servers
```

This would execute the play only against the hosts belonging to the `web_servers` group.

---

**`become`** -  Some operations require root privileges. For example, installing packages normally requires elevated privileges.

Ansible provides:

```yaml
become: true
```

This allows the tasks to be executed with elevated privileges. The idea is similar to using `sudo` manually.

For example:

```bash
sudo apt install nginx
```

becomes conceptually similar to using `become` in an Ansible playbook. You can also use privilege-escalation options to execute tasks as a particular user when required.

---

**Defining Tasks** -  Tasks are defined under:

```yaml
tasks:
```

Each task is a list item, so it begins with a hyphen.

For example:

```yaml
tasks:
  - name: Install nginx
    ...

  - name: Start nginx
    ...
```

The indentation is important because YAML relies heavily on indentation.

---

# 24. Installing Nginx

There are multiple ways to install Nginx.

One approach is to execute a shell command:

```yaml
- name: Install nginx
  shell: apt install nginx
```

However, Ansible provides a dedicated package-management module for this.

For Ubuntu, the `apt` module can be used:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

The `state: present` means that Nginx should be installed.

Using an Ansible module is generally preferable to manually constructing shell commands when an appropriate module exists.

---

# 25. Starting Nginx

Similarly, you could use a shell command:

```yaml
- name: Start nginx
  shell: systemctl start nginx
```

But Ansible provides a service module:

```yaml
- name: Start nginx
  service:
    name: nginx
    state: started
```

This describes the desired state rather than simply executing a shell command.

---

# 26. Complete First Playbook

The complete playbook can therefore be structured as:

```yaml
---
- name: Install and start nginx
  hosts: all
  become: true

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
```

This playbook contains two main tasks:

1. Install Nginx
2. Start Nginx

---

# 27. Why Use Modules Instead of Shell Commands?

You could execute:

```bash
apt install nginx
```

through the shell module.

However, when Ansible already provides a module for the operation, using the module is generally better.

For example:

```yaml
apt:
  name: nginx
  state: present
```

is preferable to:

```yaml
shell: apt install nginx
```

The module provides a structured way to describe the desired configuration.

The same principle applies to services:

```yaml
service:
  name: nginx
  state: started
```

rather than:

```yaml
shell: systemctl start nginx
```

---

# 28. Executing an Ansible Playbook

There is an important difference between the commands used for ad hoc operations and playbooks.

### Ad Hoc Commands

Use:

```bash
ansible
```

### Playbooks

Use:

```bash
ansible-playbook
```

For example:

```bash
ansible-playbook -i inventory first-playbook.yaml
```

Here:

* `ansible-playbook` → Executes an Ansible playbook
* `-i inventory` → Specifies the inventory
* `first-playbook.yaml` → Specifies the playbook to execute

If you use the default inventory location, you may not need to specify `-i`.

---

# 29. Understanding Playbook Execution

When the playbook is executed, Ansible performs several operations.

The first task you may see is:

```text
Gathering Facts
```

This happens before the tasks defined in the playbook.

Ansible collects information about the target machine.

This information can include details about:

* Operating system
* Network configuration
* Hardware
* Python environment
* Other system characteristics

After gathering facts, Ansible executes the tasks defined in the playbook.

In our example:

```text
Gathering Facts
      ↓
Install nginx
      ↓
Start nginx
```

---

# 30. Verifying Nginx

After the playbook finishes, connect to the target server and verify the service.

For example:

```bash
sudo systemctl status nginx
```

If everything worked correctly, the output should indicate that Nginx is running.

This confirms that the Ansible playbook successfully:

1. Installed Nginx
2. Started Nginx

---

# 31. Understanding Ansible Output

Ansible reports the status of each task.

Successful operations are displayed accordingly, while errors are clearly reported.

The output makes it possible to determine:

* Which task executed
* Whether it changed anything
* Whether the task failed
* Which hosts were affected

This is useful when troubleshooting playbooks.

---

# 32. Verbose Mode

Ansible provides verbosity options for understanding what is happening internally.

For example:

```bash
ansible-playbook -vvv -i inventory first-playbook.yaml
```

You can increase the verbosity level using additional `v` characters.

For example:

```text
-v
-vv
-vvv
```

Higher verbosity provides more debugging information.

---

# 33. Why Use Verbose Mode?

Normal output tells you whether the playbook succeeded.

Verbose output helps you understand **how Ansible performed the operation**.

It can show information about:

* Configuration being used
* SSH connections
* Python availability
* Fact gathering
* Modules being executed
* Arguments passed to modules
* Results returned by the target
* Detailed execution information

For example, Ansible may show that it established an SSH connection and then used Python on the target machine to execute the relevant module.

Verbose mode is especially useful while learning Ansible because it helps you understand what happens behind the scenes.

---

# 34. Ansible's Python Dependency

Ansible commonly relies on Python on the target machine for module execution.

Verbose output can help you see Ansible:

1. Establishing an SSH connection
2. Checking the target environment
3. Finding the required Python components
4. Gathering facts
5. Executing modules
6. Returning results

Understanding these internal operations becomes useful if you later want to develop or contribute Ansible modules yourself.

---

# 35. Moving Beyond Basic Playbooks

At this point, we have learned the basics:

* Install Ansible
* Configure SSH
* Create an inventory
* Run ad hoc commands
* Group servers
* Create playbooks
* Use modules
* Execute playbooks
* Use verbosity for debugging

However, real-world Ansible configurations can become significantly more complicated.

For example, consider a Kubernetes environment.

Suppose the requirement is:

```text
Create 3 EC2 instances
        |
        +--> Configure 1 as Kubernetes control plane
        |
        +--> Configure 2 as worker nodes
```

This involves multiple types of operations.

---

# 36. Terraform and Ansible

A common DevOps approach is to divide responsibilities between Terraform and Ansible.

### Terraform

Terraform is commonly used for infrastructure provisioning.

For example:

```text
Terraform
   |
   +--> Create EC2 instance 1
   +--> Create EC2 instance 2
   +--> Create EC2 instance 3
```

### Ansible

Ansible can then configure the machines:

```text
Ansible
   |
   +--> Configure control plane
   +--> Configure worker nodes
```

Although Ansible can also perform infrastructure-related operations, Terraform is specifically designed around infrastructure provisioning and infrastructure as code.

A useful analogy is that multiple tools might technically accomplish the same task, but you generally choose the tool that is designed specifically for that purpose.

---

# 37. The Problem With Large Playbooks

A Kubernetes configuration can contain many tasks.

Instead of two tasks like our Nginx example, a real configuration could contain:

* Package installation
* Configuration files
* Certificates
* Secrets
* Network configuration
* Runtime configuration
* Kubernetes components
* Control-plane configuration
* Worker-node configuration
* Service configuration
* Validation
* Error handling

A single playbook could easily grow to dozens of tasks.

Trying to put everything into one YAML file makes it difficult to:

* Read
* Maintain
* Reuse
* Debug
* Organize
* Update

This is where **Ansible Roles** become useful.

---

# 38. What Are Ansible Roles?

Ansible Roles provide a structured and efficient way to organize complex playbooks.

Instead of placing everything inside one large playbook, responsibilities can be separated into a predefined directory structure.

Conceptually:

```text
Large Playbook
      |
      +--> Tasks
      +--> Variables
      +--> Handlers
      +--> Files
      +--> Templates
      +--> Metadata
      +--> Tests
```

Roles make complex Ansible projects easier to maintain.

---

# 39. Creating an Ansible Role

Ansible Galaxy provides a command for initializing a role.

For example:

```bash
ansible-galaxy role init kubernetes
```

The `init` operation initializes the role structure.

After running the command, Ansible creates a directory:

```text
kubernetes/
```

Inside it, Ansible generates a number of directories and files.

You can inspect them with:

```bash
ls kubernetes
```

A typical role structure contains directories such as:

```text
kubernetes/
├── defaults/
├── files/
├── handlers/
├── meta/
├── tasks/
├── templates/
├── tests/
├── vars/
└── README.md
```

The generated structure provides a standard way of organizing the role.

---

# 40. Ansible Role Directory Structure

The main components are:

```text
roles/
└── kubernetes/
    ├── defaults/
    ├── files/
    ├── handlers/
    ├── meta/
    ├── tasks/
    ├── templates/
    ├── tests/
    ├── vars/
    └── README.md
```

Let's understand the purpose of each.

---

# 41. `tasks/`

The `tasks` directory contains the actual tasks performed by the role.

For example:

```text
tasks/
└── main.yml
```

Instead of placing all tasks directly inside the main playbook, the tasks can be organized inside the role.

This keeps the parent playbook much cleaner.

---

# 42. `defaults/`

The `defaults` directory is used for default variables.

For example:

```text
defaults/
└── main.yml
```

Variables that provide default values for the role can be stored here.

---

# 43. `vars/`

The `vars` directory is another location for variables.

For example:

```text
vars/
└── main.yml
```

Variables can therefore be separated from the task definitions.

The distinction between `defaults` and `vars` is an important topic to understand when working with Ansible roles.

---

# 44. `files/`

The `files` directory can contain files that need to be copied to target machines.

For example:

```text
files/
└── index.html
```

Other examples could include:

* Certificates
* Configuration files
* Static files
* Other files required by the role

A task can then use these files when configuring the target server.

---

# 45. `templates/`

The `templates` directory is used for templates.

Ansible uses the **Jinja2 templating system**.

Templates are useful when a configuration file needs dynamic values.

For example:

```text
templates/
└── nginx.conf.j2
```

A template can contain variables that Ansible fills in during execution.

---

# 46. `handlers/`

Handlers are used when you need to respond to particular changes or events.

For example, after changing an Nginx configuration, you may need to restart or reload Nginx.

A handler can be used for this type of action.

Handlers can also be used to handle situations where a particular response is required after a task changes something.

Conceptually:

```text
Task changes configuration
          |
          v
       Handler
          |
          v
 Reload / restart service
```

---

# 47. `tests/`

The `tests` directory is intended for testing the role.

Just as software projects can contain tests for their code, Ansible roles can also include testing-related files.

This makes it possible to validate that a role behaves as expected.

---

# 48. `meta/`

The `meta` directory contains metadata about the role.

Metadata can include information about:

* Role information
* Dependencies
* Licensing
* Other metadata associated with the role

This becomes particularly useful when roles are shared with the broader Ansible community.

---

# 49. `README.md`

The `README.md` file provides documentation for the role.

It can explain:

* What the role does
* How to use it
* Required variables
* Configuration
* Examples
* Other important information

This is especially useful when other people need to understand or reuse the role.

---

# 50. Parent Playbook and Roles

When using roles, the main playbook can remain relatively simple.

Instead of placing every task directly inside the parent playbook, the parent playbook can specify the host and reference the appropriate role.

Conceptually:

```text
Parent Playbook
       |
       +--> hosts
       |
       +--> roles
              |
              +--> Kubernetes role
                    |
                    +--> tasks
                    +--> handlers
                    +--> templates
                    +--> files
                    +--> variables
```

This separates the main orchestration from the detailed implementation.

---

# 51. Example Role-Based Organization

A project might look like:

```text
ansible-project/
├── playbook.yml
└── roles/
    └── jboss-standalone/
        ├── defaults/
        ├── files/
        ├── handlers/
        ├── meta/
        ├── tasks/
        ├── templates/
        ├── tests/
        ├── vars/
        └── README.md
```

The parent playbook can then reference the role rather than containing every task itself.

---

# 52. Why Roles Improve Ansible Projects

Roles are especially useful when the playbook becomes large.

They provide:

* Better organization
* Separation of responsibilities
* Reusability
* Easier maintenance
* Easier debugging
* Cleaner playbooks
* A standard directory structure

Instead of having one massive playbook, related components can be separated logically.

---

# 53. Practical Learning Path

The recommended progression is:

```text
1. Install Ansible
        ↓
2. Configure passwordless SSH
        ↓
3. Create an inventory
        ↓
4. Practice ad hoc commands
        ↓
5. Group servers
        ↓
6. Write basic playbooks
        ↓
7. Execute playbooks
        ↓
8. Use modules
        ↓
9. Learn verbose/debug output
        ↓
10. Learn roles
        ↓
11. Practice real-world role-based projects
```

Do not jump directly into complicated roles without understanding the basics first.

---

# 54. Practice Exercises

The practical exercises from this lesson should be repeated independently.

## Exercise 1 — Install Ansible

Install Ansible on a Linux machine:

```bash
sudo apt update
sudo apt install ansible
```

Verify:

```bash
ansible --version
```

---

## Exercise 2 — Configure SSH

Generate an SSH key:

```bash
ssh-keygen
```

Copy the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Add it to the target server's:

```text
~/.ssh/authorized_keys
```

Then verify:

```bash
ssh <target-ip>
```

The connection should work without a password.

---

## Exercise 3 — Create an Inventory

Create an inventory file:

```text
inventory
```

Add the target server's IP address.

Example:

```text
172.31.62.28
```

---

## Exercise 4 — Run an Ad Hoc Command

Try:

```bash
ansible -i inventory all -m shell -a "nproc"
```

Then:

```bash
ansible -i inventory all -m shell -a "df -h"
```

And create a file:

```bash
ansible -i inventory all -m shell -a "touch devops-class"
```

Verify the file on the target server.

---

## Exercise 5 — Create Inventory Groups

Create groups such as:

```ini
[db_servers]
172.31.62.101
172.31.62.102

[web_servers]
172.31.62.100
172.31.62.103
```

Run commands against only the web servers:

```bash
ansible -i inventory web_servers -m shell -a "nproc"
```

---

## Exercise 6 — Write the Nginx Playbook

Create:

```text
first-playbook.yaml
```

Write a playbook that:

1. Targets the required hosts
2. Uses privilege escalation
3. Installs Nginx
4. Starts Nginx

Execute it using:

```bash
ansible-playbook -i inventory first-playbook.yaml
```

Verify:

```bash
sudo systemctl status nginx
```

---

## Exercise 7 — Use Verbose Mode

Run the playbook with additional verbosity:

```bash
ansible-playbook -vvv -i inventory first-playbook.yaml
```

Observe:

* SSH connection details
* Fact gathering
* Python-related operations
* Module execution
* Task results

This is a useful way to understand what Ansible is doing internally.

---

## Exercise 8 — Create a Role

Initialize a role:

```bash
ansible-galaxy role init kubernetes
```

Inspect the generated structure:

```bash
ls kubernetes
```

Then explore:

```text
defaults/
files/
handlers/
meta/
tasks/
templates/
tests/
vars/
README.md
```

Understand what each directory is responsible for.

---

# 55. Important Concepts to Remember

### Ansible

An automation and configuration-management tool used to configure and manage machines.

### Inventory

Contains the hosts that Ansible manages.

### Ad Hoc Command

Used for simple, one-off operations.

### Playbook

A YAML file containing one or more structured tasks.

### Module

An Ansible component designed to perform a particular operation.

### `hosts`

Defines which inventory hosts the play should run against.

### `become`

Enables privilege escalation, commonly to the root user.

### Task

An individual operation inside a playbook.

### Role

A structured way of organizing complex Ansible automation.

### Ansible Galaxy

Provides tooling for creating and working with Ansible roles.

### Verbosity

Provides additional information about what Ansible is doing internally.

---

# 56. Interview Questions Covered by These Concepts

These concepts are useful for common Ansible interview questions, including:

1. What is Ansible?
2. What is an Ansible inventory?
3. What are Ansible ad hoc commands?
4. What is an Ansible playbook?
5. What is the difference between ad hoc commands and playbooks?
6. How do you group servers in Ansible?
7. How can you execute a task only against a particular group?
8. What are Ansible modules?
9. Why use Ansible modules instead of shell commands?
10. What is `become`?
11. What is passwordless SSH authentication?
12. What are Ansible roles?
13. Why are roles useful?
14. What is the structure of an Ansible role?
15. What are `tasks`, `handlers`, `defaults`, `vars`, `files`, and `templates`?
16. What is Ansible Galaxy?
17. How do you initialize an Ansible role?
18. How can you troubleshoot an Ansible playbook?

---

# 57. Key Takeaways

The main practical concepts from this lesson are:

```text
Ansible Installation
        ↓
Passwordless SSH
        ↓
Inventory
        ↓
Ad Hoc Commands
        ↓
Inventory Groups
        ↓
Playbooks
        ↓
Modules
        ↓
Privilege Escalation
        ↓
Playbook Execution
        ↓
Verbose Debugging
        ↓
Roles
        ↓
Ansible Galaxy
```

The most important thing at this stage is **practice**.

Start with simple ad hoc commands, then write your first playbook, and only after becoming comfortable with those concepts move toward Ansible roles and more complex real-world configurations.

For larger projects, roles provide the structure needed to keep playbooks organized, reusable, and maintainable.

---

# 58. Suggested Next Step

After completing these exercises, practice an existing role-based Ansible example.

A JBoss Standalone example is a useful starting point because it demonstrates how a more complex application configuration can be separated into an Ansible role rather than placing every task inside a single playbook.

Once you are comfortable with roles, move toward larger examples such as Kubernetes configuration.

For interview preparation, also review scenario-based Ansible questions involving:

* Roles
* Handlers
* Variables
* Inventory
* Modules
* Complex deployments
* Real-world automation scenarios

The goal is not to memorize every Ansible module. The goal is to understand **how Ansible is structured, how to find the correct module/documentation, how to organize automation, and how to troubleshoot what Ansible is doing**.
