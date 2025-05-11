
# **Prometheus and Grafana Deployment Playbooks**

This repository contains Ansible playbooks for deploying **Prometheus**, **Grafana**, and **Node Exporter** across your infrastructure. The configuration details set up monitoring for your **OpenStack instances**, **Kubernetes nodes**, and **Kubernetes pods**.
## Repository Traffic Overview

Here's the traffic overview for this repository:

- 👁️ **Total Views** Since Creation: **456** views
- 🔄 **Total Clones** Since Creation: **94** clones
- 📈 **Recent Views** (Last 14 days): **18** views
- 📊 **Recent Clones** (Last 14 days): **4** clones

---

Last traffic data update: **Sun May 11 2025 03:04:14 CET**

---
## **Overview**

The playbooks in this repository will allow you to:
- Install **Prometheus** on a control node.
- Install **Grafana** on the same control node.
- Install and configure **Node Exporter** on all nodes to gather system metrics (CPU, memory, disk, etc.), whether they are controller nodes, storage nodes, compute nodes, or Kubernetes master and worker nodes.
- Configure **Prometheus** to scrape metrics from all nodes and store them for visualization.
- Configure **Grafana** to visualize the metrics from Prometheus.

This setup is flexible enough to handle monitoring at the **infrastructure level** (controller, compute, storage nodes) as well as at the **Kubernetes cluster level** (master and worker nodes, and even the pods within the cluster).

## **Prerequisites**

- Ansible installed on your local machine or control machine.
- A working OpenStack infrastructure with nodes that have access to each other over the network.
- Root privileges on all nodes for package installations and system configuration.
- Access to the target nodes (control node, storage, compute, and Kubernetes nodes).
- Prometheus and Grafana will be installed on the **control node**, while **Node Exporter** will be installed on all nodes (whether they are part of an OpenStack setup or Kubernetes nodes).
## **Required Ports**
Ensure the following ports are accessible:
**"Control Node"**

Port 9090 (TCP): Prometheus server
Port 3000 (TCP): Grafana web interface
Port 9100 (TCP): Node Exporter (if monitoring the control node itself)

**"All Other Nodes"**

Port 9100 (TCP): Node Exporter

Make sure your firewall rules allow:

Access from your workstation to ports 9090 and 3000 on the control node for web interfaces
Access from the Prometheus server (control node) to port 9100 on all nodes for metrics collection
Inter-node communication if using clustered setups

Note: If you're using custom security groups (e.g., in OpenStack) or network policies (in Kubernetes), make sure to update them accordingly to allow these connections.
## **Repository Structure**

- **inventory**: Inventory file containing the details of the target nodes (control, storage, compute, Kubernetes master/worker nodes).
- **playbooks**:
  - **prometheus_grafana_server.yml**: Ansible playbook for installing Prometheus and Grafana on the control node.
  - **prometheus_node_exporter.yml**: Ansible playbook for installing Node Exporter on all nodes (controller, storage, compute, Kubernetes nodes).
  - **vars.yml**: Contains the list of agent IPs that Prometheus will monitor. Updating this file and re-running the playbook will automatically update the Prometheus configuration and restart the service to include new agents.
- **README.md**: This file.

## **Installation Instructions**

### Step 1: Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/hendzormati/Prometheus-Grafana-Playbooks.git
cd prometheus-grafana-Playbooks
```

### Step 2: Configure the Inventory

The **inventory** file should contain the IPs and hostnames of your nodes. Here's an example:

```ini
[control]
192.168.1.100

[storage]
192.168.1.101

[compute]
192.168.1.102
192.168.1.103
192.168.1.104
...

[k8s_master]
192.168.1.200

[k8s_worker]
192.168.1.201
192.168.1.202

[k8s_pods]
192.168.1.203
```

Make sure to replace the IPs with the actual IPs of your nodes.

### Step 3: Install Node Exporter on All Nodes

Run the playbook to install **Node Exporter** on all nodes (controller, storage, compute, Kubernetes nodes):

```bash
ansible-playbook -i inventory playbooks/prometheus_node_exporter.yml
```

### Step 4: Install Prometheus and Grafana on Control Node

Run the playbook to install **Prometheus** and **Grafana** on the control node:

```bash
ansible-playbook -i inventory playbooks/prometheus_grafana_server.yml
```

**Note**: When running the playbook for the first time, you may see the following output:
```
TASK [Check if Prometheus service is running] ***************************************
fatal: [control]: FAILED! => {"changed": false, "cmd": ["systemctl", "is-active", "prometheus"], "delta": "0:00:00.012378", "end": "2025-02-15 14:46:27.298523", "msg": "non-zero return code", "rc": 3, "start": "2025-02-15 14:46:27.286145", "stderr": "", "stderr_lines": [], "stdout": "inactive", "stdout_lines": ["inactive"]}
...ignoring
```
This is normal and expected behavior since the Prometheus service isn't running yet on the first installation. The playbook will continue and properly install and start the service.

## **Managing Monitoring Agents**

### Adding or Removing Monitoring Agents

The system uses a `vars.yml` file to maintain the list of agents that Prometheus monitors. To add or remove agents:

1. Edit the `vars.yml` file to update the list of agent IPs:
```yaml
prometheus_nodes:
  - 192.168.1.101
  - 192.168.1.102
  - 192.168.1.103
```

2. Re-run the Prometheus server playbook to apply the changes:
```bash
ansible-playbook -i inventory playbooks/prometheus_grafana_server.yml
```

This will:
- Update the Prometheus configuration with the new agent list
- Restart the Prometheus service to apply the changes
- Automatically update the available metrics in Grafana

There's no need for manual intervention - the playbook handles all necessary service restarts and configuration updates.

## **Grafana Configuration**

### Step 1: Access Grafana Dashboard

After the playbook runs successfully, you can access the Grafana dashboard through your browser. Open the following URL:

```
http://<control-node-ip>:3000
```

- Default login credentials:
  - **Username**: `admin`
  - **Password**: `admin`

You will be prompted to change the password after logging in for the first time.

### Step 2: Add Prometheus as a Data Source in Grafana

1. In Grafana, click on the **"Configuration"** (gear) icon in the left sidebar.
2. Click **"Data Sources"**.
3. Click **"Add data source"** and select **Prometheus**.
4. In the **HTTP** section, enter the following URL:
   ``` 
   http://<control-node-ip>:9090
   ```
5. Click **"Save & Test"** to confirm the connection.

### Step 3: Import a dashboard for node monitoring

1. Click on the **"+"** icon in the left sidebar and select **Import Dashboard**.
2. Enter dashboard ID: **"1860"** (this is the Node Exporter Full dashboard)
3. Click **"Load"**
4. Select your **"Prometheus"** data source
5. Click **"Import"**

## **Playbooks Details**

### **prometheus_node_exporter.yml**

This playbook installs **Node Exporter** on all nodes (controller, storage, compute, and Kubernetes nodes). It performs the following tasks:
- Downloads and extracts Node Exporter.
- Configures Node Exporter as a systemd service to run at boot.
- Starts Node Exporter on all nodes.

### **prometheus_grafana_server.yml**

This playbook installs **Prometheus** and **Grafana** on the control node. It performs the following tasks:
- Downloads and installs Prometheus.
- Configures Prometheus to scrape metrics from all nodes (controller, storage, compute, Kubernetes nodes).
- Installs Grafana and configures it to visualize metrics from Prometheus.

## **Useful Links**

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)

## **Contributing**

If you have suggestions or improvements, feel free to create an issue or submit a pull request.

## **License**

See the LICENSE file for license rights and limitations (MIT).
