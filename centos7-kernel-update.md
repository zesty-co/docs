# CentOS7 Kernel update

## Step 1: Check Your Current Kernel Version

Run: `uname -msr` the system should return output with your kernel version.

## Step 2: Update yum repos

Run: `sudo yum -y update`

## Step 3: Enable the ELRepo Repository

Run:
```bash
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

### Next, install the ELRepo repository by executing the following command:

```bash
sudo rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```

## Step 4: List Available Kernels:

```bash
yum list available --disablerepo='*' --enablerepo=elrepo-kernel
```

## Step 5: Install New CentOS Kernel Version:

**To install the latest mainline kernel:**

Run: 
```bash
sudo yum --enablerepo=elrepo-kernel install kernel-ml
```

**To install the latest long term support kernel:**

Run:
```bash
sudo yum --enablerepo=elrepo-kernel install kernel-lt
```

## Step 6: Reboot the machine

## Step 7: Edit grub files:

Run:
```bash
sudo vim /etc/default/grub
```

Once the file opens, look for the line that says **GRUB_DEFAULT=X**, and change it to **GRUB_DEFAULT=0** (zero). This line will instruct the boot loader to default to the first kernel on the list, which is the latest.

Save the file, and then type the following command in the terminal to recreate the kernel configuration:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

Now you need to reboot the server then u can run:`uname -mrs` and verify that your kernel is updated.
