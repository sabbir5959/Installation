# Vagrant + Libvirt Installation Guide (Ubuntu 24.04)

This guide covers the installation of:

- Vagrant
- Libvirt
- Vagrant Libvirt Plugin

---

# Step 1: Add HashiCorp GPG Key

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

---

# Step 2: Add HashiCorp Repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

---

# Step 3: Update Package Repository

```bash
sudo apt update
```

---

# Step 4: Install Vagrant

```bash
sudo apt install -y vagrant
```

Verify installation:

```bash
vagrant --version
```

---

# Step 5: Install Required Packages for Libvirt

```bash
sudo apt install -y \
libvirt-daemon-system \
libvirt-clients \
bridge-utils \
virt-manager
```

---

# Step 6: Install Required Packages for Vagrant Libvirt Plugin

```bash
sudo apt install -y \
build-essential \
gcc \
g++ \
make \
pkg-config \
ruby-dev \
libvirt-dev \
zlib1g-dev
```

---

# Step 7: Enable and Start Libvirt Service

```bash
sudo systemctl enable --now libvirtd
```

Check service status:

```bash
sudo systemctl status libvirtd
```

---

# Step 8: Add Current User to Libvirt Group

```bash
sudo usermod -aG libvirt $USER
```

> **Note:** Logout/Login (or reboot) after running this command.

---

# Step 9: Verify Libvirt Installation

Check Libvirt version:

```bash
virsh --version
```

List available virtual machines:

```bash
virsh list --all
```

---

# Step 10: Install Vagrant Libvirt Plugin

```bash
vagrant plugin install vagrant-libvirt
```

---

# Step 11: Verify Installed Plugins

```bash
vagrant plugin list
```

Expected output:

```text
vagrant-libvirt (x.x.x)
```

---

# Useful Commands

Check Vagrant version:

```bash
vagrant --version
```

Check installed plugins:

```bash
vagrant plugin list
```

Check Libvirt service:

```bash
systemctl is-active libvirtd
```

List virtual machines:

```bash
virsh list --all
```

List networks:

```bash
virsh net-list --all
```

List storage pools:

```bash
virsh pool-list --all
```