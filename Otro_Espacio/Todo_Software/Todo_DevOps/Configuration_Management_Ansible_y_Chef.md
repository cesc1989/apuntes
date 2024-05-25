# Configuration Management: Ansible y Chef
Inicialmente, esto iba a ser sobre Ansible, Chef, Puppet y SaltStack pero al final solo he trabajado con Ansible. Los demás quedan descartados, excepto Chef del cual hay un poquito de información.

## General
- Chef vs Puppet [https://www.upguard.com/articles/puppet-vs.-chef-revisited](https://www.upguard.com/articles/puppet-vs.-chef-revisited)
- Salt vs Chef [https://www.upguard.com/articles/salt-vs-chef](https://www.upguard.com/articles/salt-vs-chef)
- Salt vs Chef vs Puppet vs Fabric vs Ansible [http://blog.takipi.com/deployment-management-tools-chef-vs-puppet-vs-ansible-vs-saltstack-vs-fabric/](http://blog.takipi.com/deployment-management-tools-chef-vs-puppet-vs-ansible-vs-saltstack-vs-fabric/)
- Ansible vs Chef [https://www.upguard.com/articles/ansible-vs-chef](https://www.upguard.com/articles/ansible-vs-chef)
## Chef

Based on [Tapiki blog post - 2015](https://blog.overops.com/deployment-management-tools-chef-vs-puppet-vs-ansible-vs-saltstack-vs-fabric/).

Chef Pros:

- Rich collection of modules and configuration recipes.
- Code-driven approach gives you more control and flexibility over your configurations.
- Being centered around Git gives it strong version control capabilities.
- ‘Knife’ tool (which uses SSH for deploying agents from workstation) eases installation burdens

Chef Cons:

- Learning curve is steep if you’re not already familiar with Ruby and procedural coding.
- It’s not a simple tool, which can lead to large code bases and complicated environments.
- Doesn’t support push functionality
## Ansible
- [Configuration Management Use Case](https://www.ansible.com/configuration-management) - Ansible website
- [Install](http://docs.ansible.com/ansible/intro_installation.html) - [On macOS using PIP](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip) - Ansible Docs
- [Terraform + Ansible](https://github.com/hashicorp/terraform/issues/2661) - Github Issue
- [RVM Ansible role](https://github.com/rvm/rvm1-ansible) - Github
- [Ansible + Vagrant tutorial](https://github.com/leucos/ansible-tuto) - Github
- [Using Vagrant and Ansible, official docs](https://docs.ansible.com/ansible/guide_vagrant.html) - Ansible Docs
- Install gem from task: [SO](http://stackoverflow.com/a/22121802/1407371) - [Gem module documentation](http://docs.ansible.com/ansible/gem_module.html)
- Servers for Hackers Ansible repo: [Github](https://github.com/Servers-for-Hackers/ansible-infrastructure)
- [NodeJS Role](https://github.com/novuso/ansible-role-nodejs)
- [Ansible AWS Modules](http://docs.ansible.com/ansible/list_of_cloud_modules.html) - [Ansible + AWS](https://www.ansible.com/aws) and [Let's Encrypt](http://docs.ansible.com/ansible/letsencrypt_module.html)
- YAML and Jinja2 variable interpolation: [Lines staring with](http://docs.ansible.com/ansible/playbooks_variables.html#hey-wait-a-yaml-gotcha) `[{{variable}}](http://docs.ansible.com/ansible/playbooks_variables.html#hey-wait-a-yaml-gotcha)` [should be quoted](http://docs.ansible.com/ansible/playbooks_variables.html#hey-wait-a-yaml-gotcha)
- [How to Manage Multistage Environments with Ansible](https://www.digitalocean.com/community/tutorials/how-to-manage-multistage-environments-with-ansible)
- `[when](http://docs.ansible.com/ansible/latest/playbooks_conditionals.html#the-when-statement)` [statement and](http://docs.ansible.com/ansible/latest/playbooks_conditionals.html#the-when-statement) `[ansible_os_family](http://docs.ansible.com/ansible/latest/playbooks_conditionals.html#the-when-statement)`
- `[ansible_os_family](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html#ansible-os-family)` [values](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html#ansible-os-family)

**Ansible Pros**

- SSH-based, so it doesn’t require installing any agents on remote nodes.
- Easy learning curve thanks to the use of YAML.
- Playbook structure is simple and clearly structured.
- Has a variable registration feature that enables tasks to register variables for later tasks
- Much more streamlined code base than some other tools

**Ansible Cons**

- Less powerful than tools based in other programming languages.
- Does its logic through its DSL, which means checking in on the documentation frequently until you learn it
- Variable registration is required for even basic functionality, which can make easier tasks more complicated
- Introspection is poor. Difficult to see the values of variables within the playbooks
- No consistency between formats of input, output, and config files
- Struggles with performance speed at times

Based on Tapiki blog [link](http://blog.takipi.com/deployment-management-tools-chef-vs-puppet-vs-ansible-vs-saltstack-vs-fabric/) - 2015


## Privilege Escalation Settings
- [Configuring Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html#privilege-escalation-settings).

Viejo: `sudo`/`su`
Nuevo: `become`, `become_user`, `become_method`

## Ansible command module vs Ansible shell module
- [**shell**](http://docs.ansible.com/ansible/shell_module.html) module executes on `/bin/sh`
- [**command**](http://docs.ansible.com/ansible/command_module.html#command) module is not processed through shell
    > so variables like $HOME and operations like "<", ">", "|", ";" and "&" will not work


## Specify Ansible hosts file and ping servers


    ansible [HOSTS] -m ping -i [PATH/TO/FILE] -u[USERNAME] --private-key=[PATH/TO/KEYFILE]

Example

    ansible arack -m ping -i ./provisioning/hosts -u ubuntu --private-key=~/.ssh/atlanticrack.pem


## Handlers
- [Docs about Handlers](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#handlers-running-operations-on-change).
- Handler names and listen topics live in a global namespace.
## Servers for Hackers

Lots of contents about Ansible

**Videos**

- [Installation and Basics](https://serversforhackers.com/video/ansible-installation-and-basics)
- [Ansible Playbooks](https://serversforhackers.com/video/ansible-playbooks)
- [Ansible Roles](https://serversforhackers.com/video/ansible-roles)
- [Ansible Vault](https://serversforhackers.com/video/ansible-using-vault)

**Articles**

- [An Ansible Tutorial](https://serversforhackers.com/an-ansible-tutorial)


## Errors and Troubleshooting

No recuerdo de que iba esto pero tiene que ver con NodeJS.


    - name: Add Nodesource Keys
      become: True
      apt_key: url=http://deb.nodesource.com/gpgkey/nodesource.gpg.key state=present

Issues:

- [Drupal VM](https://github.com/geerlingguy/drupal-vm/issues/844)
- [Ansible](https://github.com/ansible/ansible/issues/12161)

