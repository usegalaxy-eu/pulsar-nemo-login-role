# pulsar_nemo_login (Ansible role)

Deploys [Pulsar](https://github.com/galaxyproject/pulsar) on an HPC **login
node** in **relay mode**, submitting jobs to **Slurm**. Built for the
bwForCluster NEMO (University of Freiburg) integration with usegalaxy.eu.

Because NEMO login nodes have no user-level systemd, Pulsar is kept alive by a
small auto-restart wrapper script rather than a service unit.

## What it does

- Installs Pulsar into a virtualenv on the login node
- Templates `app.yml` (relay connection + Slurm manager)
- Installs `job_metrics_conf.xml` (core, cgroup, hostname plugins)
- Installs the `start_pulsar.sh` auto-restart wrapper

## Important

- The manager is `queued_cli` with `job_plugin: Slurm`. This is required so
  jobs are **submitted to Slurm** and run on compute nodes, `queued_python`
  would run them on the login node, which is not allowed on shared HPC.
- Supply the relay URL and credentials via vault / extra-vars. The defaults are
  placeholders.

## Role variables

See `defaults/main.yml`.

| Variable | Default | Notes |
|----------|---------|-------|
| `pulsar_nemo_home` | `~/pulsar` | Install location |
| `pulsar_nemo_version` | `main` | Pin a Pulsar commit/tag that works with the node's Python |
| `pulsar_nemo_relay_url` | `http://CHANGE_ME:9000/` | **override** |
| `pulsar_nemo_relay_password` | `CHANGE_ME` | **override** |

## Example

```yaml
- hosts: nemo_login
  roles:
    - role: pulsar_nemo_login
      vars:
        pulsar_nemo_relay_url: "{{ vault_pulsar_nemo_relay_url }}"
        pulsar_nemo_relay_password: "{{ vault_pulsar_nemo_relay_password }}"
```

After deploy, start Pulsar with the wrapper:

```bash
cd ~/pulsar && nohup ./start_pulsar.sh > /dev/null 2>&1 &
```
