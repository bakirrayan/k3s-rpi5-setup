# 01 — OS Preparation

Prepare Ubuntu Server on the Raspberry Pi 5 before installing anything Kubernetes-related.

---

## Requirements

- Raspberry Pi 5 with 8GB RAM
- NVMe SSD (500GB recommended) as the boot drive
- Ubuntu Server 24.04 LTS 64-bit — flashed via Raspberry Pi Imager
- SSH access or direct terminal

---

## Step 1 — Update the system

\`\`\`bash
sudo apt update && sudo apt upgrade -y
sudo reboot
\`\`\`

---

## Step 2 — Enable cgroups

cgroups are required for Kubernetes to manage container resources. On Raspberry Pi they are not enabled by default.

\`\`\`bash
sudo nano /boot/firmware/cmdline.txt
\`\`\`

!!! warning
    Add the following to the **end of the existing single line**. Do NOT create a new line.

\`\`\`
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
\`\`\`

Reboot to apply:

\`\`\`bash
sudo reboot
\`\`\`

---

## Step 3 — Disable swap

Kubernetes requires swap to be disabled.

\`\`\`bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
\`\`\`

---

## Verify

After rebooting, confirm cgroups are active:

\`\`\`bash
cat /proc/cgroups | grep memory
\`\`\`

Expected output includes a `1` in the enabled column for memory.

---

## Step 4 — Set up SSH key authentication

Password authentication is a security risk. Set up SSH keys so you never need a password to SSH in.

On your laptop, generate a key pair if you don't have one:

\`\`\`bash
ssh-keygen -t ed25519 -C "rpi5"
\`\`\`

Copy the public key to the RPi:

\`\`\`bash
ssh-copy-id your_username@your_rpi_ip
\`\`\`

Test it works:

\`\`\`bash
ssh your_username@your_rpi_ip
\`\`\`

Then disable password authentication entirely:

\`\`\`bash
sudo nano /etc/ssh/sshd_config
\`\`\`

Set:

\`\`\`
PasswordAuthentication no
\`\`\`

Restart SSH:

\`\`\`bash
sudo systemctl restart sshd
\`\`\`

!!! warning "Recovery"
    If you ever lose access and need to reset your password, boot from a USB stick, mount the SSD and edit `/etc/shadow` directly, or use `chroot` if architectures match. For ARM64 RPi SSDs connected to an x86 laptop, install `qemu-user-static` first.

---

!!! success "Ready"
    Proceed to [02 — K3s Installation](02-k3s-installation.md)