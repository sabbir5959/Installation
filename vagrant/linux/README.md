# Vagrant and Libvirt Installation Guide (Ubuntu)

This guide explains how to install and configure **Vagrant** and **Libvirt** on Ubuntu Linux.

---

# Step 1: Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Install Required Packages

Install KVM, Libvirt, and virtualization tools.

```bash
sudo apt install -y \
qemu-kvm \
libvirt-daemon-system \
libvirt-clients \
bridge-utils \
virt-manager
```

---

# Step 3: Enable and Start Libvirt

```bash
sudo systemctl enable --now libvirtd
```

Verify the service:

```bash
sudo systemctl status libvirtd
```

Or:

```bash
systemctl is-active libvirtd
```

Expected output:

```text
active
```

---

# Step 4: Add Current User to Required Groups

```bash
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```

Apply the changes by logging out and logging back in, or reboot the system.

---

# Step 5: Verify Libvirt Installation

Check Libvirt version:

```bash
virsh --version
```

List available virtual machines:

```bash
virsh list --all
```

If no VMs exist, an empty list is expected.

---

# Step 6: Install Vagrant

## Add HashiCorp GPG Key

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

---

## Add HashiCorp Repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

---

## Install Vagrant

```bash
sudo apt update
sudo apt install -y vagrant
```

Verify installation:

```bash
vagrant --version
```

---

# Step 7: Install Vagrant Libvirt Plugin

```bash
vagrant plugin install vagrant-libvirt
```

Verify installed plugins:

```bash
vagrant plugin list
```

Expected output:

```text
vagrant-libvirt (x.x.x)
```

---

# Step 8: Create the Default Storage Pool (If Required)

Check existing storage pools:

```bash
virsh pool-list --all
```

If the `default` pool does not exist, create it:

```bash
sudo virsh pool-define-as default dir - - - - "/var/lib/libvirt/images"
sudo virsh pool-build default
sudo virsh pool-start default
sudo virsh pool-autostart default
```

Verify:

```bash
virsh pool-list --all
```

Expected output:

```text
 Name      State    Autostart
--------------------------------
 default   active   yes
```

---

# Step 9: Verify Installation

Verify Libvirt:

```bash
virsh list --all
```

Verify Vagrant:

```bash
vagrant --version
```

Verify plugin:

```bash
vagrant plugin list
```

---

# Common Vagrant Commands

Start VMs:

```bash
vagrant up --provider=libvirt
```

Connect to a VM:

```bash
vagrant ssh
```

Check VM status:

```bash
vagrant status
```

Suspend a VM:

```bash
vagrant suspend
```

Resume a VM:

```bash
vagrant resume
```

Shutdown a VM:

```bash
vagrant halt
```

Restart a VM:

```bash
vagrant reload
```

Destroy a VM:

```bash
vagrant destroy -f
```

---

# Useful Libvirt Commands

List virtual machines:

```bash
virsh list --all
```

List storage pools:

```bash
virsh pool-list --all
```

List virtual networks:

```bash
virsh net-list --all
```

Show VM information:

```bash
virsh dominfo <vm-name>
```

List VM volumes:

```bash
virsh vol-list default
```

---

# Troubleshooting

Restart Libvirt:

```bash
sudo systemctl restart libvirtd
```

Check Libvirt logs:

```bash
sudo journalctl -u libvirtd
```

Verify KVM support:

```bash
kvm-ok
```

> If `kvm-ok` is not installed:

```bash
sudo apt install cpu-checker
kvm-ok
```