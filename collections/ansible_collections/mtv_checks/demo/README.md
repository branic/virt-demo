# mtv_checks.demo

Collection roles for **MTV / OpenShift Virtualization** checks. Playbooks at the
repo root pass **no** `vars:`; inputs come from **Ansible Automation Platform**
(job template **surveys** and **credential injection**).

## Roles

- **precheck** — VMware REST `vcenter_vm_info` (VM must exist); `k8s_info` shows
  KubeVirt `VirtualMachine` must be **absent** pre-migration.
- **postcheck** — `k8s_info`: KubeVirt `VirtualMachine` must **exist**
  post-migration.
- **run** — example debug role for `site.yml`.

See the repository **README** for variable names to bind in AAP (`mtv_vcenter_*`,
`mtv_kubevirt_namespace`, `mtv_kubeconfig`, survey fields).

## Licensing

GNU General Public License v3.0 or later.
