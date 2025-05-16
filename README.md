# ZFS Replication with Ansible

A modular Ansible solution for ZFS dataset replication between hosts using syncoid and sanoid.

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/mtelvers/zfs-replication-ansible.git
   cd zfs-replication-ansible
   ```

2. Create your dataset configuration files in `dataset_configs/`:
   ```bash
   cp dataset_configs/tank_data.yml dataset_configs/my_dataset.yml
   # Edit my_dataset.yml with your dataset information
   ```

3. Update your inventory file:
   ```bash
   vi inventory.ini
   # Add your hosts
   ```

4. Run the playbook:
   ```bash
   ansible-playbook syncoid_sanoid_deploy.yml
   ```

5. Push to GitHub and open [docs/replication_topology.md](docs/replication_topology.md)

## Directory Structure

```
.
├── inventory.ini                   # Your Ansible inventory
├── syncoid_sanoid_deploy.yml       # Main playbook
├── templates/                      # Jinja2 templates
│   ├── sanoid.conf.j2              # Sanoid configuration template
│   └── replication_diagram.md.j2   # Mermaid diagram template
├── dataset_configs/                # Dataset configuration files
│   ├── tank_data.yml               # Example dataset
│   ├── tank_backups.yml            # Example dataset
│   └── tank_database.yml           # Example dataset
└── documentation/                  # Generated documentation
    └── replication_topology.md     # Generated Mermaid diagram
```

## Dataset Configuration

Each dataset is defined in its own YAML file with the following structure:

```yaml
---
dataset_path: "tank/data"           # ZFS dataset path
source_host: "zfs-source"           # Source host for this dataset
target_host: "zfs-target"           # Target host for this dataset

# Optional: Override cron schedules
syncoid_cron_minute: "15"           # When to run replication (minutes)
syncoid_cron_hour: "*/2"            # When to run replication (hours)
sanoid_cron_minute: "0"             # When to run snapshots (minutes)
sanoid_cron_hour: "*"               # When to run snapshots (hours)

# Optional: Additional syncoid options
syncoid_options: "--no-sync-snap --compress=zstd"

# Optional: Custom snapshot retention policy
retention:
  frequently: 0                     # Frequent snapshots (per 15 min)
  hourly: 36                        # Hourly snapshots to keep
  daily: 30                         # Daily snapshots to keep
  monthly: 12                       # Monthly snapshots to keep
  yearly: 2                         # Yearly snapshots to keep
  autosnap: "yes"                   # Enable automatic snapshots
  autoprune: "yes"                  # Enable automatic pruning
```

## Testing

Create a ZFS tank on a few hosts:

```
for a in x86-bm-c1.sw.ocaml.org x86-bm-c2.sw.ocaml.org x86-bm-c3.sw.ocaml.org ; do
  ssh $a fallocate -l 50GiB /root/tank.zfs
  ssh $a zpool create tank /root/tank.zfs
  ssh $a zfs list
done
ssh x86-bm-c1.sw.ocaml.org zfs create tank/dataset-01
ssh x86-bm-c1.sw.ocaml.org zfs create tank/dataset-02
ssh x86-bm-c2.sw.ocaml.org zfs create tank/dataset-03
```

