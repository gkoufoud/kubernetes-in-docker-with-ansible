# kubernetes-in-docker-with-ansible
Kubernetes Bootstrap with Ansible and Docker Compose
# 🚀 Kubernetes Bootstrap with Ansible and Docker Compose

This project provides a lightweight and repeatable way to bootstrap multiple **Kubernetes control planes** using **Docker Compose**, and register **bare-metal or virtual worker nodes** using **Ansible**.

---

## 🧱 Key Features

- 🐳 **Dockerized Control Plane**: Each Kubernetes control plane runs in isolated Docker containers using `docker-compose`.
- 🔁 **Multi-Cluster Support**: You can run multiple independent Kubernetes clusters on the same control host.
- ⚙️ **Ansible-Driven**: Infrastructure configuration is fully automated with modular Ansible roles.
- 👷 **Declarative Worker Configuration**: Just list your worker nodes in `vars.yml` to bootstrap them automatically.
- 🧩 **Optional HAProxy**: SNI-based TLS routing across clusters using HAProxy.
- 🌐 **Calico CNI Integration**: Automatically installs Calico CNI.
- 🔐 **Automatic Kubelet Certificate Approval**: Uses [kubelet-csr-approver](https://github.com/postfinance/kubelet-csr-approver) for seamless node onboarding.

---

## 🧭 Architecture Diagram

![Cluster Architecture](assets/diagram.drawio.svg)

---

## 🛠️ Implementation Details

This project **does not use `kubeadm`**. Instead, the Kubernetes control plane is launched directly with `docker-compose` using the official components:

- `etcd`
- `kube-apiserver`
- `kube-controller-manager`
- `kube-scheduler`

Each component is configured explicitly via Ansible templates.

Worker nodes join the cluster using **bootstrap tokens**, and **serving certificates are automatically approved** using the [`kubelet-csr-approver`](https://github.com/postfinance/kubelet-csr-approver) controller.

This enables a clean and flexible setup with complete control over the Kubernetes internals.

---

## 📁 Directory Structure

```text
├── main.yml                         # Main Ansible playbook
├── vars.yml                         # Your configuration: clusters, versions, hosts
├── roles/
│   ├── certs/                       # Handles SSL certificate generation
│   ├── control_plane_setup/        # Brings up etcd, kube-apiserver, scheduler, controller
│   ├── haproxy_setup/              # Optional: configures HAProxy for TLS routing
│   ├── kubeconfigs/                # Generates kubeconfig files for each component
│   └── worker_setup/               # Installs and configures kubelet, containerd on worker nodes
```


---

## 🖥️ Terminology

- **Control Host**: A physical or virtual machine (typically Debian-based) where a Kubernetes control plane runs using Docker containers.
- **Cluster**: A full Kubernetes environment (control plane + workers).
- **Worker Node**: A machine (physical or virtual) that runs the Kubernetes kubelet and joins the cluster.

---

## ⚙️ How to Use

### 1. 📦 Requirements

- Ansible
- Python 3.x
- Docker and Docker Compose installed on the control hosts
- Debian-based systems (tested on **Debian Bookworm**)

### 2. 📁 Define Your Clusters in `vars.yml`

```yaml
clusters:
  - name: cluster1
    control_host: 192.168.0.130
    api_port: 6443
    service_cidr: "10.3.0.0/16"
    pod_cidr: "10.2.0.0/16"
    k8s_version: "1.31.0"
    calico_version: "3.30.2"
    workers:
      - name: "c1n1"
        ip: 192.168.0.131
      - name: "c1n2"
        ip: 192.168.0.132
```
⚠️ Make sure each cluster on the same control host has a unique api_port.

### 3. ▶️ Run the Playbook
```bash
ansible-playbook main.yml
```
This will:

- Generate all necessary certs
- Deploy the control plane (etcd, kube-apiserver, scheduler, - controller)
- Optionally configure HAProxy
- Bootstrap all worker nodes

📌 Notes

- 🐧 Only Debian-based systems are currently supported.
- 🕸️ The kube-apiserver must be able to resolve worker node - DNS names (they're added via extra_hosts in the Compose file).
- 🔁 CSR auto-approval is handled by the lightweight controller kubelet-csr-approver.

🌐 HAProxy SNI Routing (Optional)

If haproxy is defined in vars.yml, a dynamic config is created to forward TLS traffic based on the SNI to the correct Kubernetes API server.

```yaml
haproxy:
  host: 192.168.0.70
  config_dir: /opt/haproxy/cfg
```

🧪 Tested On

- ✅ Debian 12 (Bookworm)
- ✅ Kubernetes 1.31.0
- ✅ Calico CNI 3.30.2
- ✅ containerd 1.7.2+

🧾 License

This project is licensed under the MIT License.


