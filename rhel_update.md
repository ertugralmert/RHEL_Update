<div align="center">
  <p><strong>Designed & Documented by Mertugral</strong></p>
  <p>© 2025 Mertugral. All rights reserved.</p>
  <p><a href="https://github.com/ertugralmert">GitHub</a> | <a href="https://linkedin.com/in/mertertugral">LinkedIn</a></p>
</div>

# RHEL Major Version Upgrade Guide: 7 → 8 → 9

Instructions for upgrading Red Hat Enterprise Linux (RHEL) systems through major versions. The guide covers upgrading from RHEL 7 to RHEL 8, and then from RHEL 8 to RHEL 9, using the Leapp in-place upgrade utility.  
[Documentation_RHEL7_to_RHEL8](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/upgrading_from_rhel_7_to_rhel_8/index#idm140321908504688)  
[Documentation_RHEL8_to_RHEL9](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html-single/upgrading_from_rhel_8_to_rhel_9/index#idm140617907380960)

## Introduction

Red Hat Enterprise Linux provides an in-place upgrade path between major versions using the Leapp utility. This allows you to upgrade your operating system while preserving installed applications, configurations, and data. This document covers the complete process for upgrading from RHEL 7 to RHEL 8, and then from RHEL 8 to RHEL 9.

> **Important Note**: It is not possible to upgrade directly from RHEL 7 to RHEL 9. You must first upgrade to RHEL 8 and then to RHEL 9.

## Preliminary Information

### Minor Version Updates vs. Major Upgrades

Before proceeding with major version upgrades, it's important to understand the difference between minor version updates and major version upgrades:

**Minor Version Updates:**

- RHEL 7.x → 7.9 update is done using the standard `yum update` process without Leapp.
- RHEL 8.8 → 8.10 update is done using the standard `dnf update` process without Leapp.

**Major Version Upgrades:**

- RHEL 7.9 → RHEL 8.x (requires Leapp)
- RHEL 8.x → RHEL 9.x (requires Leapp)

Sample commands for minor version updates:

```Bash
# For RHEL 7.x to 7.9 update:
subscription-manager status                  # Verify subscription
subscription-manager release --set=7.9       # Optional: Lock to 7.9
# OR
subscription-manager release --unset         # To get latest 7.x version
yum clean all
yum update -y
reboot
cat /etc/redhat-release                      # Verify: should show RHEL 7.9

# For RHEL 8.8 to 8.10 update:
cat /etc/redhat-release                      # Verify current version
subscription-manager release --set=8.10      # Lock to 8.10
dnf clean all
dnf update -y
reboot
cat /etc/redhat-release                      # Verify: should show RHEL 8.10
```

### Key Migration Terminology

While the following migration terms are commonly used in the software industry, these definitions are specific to Red Hat Enterprise Linux (RHEL).

**Update:** Sometimes called a software patch, an update is an addition to the current version of the application, operating system, or software that you are running. A software update addresses any issues or bugs to provide a better experience of working with the technology. In RHEL, an update relates to a minor release, for example, updating from RHEL 8.1 to 8.2.

**Upgrade:** An upgrade is when you replace the application, operating system, or software that you are currently running with a newer version. Typically, you first back up your data according to instructions from Red Hat. When you upgrade RHEL, you have two options:

- **In-place upgrade:** During an in-place upgrade, you replace the earlier version with the new version without removing the earlier version first. The installed applications and utilities, along with the configurations and preferences, are incorporated into the new version.
- **Clean install:** A clean install removes all traces of the previously installed operating system, system data, configurations, and applications and installs the latest version of the operating system. A clean install is ideal if you do not need any of the previous data or applications on your systems or if you are developing a new project that does not rely on prior builds.

**Operating system conversion:** A conversion is when you convert your operating system from a different Linux distribution to Red Hat Enterprise Linux. Typically, you first back up your data according to instructions from Red Hat.

**Migration:** Typically, a migration indicates a change of platform: software or hardware. Moving from Windows to Linux is a migration. Moving a user from one laptop to another or a company from one server to another is a migration. However, most migrations also involve upgrades, and sometimes the terms are used interchangeably.

- Migration to RHEL: Conversion of an existing operating system to RHEL
- Migration across RHEL: Upgrade from one version of RHEL to another

## Part 1: RHEL 7 to RHEL 8 Upgrade

### Planning and Prerequisites

#### Supported Upgrade Paths

The in-place upgrade from RHEL 7 to RHEL 8 is supported for:

| **System Configuration** | **Source OS Version** | **Target OS Version** |
| ------------------------ | --------------------- | --------------------- |
| RHEL                     | RHEL 7.9              | RHEL 8.8              |
| RHEL                     | RHEL 7.9              | RHEL 8.10 (default)   |
| RHEL with SAP HANA       | RHEL 7.9              | RHEL 8.8 (default)    |
| RHEL with SAP HANA       | RHEL 7.9              | RHEL 8.10             |

#### System Requirements

Before proceeding with the upgrade, ensure your system meets the following requirements:

1. **Hardware Requirements**:
   - Check system architecture:

```Bash
uname -m
```

   - Supported architectures: 64-bit Intel/AMD (x86_64), IBM POWER 8 (little endian), 64-bit IBM Z
   - Minimum requirements for RHEL 8: 2GB RAM (4GB recommended), 20GB disk space
1. **Operating System Version**:
   - Current RHEL version must be 7.9:

```Bash
cat /etc/redhat-release
```

   - If your system is on an earlier version, update to 7.9 first
1. **Subscription Status**:
   - System must be registered with Red Hat Subscription Manager:

```Bash
subscription-manager status
```

1. **Known Limitations**:
   - Systems with encryption (full disk, partition, or file-system) are not supported
   - Network based multipath and network storage using Ethernet or Infiniband not supported
   - Systems with Ansible products (including Ansible Tower) are not supported

### Preparation

#### 1. System Health Check

1. Verify current RHEL version:

```Bash
cat /etc/redhat-release
```

1. Check hardware resources:

```Bash
# Check memory
free -h

# Check disk space (need at least 4GB in /var/lib/leapp)
df -h

# Check CPU
nproc
```

1. Document your current system state:

```Bash
# Create a list of installed packages
rpm -qa > /root/rhel7-packages-before-upgrade.txt

# List running services
systemctl list-units --type=service --state=running > /root/active-services-before-upgrade.txt

# Check current disk usage
df -h > /root/disk-usage-before-upgrade.txt
```

#### 2. Subscription Configuration

1. Configure subscription:

```Bash
# For systems with SCA enabled, verify content access mode
subscription-manager status

# For systems without SCA, verify product subscription
subscription-manager list --installed
```

1. Set the system to access the latest RHEL 7 content:

```Bash
subscription-manager release --unset
```

#### 3. Configure Repositories

1. Enable required repositories:

```Bash
# Enable Base repository
subscription-manager repos --enable rhel-7-server-rpms

# Enable Extras repository (for Leapp)
subscription-manager repos --enable rhel-7-server-extras-rpms
```

1. Clear any version locks:

```Bash
yum versionlock clear
```

#### 4. Install Leapp and Update System

1. Install the Leapp upgrade utility:

```Bash
yum install leapp-upgrade
```

1. Update all packages to the latest RHEL 7 version and reboot:

```Bash
yum update -y
reboot
```

#### 5. Prepare Services and Configuration

1. Disable antivirus software temporarily (if installed)
2. Disable configuration management systems:

```Bash
# For Puppet
systemctl stop puppet
systemctl disable puppet

# For other systems, stop the respective services
```

1. Prepare file systems (if needed):

```Bash
# Back up /etc/fstab
cp /etc/fstab /etc/fstab.backup

# Edit /etc/fstab and comment out non-system mount points
vi /etc/fstab
```

#### 6. Create a Full System Backup

1. Using Relax-and-Recover (ReaR) - Recommended:

```Bash
# Install ReaR
yum install rear -y

# Create ReaR configuration
mkdir -p /mnt/backup
cat > /etc/rear/local.conf << EOF
OUTPUT=ISO
OUTPUT_URL=file:///mnt/backup
BACKUP=NETFS
BACKUP_URL=file:///mnt/backup
BACKUP_PROG=tar
EOF

# Create rescue image
rear -v mkrescue
```

1. Alternative: LVM Snapshot (if system is using LVM):

```Bash
# Check LVM configuration
vgs
lvs

# Create snapshot of root partition
lvcreate -L 10G -s -n root_snapshot /dev/VOLUME_GROUP/root
```

### Pre-upgrade Assessment

1. Run the pre-upgrade assessment:

```Bash
leapp preupgrade --target 8.10
```

1. Review the pre-upgrade report:

```Bash
less /var/log/leapp/leapp-report.txt
```

1. Address any inhibitors found in the report.
2. If required, provide answers to questions in the answer file:

```Bash
vi /var/log/leapp/answerfile
```

   Alternatively, use the command line:

```Bash
leapp answer --section <question_section>.<field_name>=<answer>
```

1. Re-run the pre-upgrade assessment to verify issues are resolved:

```Bash
leapp preupgrade --target 8.10
```

### Performing the Upgrade

1. Start the upgrade process:

```Bash
leapp upgrade --target 8.10
```

1. After the initial phase completes, reboot the system:

```Bash
reboot
```

1. The system will boot into a RHEL 8-based initramfs, upgrade all packages, and automatically reboot to the RHEL 8 system.

### Post-upgrade Tasks

1. Verify upgrade success:

```Bash
# Check OS version
cat /etc/redhat-release

# Check kernel version
uname -r

# Check subscription status
subscription-manager list --installed
```

1. Remove old RHEL 7 packages:

```Bash
# Identify old kernel versions
cd /lib/modules && ls -d *.el7*

# Remove weak modules from old kernels
[ -x /usr/sbin/weak-modules ] && /usr/sbin/weak-modules --remove-kernel <version>

# Remove old kernels from boot loader
/bin/kernel-install remove <version> /lib/modules/<version>/vmlinuz

# Find remaining RHEL 7 packages
rpm -qa | grep -e '\.el7' | grep -vE '^(gpg-pubkey|libmodulemd|katello-ca-consumer)' | sort

# Remove remaining RHEL 7 packages
yum remove $(rpm -qa | grep \.el7 | grep -vE 'gpg-pubkey|libmodulemd|katello-ca-consumer') -y

# Remove Leapp dependency packages
yum remove leapp-deps-el8 leapp-repository-deps-el8 -y

# Remove remaining empty directories
rm -r /lib/modules/*el7*
```

1. Update the rescue kernel:

```Bash
# Remove existing rescue kernel
rm /boot/vmlinuz-*rescue* /boot/initramfs-*rescue*

# Reinstall rescue kernel
/usr/lib/kernel/install.d/51-dracut-rescue.install add "$(uname -r)" /boot "/boot/vmlinuz-$(uname -r)"
```

1. For IBM Z architecture, update the zipl boot loader:

```Bash
zipl
```

1. Set SELinux to enforcing mode:

```Bash
# Check for denials
ausearch -m AVC,USER_AVC -ts boot

# Edit SELinux config
vi /etc/selinux/config
# Set SELINUX=enforcing

# Reboot
reboot

# Verify SELinux status
getenforce
```

1. Configure RHEL 8 repositories:

```Bash
subscription-manager repos --disable=*
subscription-manager repos --enable=rhel-8-for-$(uname -m)-baseos-rpms
subscription-manager repos --enable=rhel-8-for-$(uname -m)-appstream-rpms
```

1. Update all packages:

```Bash
dnf update -y
```

1. Create new system backup:

```Bash
rear -v mkrescue
```

## Part 2: RHEL 8 to RHEL 9 Upgrade

### Planning and Prerequisites

#### Supported Upgrade Paths

The in-place upgrade from RHEL 8 to RHEL 9 is supported for:

| **System Configuration** | **Source OS Version** | **Target OS Version** |
| ------------------------ | --------------------- | --------------------- |
| RHEL                     | RHEL 8.8              | RHEL 9.2              |
| RHEL                     | RHEL 8.10             | RHEL 9.4              |
| RHEL                     | RHEL 8.10             | RHEL 9.5              |
| RHEL with SAP HANA       | RHEL 8.8              | RHEL 9.2              |
| RHEL with SAP HANA       | RHEL 8.10             | RHEL 9.4              |

#### System Requirements

Before proceeding with the upgrade, ensure your system meets the following requirements:

1. **Hardware Requirements**:
   - Check system architecture:

```Bash
uname -m
```

   - Supported architectures: 64-bit Intel, AMD, and ARM; IBM POWER (little endian); 64-bit IBM Z
1. **Operating System Version**:
   - Current RHEL version must be 8.8 or 8.10:

```Bash
cat /etc/redhat-release
```

1. **Subscription Status**:
   - System must be registered with Red Hat Subscription Manager:

```Bash
subscription-manager status
```

1. **Known Limitations**:
   - Packages with RSA/SHA-1 signatures are not supported (SHA-1 is deprecated in RHEL 9)
   - Systems with encryption (full disk, partition, or file-system) are not supported
   - Network based multipath and network storage using Ethernet or Infiniband not supported
   - Systems with Ansible products (including Ansible Tower) are not supported

### Preparation

#### 1. System Health Check

1. Verify current RHEL version:

```Bash
cat /etc/redhat-release
```

1. Check hardware resources:

```Bash
# Check memory
free -h

# Check disk space (need at least 4GB in /var/lib/leapp)
df -h

# Check CPU
nproc
```

1. Document your current system state:

```Bash
# Create a list of installed packages
rpm -qa > /root/rhel8-packages-before-upgrade.txt

# List running services
systemctl list-units --type=service --state=running > /root/active-services-before-upgrade.txt

# Check current disk usage
df -h > /root/disk-usage-before-upgrade.txt
```

#### 2. Subscription Configuration

1. Set the system to the desired source OS version:

```Bash
# For systems using RHSM
subscription-manager release --set=8.10

# For systems using RHUI on public cloud
rhui-set-release --set=8.10
```

1. Clear any version locks:

```Bash
dnf versionlock clear
```

#### 3. Prepare Special Components

1. Convert NSS databases from DBM to SQLite (required for RHEL 9):

```Bash
# Set NSS_DEFAULT_DB_TYPE environment variable
export NSS_DEFAULT_DB_TYPE=sql

# Convert NSS databases
certutil -K -X -d /etc/pki/nssdb
```

1. If using Virtual Data Optimizer (VDO), convert to LVM management:

```Bash
# Check VDO status
vdo status

# Convert VDO devices to LVM (if needed)
# This is a complex process specific to your VDO configuration
```

1. Migrate from network-scripts to NetworkManager:

```Bash
# Check if legacy network-scripts package is installed
rpm -q network-scripts

# Verify NetworkManager is active
systemctl status NetworkManager

# Migrate custom network scripts to NetworkManager
# This process depends on your specific network configuration
```

#### 4. Configure Repositories

1. Enable required repositories:

```Bash
# Enable Base and AppStream repositoriessubscription-manager repos --enable rhel-8-for-$(uname -m)-baseos-rpmssubscription-manager repos --enable rhel-8-for-$(uname -m)-appstream-rpms
```

#### 5. Install Leapp and Update System

1. Install the Leapp upgrade utility:

```Bash
dnf install leapp-upgrade -y
```

1. Update all packages to the latest versions:

```Bash
dnf update -y
reboot
```

#### 6. Prepare Services and Configuration

1. Disable antivirus software temporarily (if installed)
2. Disable configuration management systems:

```Bash
# For Puppet
systemctl stop puppet
systemctl disable puppet

# For other systems, stop the respective services
```

1. Prepare file systems (if needed):

```Bash
# Back up /etc/fstab
cp /etc/fstab /etc/fstab.backup

# Edit /etc/fstab and comment out non-system mount points
vi /etc/fstab
```

#### 7. Create a Full System Backup

1. Using Relax-and-Recover (ReaR) - Recommended:

```Bash
# Install or update ReaR
dnf install rear -y

# Create ReaR configuration (if not already done)
mkdir -p /mnt/backup
cat > /etc/rear/local.conf << EOF
OUTPUT=ISO
OUTPUT_URL=file:///mnt/backup
BACKUP=NETFS
BACKUP_URL=file:///mnt/backup
BACKUP_PROG=tar
EOF

# Create rescue image
rear -v mkrescue
```

1. Alternative: LVM Snapshot (if system is using LVM):

```Bash
# Create snapshot of root partition
lvcreate -L 10G -s -n root_snapshot /dev/VOLUME_GROUP/root
```

### Pre-upgrade Assessment

1. Run the pre-upgrade assessment (note the SELinux context requirement):

```Bash
# If running as root
leapp preupgrade --target 9.4

# If using sudo
sudo -r unconfined_r -t unconfined_t leapp preupgrade --target 9.4
```

1. Review the pre-upgrade report:

```Bash
less /var/log/leapp/leapp-report.txt
```

1. Address any inhibitors found in the report, particularly:
   - SHA-1 signed packages
   - VDO devices not managed by LVM
   - Legacy network configuration
   - Other issues specific to your system
1. If required, provide answers to questions in the answer file:

```Bash
vi /var/log/leapp/answerfile
```

   Alternatively, use the command line:

```Bash
leapp answer --section <question_section>.<field_name>=<answer>
```

   Example for VDO question:

```Bash
leapp answer --section check_vdo.confirm=True
```

1. Re-run the pre-upgrade assessment to verify issues are resolved:

```Bash
leapp preupgrade --target 9.4
```

### Performing the Upgrade

1. Start the upgrade process (note the SELinux context requirement):

```Bash
# If running as root
leapp upgrade --target 9.4

# If using sudo
sudo -r unconfined_r -t unconfined_t leapp upgrade --target 9.4

# Alternatively, with automatic reboot
leapp upgrade --reboot --target 9.4
```

1. If not using the `--reboot` option, manually reboot the system:

```Bash
reboot
```

1. The system will boot into a RHEL 9-based initramfs, upgrade all packages, and automatically reboot to the RHEL 9 system.

### Post-upgrade Tasks

1. Verify upgrade success:

```Bash
# Check OS version
cat /etc/redhat-release

# Check kernel version
uname -r

# Check subscription status
subscription-manager list --installed
subscription-manager release
```

1. Remove old RHEL 8 packages:

```Bash
# Find remaining RHEL 8 packages
rpm -qa | grep -e '\.el[78]' | grep -vE '^(gpg-pubkey|libmodulemd|katello-ca-consumer)' | sort

# Remove remaining RHEL 8 packages
dnf remove $(rpm -qa | grep \.el[78] | grep -vE 'gpg-pubkey|libmodulemd|katello-ca-consumer')

# Remove Leapp dependency packages
dnf remove leapp-deps-el9 leapp-repository-deps-el9
```

1. Update the rescue kernel:

```Bash
# Remove existing rescue kernel
rm /boot/vmlinuz-*rescue* /boot/initramfs-*rescue*

# Reinstall rescue kernel
/usr/lib/kernel/install.d/51-dracut-rescue.install add "$(uname -r)" /boot "/boot/vmlinuz-$(uname -r)"
```

1. For IBM Z architecture, update the zipl boot loader:

```Bash
zipl
```

1. Set SELinux to enforcing mode:

```Bash
# Check for denials
ausearch -m AVC,USER_AVC -ts boot

# Edit SELinux config
vi /etc/selinux/config
# Set SELINUX=enforcing

# Reboot
reboot

# Verify SELinux status
getenforce
```

1. Verify and update security policies:

```Bash
# Check current cryptographic policy
update-crypto-policies --show

# If using SHA-1, you may need to enable it explicitly
# update-crypto-policies --set DEFAULT:SHA1

# Enable logrotate timer (not active by default after upgrade)
systemctl enable --now logrotate.timer
```

1. Assess security compliance (optional):

```Bash
# Install the security compliance tools
dnf install scap-security-guide -y

# Evaluate system against a security profile (e.g., PCI-DSS)
oscap xccdf eval --profile pci-dss --report report.html /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```

1. Create new system backup:

```Bash
# Update ReaR if needed
dnf install rear -y

# Create new backup image
rear -v mkrescue
```

## Troubleshooting

### Common Issues with RHEL 7 to RHEL 8 Upgrades

1. **Subscription Issues**:

```Bash
# Check subscription status
subscription-manager status

# Re-register if needed
subscription-manager unregister
subscription-manager register --username=<username> --password=<password>
```

1. **Repository Issues**:

```Bash
# Verify repositories
subscription-manager repos --list-enabled

# Clean and reconfigure
subscription-manager repos --disable=*
subscription-manager repos --enable rhel-8-for-$(uname -m)-baseos-rpms
subscription-manager repos --enable rhel-8-for-$(uname -m)-appstream-rpms
```

1. **Package Conflicts**:

```Bash
# Run pre-upgrade with debug
leapp preupgrade --target 8.10 --debug
```

1. **SELinux Problems**:

```Bash
# Check SELinux status
getenforce

# Set to permissive temporarily
setenforce 0
```

### Common Issues with RHEL 8 to RHEL 9 Upgrades

1. **SHA-1 Signed Packages**:

```Bash
# Identify SHA-1 signed packages
rpm -qa --qf '%{NAME}-%{VERSION}-%{RELEASE} %{SIGPGP:pgpsig}\n' | grep 'Key ID' | grep 'SHA1'

# Remove or update identified packages
dnf remove <package_name>
```

1. **VDO Issues**:

```Bash
# Check VDO status
vdo status

# Verify VDO LVM conversion
lvs -o vdo_pool
```

1. **NSS Database Problems**:

```Bash
# Find Berkeley DB format databases
find / -name "*.db" -type f | xargs file | grep "Berkeley DB"

# Convert remaining databases
export NSS_DEFAULT_DB_TYPE=sql
certutil -K -X -d <database_directory>
```

1. **Network Configuration Issues**:

```Bash
# Check NetworkManager status
systemctl status NetworkManager

# Verify network interfaces
ip addr
```

1. **Disk Space Issues**:

```Bash
# Check space in /var/lib/leapp
df -h /var/lib/leapp

# If needed, move to a partition with more space
mkdir -p /mnt/leapp
mv /var/lib/leapp/* /mnt/leapp/
rmdir /var/lib/leapp
ln -s /mnt/leapp /var/lib/leapp
```

## Special Considerations

### High Availability (HA) Clusters

1. Check cluster status before upgrading:

```Bash
pcs status
```

1. Move resources to another node before upgrading each node:

```Bash
pcs resource move <resource_name> <target_node>
```

1. Follow Red Hat's recommendations for upgrading HA clusters.

### Database Servers

1. Backup databases before upgrading:

```Bash
# PostgreSQL
pg_dump -U postgres -F c -f /backup/postgres_dump.custom all

# MySQL/MariaDB
mysqldump --all-databases > /backup/mysql_dump.sql
```

1. Verify database compatibility with the target RHEL version.

### Web Servers

1. Backup web server configurations:

```Bash
# Apache
cp -rp /etc/httpd /etc/httpd.bak

# Nginx
cp -rp /etc/nginx /etc/nginx.bak
```

1. Validate configurations:

```Bash
# Apache
httpd -t

# Nginx
nginx -t
```

## Best Practices

1. **Planning and Timing**:
   - Schedule upgrades during maintenance windows
   - Ensure sufficient time for the complete process
   - Check system load before starting:

```Bash
uptimetop
```

1. **Backup Strategy**:
   - Always create full system backups
   - Additionally back up critical data directories:

```Bash
tar -czvf /mnt/backup/etc_backup.tar.gz /etctar -czvf /mnt/backup/home_backup.tar.gz /home
```

   - Test backups for recoverability before upgrading
---
# RHEL Major Version Upgrade – Issues, Commands, and Solutions

This document outlines the issues we faced during the in‐place upgrade (RHEL 7 → 8 → 9), along with the commands and solutions used to resolve them.

## 1. Issue: Kernel Driver pata_acpi Is Loaded in RHEL 7 but Removed in RHEL 8

### Problem
Leapp's rpm_scanner reported:

> "Detected loaded kernel drivers which have been removed in RHEL 8. Upgrade cannot proceed."

This is due to the pata_acpi module still being loaded on the system even though it is not supported in RHEL 8.

### Commands and Solutions

**Remove the Module (Temporary)**
```bash
modprobe -r pata_acpi
```
Note: This command may not force removal if the module is in use.

**Blacklist the Module (Permanent)**
```bash
echo "blacklist pata_acpi" >> /etc/modprobe.d/blacklist.conf
```

**Update initramfs**
```bash
dracut -f
```

**Reboot the System**
```bash
reboot
```

**Verify Module Removal**
```bash
lsmod | grep pata_acpi
```
No output means the module is no longer loaded.

## 2. Issue: SSH Root Login Warning

### Problem
Leapp reports warnings related to remote login using the root account due to the default RHEL 8 SSH configuration (PermitRootLogin prohibit-password).

### Commands and Solutions
If you require root login with a password in RHEL 8:

**Edit the SSH Configuration**
```bash
vi /etc/ssh/sshd_config
```

Change or add the line:
```
PermitRootLogin yes
```

**Restart the SSH Service**
```bash
systemctl restart sshd
```

Alternatively, if you do not require root login via password, you may simply answer the Leapp question in the answer file (or via the leapp answer command) to acknowledge that this risk is accepted.

## 3. Issue: Malformed RPM Metadata Causing "ValueError: too many values to unpack"

### Problem
Leapp's rpmscanner fails when parsing an RPM package that has extra pipe (|) characters in its metadata (e.g., in the Packager or Release fields).
Error Message:

> "ValueError: too many values to unpack"

This was observed in the gpg-pubkey packages.

### Commands and Solutions

**Identify the Problematic Package(s)**
```bash
for pkg in $(rpm -qa); do
  out=$(rpm -q --qf '%{NAME}|%{VERSION}|%{RELEASE}|%{EPOCH}|%{PACKAGER}|%{ARCH}|%{SIGPGP:pgpsig}\n' "$pkg")
  fields=$(echo "$out" | awk -F'|' '{print NF}')
  if [ "$fields" -ne 7 ]; then
    echo "Problematic package: $pkg"
    echo "Metadata: $out"
    echo "----"
  fi
done
```

This script checks each installed package to ensure the metadata splits into exactly 7 fields.

**Remove the Problematic Package**
For example, if the output shows:
```
gpg-pubkey-477dea46-5e67f953
```

Remove it with:
```bash
rpm -e gpg-pubkey-477dea46-5e67f953
```

Or using yum:
```bash
yum remove gpg-pubkey-477dea46-5e67f953
```

Note: If yum does not remove it because it is not treated as a "normal" package, use rpm directly.

## 4. Issue: SHA-1 Signed Packages

### Problem
RHEL 9 does not support SHA-1 signed packages. The Leapp preupgrade/upgrade process reported SHA-1 issues for packages such as dscsys1 and dsesame.

### Commands and Solutions

**Verify if SHA-1 Signed Packages Exist**
```bash
rpm -qa --qf '%{NAME}-%{VERSION}-%{RELEASE} %{SIGPGP:pgpsig}\n' | grep 'SHA1'
```

If output shows packages with SHA1, they must be addressed.

**Solution Options:**

1. **Remove the Packages:**
```bash
yum remove dscsys1 dsesame
```

2. **Replace with Updated Versions:**
Contact the vendor or check if updated (SHA-256 signed) versions are available and install those.

3. **Bypassing SHA-1 Check:**
There is no supported method to simply bypass the SHA-1 check via leapp answer. The proper resolution is to remove or update these packages.

## 5. Issue: Read-Only Filesystem Error in /var/lib/leapp

### Problem
Leapp reports an error such as:

> "Failed to create directory ... read-only file system"

This indicates that Leapp's scratch directory is mounted as read-only.

### Commands and Solutions

**Check Mount Status**
```bash
mount | grep /var/lib/leapp
df -h /var/lib/leapp
```
Look for ro (read-only) in the output.

**Remount as Read-Write**
```bash
mount -o remount,rw /var/lib/leapp
```
Or adjust the /etc/fstab entry accordingly.

**Ensure Sufficient Disk Space**
```bash
df -h /var/lib/leapp
```
At least 4-5 GB should be free.

**Check and Restore SELinux Contexts (if applicable)**
```bash
restorecon -Rv /var/lib/leapp
```

## 6. Issue: OverlayFS/XFS ftype=0 Issue – Using LEAPP_OVL_IMG_FS_EXT4

### Problem
On some systems, OverlayFS used by Leapp conflicts with XFS filesystems (especially if XFS was created with ftype=0). This can result in "read-only" errors during upgrade.

### Commands and Solutions

**Set Environment Variable to Force ext4 Overlay**
```bash
export LEAPP_OVL_IMG_FS_EXT4=1
```
This forces Leapp to use an ext4-formatted overlay image instead of one based on XFS.

**Run the Preupgrade Again**
```bash
leapp preupgrade --target 8.10
```
Then proceed with the upgrade if the error is resolved.

## 7. Issue: "Excluded by module filtering" Error

### Problem
When installing Leapp-related packages (or other packages), DNF may report "excluded by module filtering" because a module stream is filtering out the package.

### Commands and Solutions

**List Active Modules**
```bash
dnf module list
```

**Reset/Disable Conflicting Modules (Example for Python39)**
```bash
dnf module reset python39
dnf module disable python39
```
Repeat for any module that might be causing the filtering issue.

**Clean and Retry Package Installation**
```bash
dnf clean all
dnf install leapp-upgrade-el8toel9
```

## 8. Issue: DistributionNotFound Error ("leapp==1.0.0" not found)

### Problem
Leapp reports:

> "DistributionNotFound: The 'leapp==1.0.0' distribution was not found..."

This indicates that an expected version of Leapp is missing or that old Python site-packages remain.

### Commands and Solutions

**Remove All Leapp-Related Packages**
```bash
rpm -qa | grep leapp
dnf remove leapp-upgrade-el8toel9 leapp-deps-el9 leapp-repository-deps-el9 python3-leapp
```

**Clean Python Site-Packages (if needed)**
```bash
find /usr/lib/python3* /usr/local/lib/python3* -type d -name "leapp*"
rm -rf /usr/lib/python3*/site-packages/leapp*
rm -rf /usr/local/lib/python3*/site-packages/leapp*
```

**Clean DNF Cache**
```bash
dnf clean all
```

**Reinstall Leapp**
```bash
dnf install leapp-upgrade-el8toel9
```

**Test the Preupgrade**
```bash
leapp preupgrade --target 9.4
```

## 9. Post-Upgrade Checks

After completing the upgrade (whether RHEL 7 → 8 or RHEL 8 → 9), perform the following verifications:

**Verify OS Version and Kernel**
```bash
cat /etc/redhat-release
uname -r
```

**Check Subscription Status**
```bash
subscription-manager list --installed
```

**Clean Up Old Packages**

1. Identify and Remove Old Kernel/El7 Packages:
```bash
rpm -qa | grep '\.el7'
yum remove <old_el7_packages> -y
```

2. Remove Weak Modules (if any):
```bash
[ -x /usr/sbin/weak-modules ] && /usr/sbin/weak-modules --remove-kernel <old_kernel_version>
```

3. Remove Old Kernels from Boot Loader (Optional):
```bash
/bin/kernel-install remove <old_kernel_version> /lib/modules/<old_kernel_version>/vmlinuz
```

**Update the Rescue Kernel**
```bash
rm /boot/vmlinuz-*rescue* /boot/initramfs-*rescue*
/usr/lib/kernel/install.d/51-dracut-rescue.install add "$(uname -r)" /boot "/boot/vmlinuz-$(uname -r)"
```

**Verify SELinux Mode**
```bash
getenforce
```
If it's not "Enforcing", set it in /etc/selinux/config and reboot.

**Check Critical Services and Logs**

1. List running services:
```bash
systemctl list-units --type=service --state=running
```

2. View error logs:
```bash
journalctl -p 3 -xb
```

**Final Package Update**
```bash
dnf clean all
dnf update -y
```

## Summary

During the upgrade process, we encountered and resolved the following issues:

1. **Kernel Driver pata_acpi**
   - Removed via modprobe -r pata_acpi, blacklisted it, updated initramfs, and rebooted.

2. **SSH Root Login Warning**
   - Modified /etc/ssh/sshd_config to PermitRootLogin yes and restarted SSH if needed.

3. **Malformed RPM Metadata**
   - Identified problematic packages (e.g., gpg-pubkey with extra delimiters) and removed them using rpm -e.

4. **SHA-1 Signed Packages**
   - Detected SHA-1 signed packages (e.g., dscsys1, dsesame) and removed or planned to update them.

5. **Read-Only Filesystem Error**
   - Ensured /var/lib/leapp was mounted read-write, checked disk space, and restored SELinux contexts.

6. **OverlayFS Issue**
   - Set export LEAPP_OVL_IMG_FS_EXT4=1 to force overlay image creation as ext4.

7. **Module Filtering Issue**
   - Reset/disabled conflicting modules via dnf module reset/disable.

8. **DistributionNotFound for Leapp**
   - Removed old Leapp packages and cleaned Python site-packages, then reinstalled Leapp via DNF.

9. **Post-Upgrade Validations**
   - Verified OS version, kernel, subscription, cleaned old packages, updated rescue kernel, and ensured SELinux is enforcing.

---
<div align="center">
  <p><strong>Designed & Documented by Mertugral</strong></p>
  <p>© 2025 Mertugral. All rights reserved.</p>
  <p><a href="https://github.com/ertugralmert">GitHub</a> | <a href="https://linkedin.com/in/mertertugral">LinkedIn</a></p>
</div>
 

