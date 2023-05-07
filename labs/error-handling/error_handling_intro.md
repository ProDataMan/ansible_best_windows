# Ansible Error Handling
## Scenario

We have to set up automation to pull down a data file, from a notoriously unreliable third-party system, for integration purposes. Create a playbook that attempts to download https://bit.ly/3dtJtR7 and save it as `transaction_list` to `localhost`. The playbook should gracefully handle the site being down by outputting the message "Site appears to be down. Try again later." to stdout. If the task succeeds, the playbook should write "File downloaded." to stdout. No matter if the playbook errors or not, it should always output "Attempt completed." to stdout.

If the report is collected, the playbook should write and edit the file to replace all occurrences of `#BLANKLINE` with a line break '\n'.

## Create playbook

### Prerequisites

Create a playbook named `report.yml`

In the VS Code Explorer pane:

Right Click in the explorer pane

Select New Folder

Name the new folder 'error-handling'

Right Click on the `error-handling` folder

Select New File

Name the new file 'report.yml'

Paste the code below into the file

First, we'll specify our **host** and **tasks** (**name**, and **debug** message):

```yaml
---
- hosts: localhost
  tasks:
    - name: download transaction_list
      get_url:
        url: https://bit.ly/3dtJtR7
        dest: /home/ubuntu/ansible-working/error-handling/transaction_list
    - debug: msg="File downloaded"
```
## Copy files the maint folder from class files

In windows Explorer copy the `c:\GitRepos\ansible-best-practices-windows\error-handling\maint\` folder to `c:\GitRepos\ansible-working\error-handling\maint\`


### Add connection failure logic

We need to reconfigure a bit here, adding a **block** keyword and a **rescue**, in case the URL we're reaching out to is down:

In Visual Studio Code edit the report.yml file to include the updates below

```yaml
---
- hosts: localhost
  tasks:
    - name: download transaction_list
      block:
        - get_url:
            url: https://bit.ly/3dtJtR7
            dest: /home/ubuntu/ansible-working/error-handling/transaction_list
        - debug: msg="File downloaded"
      rescue:
        - debug: msg="Site appears to be down.  Try again later."
```



### Add an always message

An **always** block here will let us know that the playbook at least made an attempt to download the file:

In Visual Studio Code edit the report.yml file to include the updates below

```yaml
---
- hosts: localhost
  tasks:
    - name: download transaction_list
      block:
        - get_url:
            url: https://bit.ly/3dtJtR7
            dest: /home/ubuntu/ansible-working/error-handling/transaction_list
        - debug: msg="File downloaded"
      rescue:
        - debug: msg="Site appears to be down.  Try again later."
      always:
        - debug: msg="Attempt completed."
```

### Replace '#BLANKLINE' with '\n'

We can use the **replace** module for this task, and we'll sneak it in between the **get_url** and first **debug** tasks.

In Visual Studio Code edit the report.yml file to include the updates below

```yaml
---
- hosts: localhost
  tasks:
    - name: download transaction_list
      block:
        - get_url:
            url: https://bit.ly/3dtJtR7
            dest: /home/ubuntu/ansible-working/error-handling/transaction_list
        - replace:
            path: /home/ubuntu/ansible-working/error-handling/transaction_list
            regexp: "#BLANKLINE"
            replace: '\n'
        - debug: msg="File downloaded"
      rescue:
        - debug: msg="Site appears to be down.  Try again later."
      always:
        - debug: msg="Attempt completed."
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

## Run the playbook 

```
ansible-playbook /home/ubuntu/ansible-working/error-handling/report.yml
```

If all went well, we can read the downloaded text file:

```
cat /home/ubuntu/ansible-working/error-handling/transaction_list
```

After confirming the playbook successfully downloads and updates the `transaction_list` file, pull the latest changes from the repository, and run the `break_stuff.yml` playbook in the `maint` directory to simulate an unreachable host. 


```sh
ansible-playbook ~/ansible-working/lab-error-handling/maint/break_stuff.yml --tags service_down
```

Confirm the host is no longer reachable 
```sh
curl -L -o transaction_list https://bit.ly/3dtJtR7
```

Run the playbook again and confirm it gracefully handles the failure.



Restore the service using `break_stuff.yml`, and confirm the `report.yml` playbook reports the service is back online.

```
ansible-playbook ~/ansible-best-practices/labs/error-handling/maint/break_stuff.yml --tags service_up
```

```
ansible-playbook /home/ubuntu/lab-error-handling/report.yml
```



## Congrats!

