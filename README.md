# ansible-aipscan

Installs [AIPscan] on Linux hosts using Ansible.

## Compatibility

This role targets AIPscan v0.9.0 and newer. The Molecule test matrix covers
Ubuntu 22.04, 24.04, and 26.04, plus Rocky Linux 9.

> [!IMPORTANT]
> For legacy deployments targeting AIPscan v0.8.0 or earlier, use the
> [legacy-0.7] tag of this role. If you are upgrading and need to migrate an
> existing SQLite database, see the [migration notes][sqlite-migration].

## Role variables

Review [`defaults/main.yml`] for the full list of tunable variables. The file
includes inline comments for clarity and self-documentation.

## Dependencies

AIPscan depends on **MySQL** and **RabbitMQ**. In most deployments, **Nginx** is
used as a load balancer and to serve static files, while **Typesense** can
optionally be deployed to improve reporting performance.

The choice of how to deploy and manage these services is left to the system
administrator. See [`molecule/default/requirements.yml`] for the supporting
Ansible roles used in our test pipeline.

## Example playbook

Take a look at [`molecule/default/converge.yml`], which we're using in our
testing pipeline, for a complete example.

Below is a minimal example playbook that installs AIPscan with a single storage
source:

```yaml
- hosts: aipscan_server
  roles:
    - role: artefactual.aipscan
      vars:
        aipscan_secret_key: "secretkeyvalue"
        aipscan_storage_sources:
          - name: "Demo Storage Service"
            url: "http://192.168.1.100:8000"
            username: "demouser"
            api_key: "demoapikey"
```

## Upgrading

Set `aipscan_version` to the release you want to deploy. Leave
`aipscan_python_version` empty to use that release's `.python-version` file. For
example, to deploy the release candidate:

```yaml
aipscan_version: "0.9.0rc2"
aipscan_python_version: ""
```

The role uses uv to install Python as the AIPscan service user. When the
requested interpreter differs from the existing virtualenv's interpreter, it
stops the web and worker services and rebuilds the virtualenv. Packages added
manually to that virtualenv must be installed again. The database and downloads
directory are retained. Allow for downtime while the environment is rebuilt and
the package is installed.

Changes to installed packages also restart both services, even when Python and
configuration stay the same. Restarts happen after database migrations and
service configuration. If installation fails after services have stopped, fix
the failure and rerun the role to complete the deployment.

If you pin `aipscan_uv_version`, use a release that knows about the requested
Python version; uv's available Python downloads depend on its release. See
[uv's Python version documentation][uv-python].

Run the upgrade scenario with:

```sh
make test SCENARIO=upgrade DISTRO=ubuntu2404
```

It installs rc1 on Python 3.14, upgrades to rc2 on Python 3.15.1, checks
idempotence, and exercises package changes without replacing Python.

## Tags

The role exposes the following tags to run a subset of tasks:

- `uv` – install or update the uv packaging tool.
- `install` – create the managed virtual environment and install AIPscan.
- `database` – run Flask database migrations.
- `service` – update configuration and manage systemd units.

## License

This project is licensed under the Apache-2.0 license ([LICENSE] or
<https://opensource.org/licenses/Apache-2.0>).

[aipscan]: https://github.com/artefactual-labs/AIPscan
[legacy-0.7]: https://github.com/artefactual-labs/ansible-aipscan/releases/tag/legacy-0.7
[license]: ./LICENSE
[sqlite-migration]: https://github.com/artefactual-labs/AIPscan/tree/v0.8.0-beta?tab=readme-ov-file#production-deployments
[uv-python]: https://docs.astral.sh/uv/concepts/python-versions/
[`defaults/main.yml`]: ./defaults/main.yml
[`molecule/default/converge.yml`]: ./molecule/default/converge.yml
[`molecule/default/requirements.yml`]: ./molecule/default/requirements.yml
