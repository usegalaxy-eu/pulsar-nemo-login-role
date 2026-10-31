# pulsar_nemo_login (Ansible role)

Deploys [Pulsar](https://github.com/galaxyproject/pulsar) on an HPC **login node** in **relay mode**, submitting jobs to **Slurm**. Built for bwForCluster NEMO and usegalaxy.eu.

NEMO login nodes have no user-level systemd, so Pulsar runs under supervisord in user space, started on boot by an `@reboot` cron entry.

## What it does

- venv on the NEMO Python module (3.11, the relay client needs >= 3.10)
- installs `pulsar-app` and `pulsar-relay-client`, both pinned
- templates `app.yml` (relay connection, staging dir, Slurm manager) and `job_metrics_conf.xml`
- supervisord config + launcher, `@reboot` cron
- optional: monthly `ws_extend` so the staging workspace doesn't expire

## Important

- manager is `queued_cli` with `job_plugin: Slurm`, so jobs run on compute nodes. `queued_python` would run them on the login node, which is not allowed on shared HPC
- use a non-admin relay account
- home has a 100 GB quota, put staging in a workspace
- cron only has `/usr/bin:/bin` in PATH and NEMO's `ws_extend` calls `ws_allocate` by name. the role sets PATH, output goes to `~/pulsar/ws_extend.log`

## Role variables

| Variable | Default | Notes |
|----------|---------|-------|
| `pulsar_nemo_home` | `~/pulsar` | install location |
| `pulsar_nemo_staging_directory` | `~/pulsar/jobs_directory` | job staging, use a workspace on NEMO |
| `pulsar_nemo_workspace` | `""` | workspace to extend monthly, empty = off |
| `pulsar_nemo_python_module` | `lang/python/3.11.5-gcccore-13.2.0` | |
| `pulsar_nemo_version` | `0.15.15` | pulsar-app |
| `pulsar_nemo_relay_client_version` | `==0.2.1` | match the relay |
| `pulsar_nemo_relay_url` | `http://CHANGE_ME:9000/` | **override** |
| `pulsar_nemo_relay_username` | `CHANGE_ME` | **override**, non-admin |
| `pulsar_nemo_relay_password` | `CHANGE_ME` | **override** |
| `pulsar_nemo_relay_allow_insecure` | `true` | `false` once the relay has https |
| `pulsar_nemo_start` | `true` | `false` = install only, don't start |
| `pulsar_nemo_slurm_params` | `{}` | fixed slurm settings for every job, e.g. `partition: milan`, `time: "4-0"`, `ncpus: 4`, `"--mem-per-cpu": 3800M`. queued_cli ignores galaxy's native spec |

## Example

```yaml
- hosts: nemo_login
  roles:
    - role: pulsar_nemo_login
      vars:
        pulsar_nemo_staging_directory: /work/classic/fr_xx1234-pulsar/staging
        pulsar_nemo_workspace: pulsar
        pulsar_nemo_relay_url: "{{ vault_pulsar_nemo_relay_url }}"
        pulsar_nemo_relay_username: "{{ vault_pulsar_nemo_relay_username }}"
        pulsar_nemo_relay_password: "{{ vault_pulsar_nemo_relay_password }}"
        pulsar_nemo_relay_allow_insecure: false
```

create the workspace once before: `ws_allocate pulsar 100`

status / restart: `~/pulsar/.venv/bin/supervisorctl -c ~/pulsar/supervisord.conf status` (or `restart pulsar`)

## Status

installed and checked on NEMO login2 in Oct 2026: pulsar-app 0.15.15 + pulsar-relay-client 0.2.1 on Python 3.11.5. not yet connected to the production relay.

## Author

Created by Dmitrijs Sizovs in 2026.
