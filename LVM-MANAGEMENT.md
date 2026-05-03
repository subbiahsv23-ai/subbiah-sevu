# LVM Management Quick Reference Guide

## 📋 Overview

Complete LVM (Logical Volume Management) automation framework for:
- Physical Volume (PV) creation and management
- Volume Group (VG) creation and extension
- Logical Volume (LV) creation, resizing, and mounting
- Snapshot creation and backup automation
- Health monitoring and alerting
- Performance tracking and reporting

---

## 🚀 Quick Start

### 1. Configure Your Storage

Edit `ansible/vars/lvm-config.yml`:

```yaml
lvm_disks:
  - /dev/sdb
  - /dev/sdc

lvm_volume_groups:
  - name: vg_data
    pvs:
      - /dev/sdb
      - /dev/sdc

lvm_logical_volumes:
  - name: lv_data
    vg: vg_data
    size: 100G
    fstype: ext4
    mount_point: /data
```

### 2. Run LVM Management Playbook

```bash
ansible-playbook ansible/playbooks/lvm-management.yml -i inventory/hosts.yml
```

### 3. Run Specific Phases

```bash
# Phase 1: Setup
ansible-playbook ansible/playbooks/lvm-management.yml --tags phase1

# Phase 2: Physical Volumes
ansible-playbook ansible/playbooks/lvm-management.yml --tags phase2

# Phase 3: Volume Groups
ansible-playbook ansible/playbooks/lvm-management.yml --tags phase3

# Phase 4: Logical Volumes
ansible-playbook ansible/playbooks/lvm-management.yml --tags phase4

# Phase 5: Snapshots
ansible-playbook ansible/playbooks/lvm-management.yml --tags phase5

# Phase 6: Monitoring
ansible-playbook ansible/playbooks/lvm-management.yml --tags phase6

# Phase 7: Resize
ansible-playbook ansible/playbooks/lvm-management.yml --tags phase7
```

---

## 📊 7-Phase Automation Pipeline

### Phase 1: LVM Infrastructure Setup
- Install lvm2 and device-mapper packages
- Start and enable LVM services
- Configure device mapper

### Phase 2: Physical Volume (PV) Management
- Scan for new disks
- Initialize physical volumes
- Display PV status and reports

### Phase 3: Volume Group (VG) Management
- Create volume groups from PVs
- Extend existing VGs
- Display VG status

### Phase 4: Logical Volume (LV) Management
- Create logical volumes
- Format with filesystems (ext4, xfs)
- Create mount points
- Mount volumes
- Generate reports

### Phase 5: LVM Snapshots
- Create point-in-time snapshots
- Mount snapshots (read-only)
- Prepare for backup operations

### Phase 6: Monitoring & Health
- Check disk usage
- Monitor LVM health status
- Detect warnings/errors
- Generate comprehensive reports

### Phase 7: Resize Operations
- Resize logical volumes online
- Resize physical volumes
- Extend filesystems

---

## 🔧 Common Operations

### Create a New Logical Volume

```yaml
lvm_logical_volumes:
  - name: lv_newdata
    vg: vg_data
    size: 50G
    fstype: ext4
    mount_point: /newdata
    mount_opts: defaults,noatime
    create_fs: yes
```

### Resize a Logical Volume

```yaml
lvm_resize_volumes:
  - vg: vg_data
    name: lv_data
    new_size: 150G  # Increase from 100G to 150G
```

### Create Snapshots for Backup

```yaml
lvm_snapshots:
  - vg: vg_data
    lv: lv_data
    snapshot_name: lv_data_backup
    snapshot_size: 20G
    snapshot_mount: /mnt/data_backup
```

### Extend Volume Group

```yaml
lvm_vg_extend:
  - name: vg_data
    add_pvs:
      - /dev/sde
```

---

## 📁 Directory Structure

```
ansible/
├── playbooks/
│   ├── lvm-management.yml      # Main LVM playbook (7 phases)
│   └── lvm-monitoring.yml      # Monitoring setup
├── roles/
│   └── lvm_monitoring/
│       └── tasks/main.yml      # Monitoring role tasks
├── vars/
│   └── lvm-config.yml          # Configuration variables
└── inventory/
    └── hosts.yml               # Server inventory

scripts/
├── lvm-health-check.sh         # Health monitoring script
├── lvm-snapshot-backup.sh      # Snapshot backup script
└── lvm-snapshot-cleanup.sh     # Cleanup old snapshots

Logs & Reports:
/var/log/ansible/lvm/
├── execution.log               # Execution logs
├── reports/
│   ├── pv-report-*.log         # Physical volume reports
│   ├── vg-report-*.log         # Volume group reports
│   ├── lv-report-*.log         # Logical volume reports
│   ├── snapshots-report-*.log  # Snapshot reports
│   ├── health-check-*.log      # Health check reports
│   ├── health-check-*.html     # Health check HTML reports
│   └── resize-report-*.log     # Resize operation reports
```

---

## 🖥️ Ansible Tower Integration

### 1. Create Machine Credential
- SSH Key: Upload your private key
- Username: ubuntu or your user

### 2. Create Project
```bash
git clone <your-repo> /var/lib/awx/projects/subbiah-sevu
```

### 3. Create Job Template

**Template Name**: LVM Management
- Project: subbiah-sevu
- Playbook: ansible/playbooks/lvm-management.yml
- Inventory: Default
- Credentials: Machine Credential
- Extra Variables:
  ```yaml
  lvm_disks:
    - /dev/sdb
    - /dev/sdc
  ```

### 4. Create Monitoring Job Template

**Template Name**: LVM Health Monitoring
- Playbook: ansible/playbooks/lvm-monitoring.yml
- Schedule: Every 30 minutes

### 5. Launch Job

Via Tower UI or API:
```bash
curl -X POST https://tower.example.com/api/v2/job_templates/X/launch/ \
  -H "Authorization: Bearer $TOKEN" \
  -d '{}'
```

---

## 📊 Automated Monitoring & Scheduled Tasks

### Cron Jobs Setup

Health check every 30 minutes:
```bash
*/30 * * * * /usr/local/bin/lvm-health-check.sh
```

Snapshot backup daily at 2 AM:
```bash
0 2 * * * /usr/local/bin/lvm-snapshot-backup.sh
```

Cleanup old snapshots daily at 3 AM:
```bash
0 3 * * * /usr/local/bin/lvm-snapshot-cleanup.sh
```

### Systemd Timers (Preferred)

```bash
systemctl start lvm-health-check.timer
systemctl start lvm-snapshot-backup.timer
systemctl start lvm-snapshot-cleanup.timer
```

---

## 🔍 Viewing Reports and Logs

### Check Execution Logs
```bash
cat /var/log/ansible/lvm/execution.log
```

### View LVM Health Report
```bash
tail -50 /var/log/ansible/lvm/reports/health-check-*.log
```

### Open HTML Health Report
```bash
firefox /var/log/ansible/lvm/reports/lvm-health-*.html
```

### Check LVM Summary
```bash
cat /var/log/ansible/lvm/lvm-summary-*.txt
```

---

## 🆘 Troubleshooting

### Check LVM Status Manually
```bash
# List all physical volumes
pvs

# List all volume groups
vgs

# List all logical volumes
lvs

# Check for errors
lvmdiskscan
pvck
vgck
lvscan
```

### Check Mounted Volumes
```bash
df -h | grep mapper
mount | grep /dev/mapper
```

### View LVM Events
```bash
dmesg | grep -iE "lvm|device-mapper"
journalctl -u lvm2-lvmetad.service
journalctl -u dm-event.service
```

### Unmount and Remove LV
```bash
umount /mount_point
lvremove -f /dev/vg_name/lv_name
```

---

## 📈 Performance Monitoring

### Monitor Disk I/O
```bash
iostat -x 1 10  # Every 1 second for 10 iterations
```

### Monitor LVM Operations
```bash
lvs -o lv_name,vg_name,lv_size,lv_status -a
```

### Check Snapshot Utilization
```bash
lvs -o lv_name,snap_percent  # Snapshot usage percentage
```

---

## 🔐 Security Best Practices

1. **Backup Before Changes**: Always create snapshots before modifications
2. **Test in Dev First**: Run on test servers before production
3. **Monitor Disk Space**: Set alerts at 80% threshold
4. **Regular Health Checks**: Run health checks every 30 minutes
5. **Audit Logging**: Enable audit logging for LVM changes
6. **Access Control**: Restrict LVM commands to authorized users

---

## 📝 Example: Complete Setup

### Step 1: Configure Storage

```yaml
# ansible/vars/lvm-config.yml
lvm_disks:
  - /dev/sdb
  - /dev/sdc

lvm_volume_groups:
  - name: vg_production
    pvs:
      - /dev/sdb
      - /dev/sdc

lvm_logical_volumes:
  - name: lv_app
    vg: vg_production
    size: 100G
    fstype: ext4
    mount_point: /opt/app
    mount_opts: defaults,noatime
```

### Step 2: Run Playbook

```bash
ansible-playbook ansible/playbooks/lvm-management.yml \
  -i ansible/inventory/hosts.yml \
  -e "@ansible/vars/lvm-config.yml"
```

### Step 3: Verify Setup

```bash
pvs
vgs
lvs
df -h | grep mapper
```

### Step 4: Setup Monitoring

```bash
ansible-playbook ansible/playbooks/lvm-monitoring.yml \
  -i ansible/inventory/hosts.yml
```

---

## ✅ Verification Checklist

- [ ] LVM2 packages installed
- [ ] Physical volumes created and initialized
- [ ] Volume groups created
- [ ] Logical volumes created and formatted
- [ ] Mount points created and volumes mounted
- [ ] Snapshots configured
- [ ] Health monitoring running
- [ ] Cron jobs scheduled
- [ ] Reports generating successfully
- [ ] Alerting configured

---

## 📞 Support & Documentation

For detailed LVM documentation:
```bash
man lvm
man pvs
man vgs
man lvs
man lvcreate
man vgcreate
man pvcreate
```

---

**Last Updated**: 2026-05-03
**Version**: 1.0
**Repository**: subbiahsv23-ai/subbiah-sevu
