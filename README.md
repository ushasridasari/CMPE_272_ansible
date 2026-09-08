# Ansible Multi-VM Web Server Deployment

This project uses Ansible to deploy an nginx web server on two Ubuntu VMs created with Multipass.

- VM1 displays `Hello World from SJSU-1`
- VM2 displays `Hello World from SJSU-2`
## Prerequisites

- macOS
- Homebrew
- Multipass
- Ansible
- `community.general` Ansible collection

Install the required tools:

```
brew install --cask multipass
brew install ansible
ansible-galaxy collection install community.general
```

## Project Files

```
ansible.cfg
inventory.ini
site.yml
deploy.yml
undeploy.yml
templates/index.html.j2
templates/sjsu.conf.j2
```

## Setup

Create two Ubuntu VMs:

```
multipass launch 22.04 --name vm1 --cpus 1 --memory 1G --disk 5G
multipass launch 22.04 --name vm2 --cpus 1 --memory 1G --disk 5G
multipass list
```

Update `inventory.ini` with the IP addresses shown by `multipass list`:

```
[webservers]
vm1 ansible_host=VM1_IP sjsu_id=1
vm2 ansible_host=VM2_IP sjsu_id=2

[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/sjsu_lab
```

Make sure SSH access is configured for both VMs.

## Run the Deployment

Check connectivity:

```
ansible webservers -m ping
```

Check the playbook syntax:

```
ansible-playbook site.yml --syntax-check
```

Deploy nginx:

```
ansible-playbook site.yml --tags deploy
```

## Verify the Servers

Open the following URLs in a browser or use `curl`:

```
curl http://VM1_IP:8080
curl http://VM2_IP:8080
```

The responses should identify SJSU-1 and SJSU-2.

Running the deployment again should be idempotent:

```
ansible-playbook site.yml --tags deploy
```

## Remove the Deployment

To stop nginx, remove the web files, delete the firewall rule, and uninstall nginx:

```
ansible-playbook site.yml --tags undeploy
```

## Notes

- Always run `site.yml` with either the `deploy` or `undeploy` tag.
    
- Multipass IP addresses may change after restarting the VMs. Update `inventory.ini` if needed.
