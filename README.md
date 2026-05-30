Repository Name:
92-veeam-backup-kernel-module-repair

README.md

# 92. Veeam Backup Kernel Module Repair

## Objective
Troubleshoot and restore failed Veeam backups on dev-1 by identifying and correcting missing Veeam kernel modules required by the active kernel.

## Environment
- RHEL 9
- Veeam Agent for Linux
- dev-1
- dev-2

## Tasks Performed

1. Reviewed Veeam backup session history using:
   sudo veeamconfig session list

2. Identified repeated backup failures and confirmed the active kernel version:
   uname -r

3. Verified required Veeam modules were missing from the running kernel:
   lsmod | grep -E 'veeam|bdev'

4. Compared module files on a working server (dev-2).

5. Copied the required Veeam modules from dev-2 to dev-1:
   - veeamblksnap.ko.xz
   - bdevfilter.ko.xz

6. Moved the modules into:
   /lib/modules/5.14.0-700.el9.x86_64/extra/

7. Rebuilt kernel module dependencies:
   sudo depmod -a

8. Loaded the required modules:
   sudo modprobe bdevfilter
   sudo modprobe veeamblksnap

9. Verified modules successfully loaded:
   lsmod | grep -E 'veeam|bdev'

10. Started and validated a new Veeam backup job.

11. Verified backup completion:
    sudo veeamconfig session list

12. Reviewed Veeam service logs:
    sudo journalctl -xeu veeamservice --no-pager | tail -50

## Validation

Verified:
- veeamblksnap module loaded
- bdevfilter module loaded
- Veeam service running normally
- Backup job completed successfully
- No critical Veeam service errors present

## Commands Used

uname -r

sudo veeamconfig session list

lsmod | grep -E 'veeam|bdev'

scp /lib/modules/5.14.0-700.el9.x86_64/extra/veeamblksnap.ko.xz root@dev-1:/tmp/

scp /lib/modules/5.14.0-700.el9.x86_64/extra/bdevfilter.ko.xz root@dev-1:/tmp/

sudo mv /tmp/veeamblksnap.ko.xz /lib/modules/5.14.0-700.el9.x86_64/extra/

sudo mv /tmp/bdevfilter.ko.xz /lib/modules/5.14.0-700.el9.x86_64/extra/

sudo depmod -a

sudo modprobe bdevfilter

sudo modprobe veeamblksnap

sudo journalctl -xeu veeamservice --no-pager | tail -50

## Result

Successfully restored Veeam backup functionality on dev-1 by installing the correct kernel modules for the active kernel, validating module loading, and confirming successful backup execution.
