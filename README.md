# newrelic-ansible-tutorial

I am doing this in a MacOS terminal.


let's confirm we have ansible installed on our laptop
```
ansible-playbook --version
```
let's create new folder to hold the code we need to deploy newrelic with ansible.
```
mkdir newlic_ansible
```

let's move into that folder
```
cd newlic_ansible
```

let's create a folder to store our ansible roles, in our case just the newrelic role.
```
mkdir roles
ansible-galaxy install newrelic.newrelic-infra -p roles 
```

let's make the folder to hold our configuration files
fix your ssh user and password as needed. You will need an account with passwordless sudo access.

```
mkdir group_vars
cat <<EOF >> group_vars/all
---
# file: group_vars/all
ansible_connection: ssh
ansible_ssh_user: vagrant
ansible_ssh_pass: vagrant
EOF
```
confirm the content of the file
```
cat group_vars/all
```
let's set some sane default ssh configuration for ansible
```
cat <<EOF >> ansible.cfg
[defaults]
host_key_checking = False
ssh_args = -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no
EOF
```
confirm the content of the file
```
cat ansible.cfg
```

Now we need to create a playbook file to deploy newrelic to our hosts.
You may create a file with the Following content. Adjust as necessary
see the roles/newrelic.newrelic-infra/meta/main.yml file for supported Operating Systems.
I am calling my host groups "frontends" in the example below.
'*' for agent version means latest. You may insert a specific version number if needed x.y.z
new relic will log to the /var/log/nr-infra.log. Note that if you change this the folder should exist

```
cat <<EOF >> setup.yml
- name: NewRelic
  hosts: frontends
  become: true
  roles:
    - name: newrelic.newrelic-infra
      vars:
        nrinfragent_os_name: Debian 
        nrinfragent_os_version: trusty
        nrinfragent_version: '*'
        nrinfragent_config:
          license_key: INSERT_YOUR_LICENCE_KEY_HERE
          log_file: /var/log/nr-infra.log
          log_to_stdout: false
EOF
```
let's confirm the content of the file
```
cat setup.yml
```

Now we can finally create the file describing the list of the hosts we want to deploy to.
```
cat <<EOF >> hosts
[frontends]
192.168.0.83
192.168.0.84
EOF
```
let's confirm the content of the file
```
cat hosts
```

we can now run ansible with the following command:
```
ansible-playbook -i hosts setup.yml
```

within five minutes the hosts should show up in the dashboard.
