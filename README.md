# Multi-Node OpenShift Deployment (3-Node)

This directory contains the configuration files required to deploy a 3-node Multi-Node OpenShift (MNO) cluster using the **Agent-based Installer**. This setup typically consists of 3 Master nodes that also run worker workloads (Masters-as-Workers).

## Prerequisites

- **OpenShift Installer**: `openshift-install` binary.
- **Pull Secret**: Obtained from [console.redhat.com](https://console.redhat.com/openshift/install/pull-secret).
- **SSH Key**: Public key for the jump host to access nodes via SSH.
- **Infrastructure**: 3 Physical/VM nodes with fixed MAC addresses and networking.

## Configuration Files

### 1. `install-config.yaml`
This file contains the cluster-wide configuration specifications.
- **`baseDomain`**: Your actual domain (e.g., `lab.local`).
- **`metadata.name`**: Name of your OpenShift cluster.
- **`controlPlane.replicas`**: Set to `3` for Compact Cluster.
- **`compute.replicas`**: Set to `0` (Master nodes will handle workloads).
- **`platform.baremetal`**: Defines the `apiVIPs` and `ingressVIPs` for cluster access.

### 2. `agent-config.yaml`
This file defines node-specific configurations for the Agent-based installer.
- **`rendezvousIP`**: The IP of the first master node (Master 0).
- **`hosts`**: List of host configurations including:
    - `hostname`: Descriptive name for each node.
    - `role`: Set to `master` for all three nodes.
    - `rootDeviceHints`: Serial number of the OS installation disk (SSD/NVMe)/ incase for server put raid controller serial number.
    - `interfaces`: MAC address and interface name of the primary NIC.
    - `networkConfig`: Static IP, Gateway, and DNS configuration for each node.

## Customization Guide

1. **Update `install-config.yaml`**:
   - Replace `<Domain>` with your base domain.
   - Replace `<Cluster-Name>` with your cluster name.
   - Replace `<API-VIP>` and `<Ingress-VIP>` with reserved IPs in your subnet.
   - Paste your `<Pull Secret>` and `<SSH-KEY>`.

2. **Update `agent-config.yaml`**:
   - For each node (Master 1, 2, 3), replace the placeholders:
     - `<master-X-hostname>`
     - `<master-X-OS-SSD-serial-number>`
     - `<master-X-NIC-interface-name>`
     - `<master-X-NIC-mac-address>`
     - `<master-X-ip>`
     - `<master-X-gateway-ip>`
     - `<dns-server-1>`

## Building the Agent ISO

Once the configuration files are ready, follow these steps to generate the installation ISO:

1.  **Download the installer**:
    ```bash
    # Example for OpenShift 4.14
    curl -O https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
    tar -xvf openshift-install-linux.tar.gz
    ```

2.  **Generate the Agent ISO**:
    ```bash
    ./openshift-install agent create image --dir .
    ```
    This will generate an `agent.iso` file in the current directory.

## Deployment

1.  **Boot the nodes**: Mount the `agent.iso` to all 3 nodes and boot them.
2.  **Installation**: The nodes will boot into the Agent-based installer, configure their networking based on `agent-config.yaml`, and start the installation process automatically.
3.  **Monitor**:
    ```bash
    ./openshift-install agent wait-for install-complete --dir .
    ```

## Post-Installation

Once the installation is complete, you can access the cluster:
- **Console**: `https://console-openshift-console.apps.<cluster-name>.<domain>`
- **CLI**: The `auth` directory generated during the build will contain the `kubeconfig` and `kubeadmin-password`.
