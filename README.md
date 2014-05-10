EC2 Rails server playbooks
==========================

Ansible Playbooks for creating EC2 boxes ready to host Rails apps.

Full instructions on http://wp.me/plrKl-mN

How to provision an instance
----------------------------

The basic command needed to create a new instance from scratch is this:

```bash
ansible-playbook provisioning.yml --extra-vars "rails_env=staging site=example.com" -i localhost
```


`provisioning.yml` is the main provisioning playbook. The configurable parameters are `rails_env` and `site`, which are self explanatory. We use `localhost` as the inventory because we do not yet know the remote hosts which we will use (we are creating it).

The configuration of the machines are stored in the file `roles/ec2_creation/vars/main.yml` and they follow a hierarchical pattern. The variable `default_values` has the default and most used parameters. Then we have two dictionaries, one for `staging` and one for `production`, each one with the different machines, and their specific options and overrides (for example, the `elastic_ip` which has to be unique). Finally, there is a key `instance_values` that serves as a shortcut variable. It will hold the calculated final options of the machine to be created based on the values read from the command line. It uses some `Jinja 2` black magic to get them, which you can read about in http://docs.ansible.com/playbooks_variables.html.


Common system values and tasks
------------------------------

The common stuff to be configured and installed on the instances is in the `common` role. You can find the different system values used in the `roles/common/vars/main.yml` file, such as the minimum `yum` packages to install, or the `ruby` and `passenger` versions to be used.


How to add an ssh key
---------------------
The ssh keys are set up in the common role. The list of public ssh keys to use is in the `roles/common/files/ssh_keys` folder and they are named using the user name.

The actual ssh `authorized_keys` configuration is done on the task `roles/common/tasks/rails_user.yml` and is marked with the tag `ssh_keys` so it can be run individually.

To add a new ssh user access, the key must be added as a separate file on the mentioned folder *and also added to the items list on the task*. Maybe in the future we can change the task so it uses all files from that folder automatically.

After having modified the playbook, you can run this command to add the new key (COMMENT TO BE REVIEW):

```bash
ansible-playbook -i production provisioning.yml --tags ssh_keys
```

