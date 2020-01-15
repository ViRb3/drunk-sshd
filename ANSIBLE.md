# Ansible
## Ansible remote patching
This repo contains a pre-made Ansible [playbook](ansible-playbook.yml). To use it, run something like:
```
ansible-playbook ansible-playbook.yml
```
> TODO: Preserve sshd timestamp

### Password authentication
Refer to the template [inventory file](inventory.ini). To run the playbook with it:
```
ansible-playbook -i inventory.ini ansible-playbook.yml
```

### Handling sudo password prompt
Check the `vars` > `ansible_become_pass` property in the [playbook](ansible-playbook.yml)