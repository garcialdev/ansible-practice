# Automating Kubernetes VM Creation in Proxmox Using Ansible

This document explains how an Ansible playbook is used to:

* clone virtual machines from a Proxmox template
* assign static IP addresses via cloud-init
* wait for the VMs to boot
* dynamically add them to Ansible inventory
* configure hostnames
* install **Oh-My-Posh** automatically

This approach allows you to spin up Kubernetes nodes (or any Linux VMs) repeatedly and consistently.

---

## Prerequisites

Before running the playbook:

* A working **Proxmox VE** environment
* Ansible installed on your control machine
* A **cloud-init enabled template VM** in Proxmox
* API access credentials for Proxmox
* SSH access to VMs (cloud-init user)

❗The template must already be converted to template and prepared with cloud-init support.

---

## What the Playbook Does (High-Level Overview)

The playbook runs in **three separate plays**:

1. **Clone VMs from the Proxmox template**
2. **Wait for the VMs to boot and become reachable**
3. **Configure the newly created VMs**

---

## Variables Overview

Some key variables defined:

* `proxmox_host` – Proxmox API hostname or IP
* `proxmox_user` – Proxmox account (root@pam etc.)
* `template_name` – name of the template to clone
* `vm_list` – defines VM names and static IP addresses
* `cloud_user` / `cloud_password` – cloud-init login credentials
* `gateway` – default network gateway

Example VM list:

```yaml
vm_list:
  - name: k4
    ip: 10.0.0.121
  - name: k5
    ip: 10.0.0.122
```

---

## Play 1 — Clone VMs From Template

This section connects to Proxmox and clones the template.

Key actions:

* full clone of the template
* set CPU/memory resources
* configure static IP addressing
* enable **cloud-init user credentials**
* attach network to `vmbr0`

Static networking is configured using:

* `ipconfig`
* `gateway`
* cloud-init generated network files

Full clone is used to avoid linked clone storage issues:

```yaml
full: true
```

---

## Play 2 — Wait for SSH & Dynamically Add Hosts

After cloning, the VMs need time to:

* boot for the first time
* generate SSH keys
* apply cloud-init configuration

The play waits for **port 22** to respond.

Then Ansible:

* dynamically injects new VMs into inventory
* assigns SSH user (`ubuntu` in this case)

No manual inventory editing required.

---

## Play 3 — Configure the VMs

Once reachable, each VM is configured:

### ✔ Hostname is set

Keeping names consistent with VM names

### ✔ /etc/hosts updated

Prevents hostname resolution issues in clusters

### ✔ Prerequisites installed

Curl, wget, unzip, etc.

### ✔ Oh-My-Posh installed

The shell is customized automatically using:

* install script
* Meslo font
* Catppuccin theme
* `.bashrc` customization

After relogin, prompt theme is active.

---

## Why This Approach Is Useful

This workflow is ideal when building:

* Kubernetes clusters
* lab environments
* repeatable VM deployments
* Infrastructure-as-Code learning setups

It combines:

* **Proxmox automation**
* **cloud-init**
* **Ansible orchestration**

No manual VM creation is necessary anymore.

---

## Running the Playbook

Store file as:

```
proxmox-k8s-automation.md
```

Run playbook:

```bash
ansible-playbook yourfile.yml
```

---

If you want, I can also:

* add screenshots placeholders
* add troubleshooting section
* convert this into a blog-style tutorial
* add section on creating the Proxmox template
* add command snippets for `qm` template creation
