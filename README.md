# MTV Demo Ansible Project

## Local environment (`.venv`)

Use the project virtual environment for Ansible tooling (`ansible-playbook`, `ansible-galaxy`, `ansible-lint`, `ansible-navigator`, `yamllint`). From the repository root:

```bash
source .venv/bin/activate
```

Install Galaxy dependencies (for example `vmware.vmware_rest` and `kubernetes.core`) into `./collections` after activating the venv:

```bash
ansible-galaxy collection install -r collections/requirements.yml -p collections
```

If the full requirements file fails (for example a git-sourced collection is unreachable), install only what the migration check playbooks need:

```bash
ansible-galaxy collection install 'vmware.vmware_rest:4.10.0' 'kubernetes.core:5.2.0' -p collections
```

Controller / AAP job templates use your **execution environment**, not this repo’s `.venv`.

**Ansible tooling:** On **Python 3.14**, `ansible-compat` (used by `ansible-lint`)
requires **ansible-core ≥ 2.20**. Install pinned versions with:

```bash
uv pip install -r requirements-ansible.txt
```

## MTV pre/post migration check playbooks

Root playbooks for an AAP workflow (pre → [ocp-virt migration](https://github.com/rhpds/ocp-virt) → post):

- [`pre_migration_checks.yml`](pre_migration_checks.yml) — For each VM in `user_selected_vm_names`: **exists in VMware** (`vmware.vmware_rest.vcenter_vm_info`) and **does not exist** as a KubeVirt `VirtualMachine` on OpenShift (`kubernetes.core.k8s_info`).
- [`post_migration_checks.yml`](post_migration_checks.yml) — For each VM: **exists** as a KubeVirt `VirtualMachine` (`kubernetes.core.k8s_info`).

The playbooks **do not define `vars:`**. Supply everything through the **job template** in Ansible Automation Platform: **credentials** (injected as variables on the job) and a **survey** / extra variables.

### Survey (ocp-virt Basic Survey)

Use the same variable names as the [ocp-virt](https://github.com/rhpds/ocp-virt) migration job: `provider_name`, `plan_name`, `storagemap_name`, `networkmap_name`, `user_selected_vm_names`. The roles read them when present (for logging / context).

### Credentials and extra variables (bind on the job template)

Configure your credential types (or custom credential / extra vars) so the job receives at least:

| Variable | When | Purpose |
| --- | --- | --- |
| `mtv_vcenter_hostname` | Pre only | vCenter host |
| `mtv_vcenter_username` | Pre only | vCenter user |
| `mtv_vcenter_password` | Pre only | vCenter password |
| `mtv_vcenter_validate_certs` | Pre (optional) | VMware TLS validation (default true if unset) |
| `mtv_kubevirt_namespace` | Pre and post (optional) | Namespace for KubeVirt `VirtualMachine` objects (defaults to `default` if unset) |
| `mtv_kubeconfig` | Pre and post (optional) | Path to kubeconfig for `kubernetes.core`; omit if using `KUBECONFIG` / in-cluster auth in the execution environment |

Map fields from your VMware and OpenShift/Kubernetes credentials to these names in the automation controller, or expose them as **extra variables** on the job template where appropriate.

The **execution environment** for the pre-check job must include **`aiohttp`** (required by `vmware.vmware_rest`). The VMware REST API targets **vSphere 7.0.3 or newer**.

Roles live under [`collections/ansible_collections/mtv_checks/demo/`](collections/ansible_collections/mtv_checks/demo/).

## Included content/ Directory Structure

The directory structure follows best practices recommended by the Ansible
community. Feel free to customize this template according to your specific
project requirements.

```shell
 ansible-project/
 |── .devcontainer/
 |    └── docker/
 |        └── devcontainer.json
 |    └── podman/
 |        └── devcontainer.json
 |    └── devcontainer.json
 |── .github/
 |    └── workflows/
 |        └── tests.yml
 |    └── ansible-code-bot.yml
 |── .vscode/
 |    └── extensions.json
 |── collections/
 |   └── requirements.yml
 |   └── ansible_collections/
 |       └── project_org/
 |           └── project_repo/
 |               └── README.md
 |               └── roles/sample_role/
 |                         └── README.md
 |                         └── tasks/main.yml
 |── inventory/
 |   |── hosts.yml
 |   |── argspec_validation_inventory.yml
 |   └── groups_vars/
 |   └── host_vars/
 |── ansible-navigator.yml
 |── ansible.cfg
 |── devfile.yaml
 |── linux_playbook.yml
 |── network_playbook.yml
 |── README.md
 |── site.yml
```

## Validation

With `.venv` active and collections installed under `./collections`:

```bash
export ANSIBLE_LOCAL_TEMP="$PWD/.ansible/tmp"
mkdir -p "$ANSIBLE_LOCAL_TEMP"
ansible-playbook --syntax-check pre_migration_checks.yml post_migration_checks.yml site.yml
ansible-lint pre_migration_checks.yml post_migration_checks.yml site.yml \
  collections/ansible_collections/mtv_checks/demo/
yamllint -d relaxed pre_migration_checks.yml post_migration_checks.yml site.yml
```
