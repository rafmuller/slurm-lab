# Slurm Cluster via Vagrant to learn Slurm

This small repository is used to easily setup a Slurm cluster for learning purposes. Using VirtualBox and Vagrant, you can easily setup a Slurm cluster with multiple nodes on a laptop or personal computer. Not intended for production use, but rather for learning purposes. A laptop with 32GB of RAM would be more than sufficient to start a cluster with 3 nodes.

## Key Capabilities

### 1. Dynamic Cluster Scaling

The cluster is defined by a single Ruby hash (`slurm_cluster`) in the `Vagrantfile`. Adding or removing nodes is as simple as updating this hash:

- **Automatic VM Definition**: Vagrant automatically creates and configures the network for every node in the hash.
- **Dynamic `/etc/hosts`**: Every node in the cluster is automatically aware of every other node by hostname.

### 2. Template-Driven Configuration

Instead of static configuration files, this project uses **ERB templates** (located in `templates/`). This allows for:

- **Dynamic `slurm.conf`**: The Slurm configuration is automatically generated to include all defined worker nodes and point to the correct controller.
- **Secure Secret Injection**: Database passwords and other sensitive environment variables are injected into configuration files at runtime without being hardcoded in the source.
- **Reusable Scripts**: Provisioning scripts are broken down into logical templates (Munge, MariaDB, Slurmctld, Slurmd) for better maintainability.

### 3. Reliable File Provisioning

The project uses **Quoted Heredocs** (`cat <<'EOF'`) within provisioning scripts. This ensures that complex configuration files with special characters are written to the virtual machines exactly as rendered by the templates, avoiding shell expansion errors.

## Prerequisites

- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Environment variable `SLURM_MARIADB_PASS` set on your host machine.

## Getting Started

1. **Set your database password**:

   ```bash
   export SLURM_MARIADB_PASS="your_secure_password"
   ```

2. **Launch the cluster**:

   ```bash
   vagrant up
   ```

3. **Verify the cluster**:

   SSH into the controller and check the node status:

   ```bash
   vagrant ssh controller
   sinfo
   ```

## Project Structure

- `Vagrantfile`: The main infrastructure-as-code definition.
- `templates/`:
    - `all_nodes_linux.sh.erb`: Base OS configuration.
    - `all_nodes_slurm.conf.erb`: The master Slurm configuration template.
    - `controller_mariadb.sh.erb`: MariaDB installation and security setup.
    - `controller_slurmdb.sh.erb`: Slurm Database Daemon setup.
    - `controller_slurmctl.sh.erb`: Slurm Controller installation.
    - `worker_slurm_install.sh.erb`: Slurm Worker installation.
