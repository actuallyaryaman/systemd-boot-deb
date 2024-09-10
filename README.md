## systemd-boot-deb
replace the default grub bootoader with systemd-boot

```markdown
# Configuring systemd-boot Boot Loader

## Step 1: Create the Boot Config File

Start by creating a boot configuration file `/boot/efi/loader/loader.conf` with the following content:

```bash
default systemd
timeout 1
editor 1
```

## Step 2: Create Directory for Kernel Configuration Files

Create the directory that will hold the configuration files for the installed kernel images:

```bash
mkdir -p /boot/efi/loader/entries
```

## Step 3: Install the Boot Loader

Install the boot loader with the `bootctl` command:

```bash
bootctl install --path=/boot/efi
```

You can check if the installation was successful with the `efibootmgr` command. This command is also useful to change the boot order and to delete old boot entries. The systemd-boot entry should show up as **"Linux Boot Manager"**.

## Step 4: Configure systemd-boot to Recognize Installed Kernel Images

Now systemd-boot is installed, but it does not know about the installed kernel images in the system.

### Step 4.1: Create the Post-install Script

Use the `kernel-install` command to add and remove the kernel to and from the systemd-boot directory structure. Create a `/etc/kernel/postinst.d/zz-update-systemd-boot` file with the following content:

```bash
#!/bin/sh
set -e

/usr/bin/kernel-install add "$1" "$2"

exit 0
```

### Step 4.2: Create the Post-removal Script

Create a `/etc/kernel/postrm.d/zz-update-systemd-boot` file with the following content:

```bash
#!/bin/sh
set -e

/usr/bin/kernel-install remove "$1"

exit 0
```

## Step 5: Manually Add the Kernel

Create the `/boot/efi/<machine-id>` directory if it doesn't exist (you can find the machine-id from the file `/etc/machine-id`) and run `kernel-install` manually to force the initial install. The command will look like this:

```bash
kernel-install add `uname -r` /boot/vmlinuz-`uname -r`
```

Now you should see all installed kernel images in the systemd-boot menu.

## Optional: Keep GRUB as a Fallback

It is not necessary to remove GRUB, as long as it appears after systemd-boot in the output of `efibootmgr`. This also provides a useful fallback in case systemd-boot encounters any problems.
```
