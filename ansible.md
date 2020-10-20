# Ansible

### Ansible Playbook commands

#### Bootstrap an individual node, without an inventory file

`$ ansible-playbook -K -k -u jay -i "phantom.local.lan," bootstrap_ubuntu.yml`

---

### Ansible Server commands

#### Ping all hosts

  `$ ansible all -m ping`

#### Run a command against all hosts

  `$ ansible all -s -m shell -a 'apt-get update'`

#### Install a package on all hosts

  `$ ansible all -s -m apt -a 'pkg=frozen-bubble state=installed update_cache=true'`

---

### Ansible Vault

#### Create an encrypted file

  `$ ansible-vault create filename.yml`

#### Encrypt an existing file

  `$ ansible-vault encrypt filename.yml`

#### Decrypt a file

  `$ ansible-vault decrypt filename.yml`

#### View an encrypted file

  `$ ansible-vault view filename.yml`

#### Edit an encrypted vault file

  `$ ansible-vault edit filename.yml`

#### Reset Ansible vault password

  `$ ansible-vault rekey filename.yml`

#### Decrypt content at run time in an Ansible playbook

  `$ ansible-playbook filename.yml --vault-id @prompt`

#### Use vault password from a file

  `$ ansible-playbook filename.yml --vault-id /etc/ansible/vault_pass.txt`

---

### Misc

#### Check a playbook

  `$ ansible-playbook playbook.yml --check`

#### Dynamic inventory

If the inventory file is marked as executable, ansible will execute it rather than just reading it. The inventory file that is used is set in ansible.cfg. It must support the following two command-line flags:

    --host=<hostname> (for showing host details)
    --list (for listing groups)

#### Building ansible from source

Install dependencies: `asciidoc cdbs devscripts`

  `$ cd /usr/src`
  `$ git clone --recursive https://github.com/ansible/ansible.git`
  `$ git checkout tags/v2.1.1.0-1`
  `$ git submodule update`
  `$ make DEB_DIST=jessie deb`

---

### Useful sites

#### Ansible vault examples

<https://www.linuxtechi.com/use-ansible-vault-secure-sensitive-data/>
