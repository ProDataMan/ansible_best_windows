# Ansible Best Practices for Windows - Inventory Lab Exercise

## Objective

The objective of this lab exercise is to familiarize yourself with the best practices of using Ansible for Windows and to apply the following:

- Use of [win_ping module](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_ping_module.html) to ensure connectivity between the Ansible control node and the Windows host
- Use of [win_feature module](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_feature_module.html) to install or remove Windows features
- Use of [win_iis_website module](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_iis_website_module.html) to manage IIS websites and application pools

## Lab Environment

This lab requires the following:

- A Windows Server 2019 or Windows Server 2022 host with IIS installed
- An Ansible control node with the following:
  - Ansible 2.9 or later installed
  - Access to the Windows host via WinRM

## Provisioning an IIS Server using Ansible

These instructions will guide you through the process of provisioning an IIS (Internet Information Services) server using Ansible on a Windows host.

### Prerequisites

- A Windows host running IIS
- Python 3 installed on the Windows host
- Pip3 installed on the Windows host

### Inventory File
Perform the following steps in VS Code on Windows Target 1

In the VS Code Explorer pane:

1. Right Click in the explorer pane
1. Select `New File`
1. Name the new file 'inventory_vars.yml'
1. Paste the code below into the file


Create an inventory file with the following content:

```yml
all:
  hosts:
    win_node1:
      ansible_host: <IP or hostname of the IIS server>:
      ansible_connection: winrm
      ansible_winrm_transport: ntlm
      ansible_winrm_server_cert_validation: ignore
      ansible_user: <username with administrative privileges>
      ansible_password: <password for the username>
```
Replace `<IP or hostname of the IIS server>` with the IP address or hostname of your IIS server, and replace `<username with administrative privileges>` and `<password for the username>` with the username and password of an account with administrative privileges on the IIS server.

This format is great for a single host but imagine you have 10s or 100s of host repeating this information could get tiresome
Try using variables instead:

Update the inventory file with the following content:

```yml
all:
  hosts:
    win_node1:
      ansible_host: <IP or hostname of the IIS server>:
    win_node2:
      ansible_host: <IP or hostname of the IIS server>:
    win_node3:
      ansible_host: <IP or hostname of the IIS server>:
    win_node4:
      ansible_host: <IP or hostname of the IIS server>:
    win_node5:
    ansible_host: <IP or hostname of the IIS server>:    
  vars:
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
    ansible_winrm_server_cert_validation: ignore
    ansible_user: <username with administrative privileges>
    ansible_password: <password for the username>
```

> Notice that by moving all repeaded elements into the vars: section we can reduce duplication when create multiple host entries

### Commit and Push Changes to GitHub

1. In the sidebar, click on the "Source Control" icon (it looks like a branch).
2. In the "Source Control" pane, review the changes you made to the file.
3. Enter a commit message that describes the changes you made.
4. Click the checkmark icon to commit the changes.
5. Click on the "..." menu in the "Source Control" pane, and select "Push" to push the changes to GitHub.

## Update the Ansible Control Host

1. Return to the connection to your Ansible control host in PuTTY on Windows Target 1.
2. Navigate to the directory where you cloned repository.
3. Run `git pull` to update the repository on the control host.

### Running Ad-hoc Commands

To run ad-hoc commands on the IIS server, use the following command:

```bash
ansible all -i <path to inventory file> -m <module name> -a "<module arguments>"
```

Replace `<path to inventory file>` with the path to your inventory file, `<module name>` with the name of the Ansible module you want to use (e.g. `win_command`), and `<module arguments>` with the arguments for the module.

For example, to run the `win_command` module to execute the `whoami` command on the IIS server, use the following command:

```bash
ansible all -i inventory_vars.yml -m win_command -a "whoami"
```
Or `win_feature` to install the `web-server` feature (IIS)

```
ansible all - inventory_vars.yml -m win_feature -a "Web-Server"
```
### Running Playbooks

To create a playbook that installs IIS using the `win_feature` module, create a YAML file with the following content:
Remember to create, save and sync the file in VS Code on Windows Target 1

In the VS Code Explorer pane:

1. Right Click in the explorer pane
1. Select `New File`
1. Name the new file 'install_iis.yml'
1. Paste the code below into the file

```yml
- name: Install IIS
  hosts: all
  tasks:
    - name: Install IIS
      win_feature:
        name: Web-Server
        state: present
```

### Commit and Push Changes to GitHub

1. In the sidebar, click on the "Source Control" icon (it looks like a branch).
2. In the "Source Control" pane, review the changes you made to the file.
3. Enter a commit message that describes the changes you made.
4. Click the checkmark icon to commit the changes.
5. Click on the "..." menu in the "Source Control" pane, and select "Push" to push the changes to GitHub.

## Update the Ansible Control Host

1. Return to the connection to your Ansible control host in PuTTY on Windows Target 1.
2. Navigate to the directory where you cloned repository.
3. Run `git pull` to update the repository on the control host.
4. Execute the playbook.

```bash
ansible-playbook install_iis.yml -i inventory_vars.yml
```

This will install IIS on the target server.
To confirm the web server was installed open a browser and visit
```
http://localhost
```

### Conclusion

By following these instructions, you now have an IIS server provisioned and managed by Ansible on a Windows host.
