# VKS Lab Automation Scripts - Complete Documentation

A comprehensive collection of shell scripts that automate common tasks in the VMware vSphere Kubernetes Services (VKS) learning lab environment. These scripts streamline repetitive tasks and help students focus on learning VKS concepts rather than typing commands.

**Lab Environment:** TMM-2026-2-NODE-VKS-3.6.2-LAB-DEPLOYED | **Kubernetes Version:** v1.35 | **vSphere:** 9.0.2

---

## 🚀 Quick Start (30 seconds)

```bash
cd "/Users/charlesl/Documents/Development/2 node vks scripts"
./vks-help                    # View quick reference
./login-ns1                   # Login to Supervisor
./create-vks1                 # Create a VKS cluster
```

---

## 📚 Table of Contents

1. [Start Here](#start-here)
2. [Prerequisites](#prerequisites)
3. [File Structure](#file-structure)
4. [Main Scripts](#main-scripts)
5. [Configuration](#configuration)
6. [Typical Workflows](#typical-workflows)
7. [Troubleshooting](#troubleshooting)
8. [Advanced Usage](#advanced-usage)

---

## Start Here

### For New Users

1. **Run the quick reference:** `./vks-help`
2. **Test connectivity:** `./login-ns1`
3. **Create your first cluster:** `./create-vks1`

### For Experienced Users

All scripts use configuration from `vks-lab.conf`. Edit this file to customize IP, credentials, cluster names, and resource configurations.

---

## Prerequisites

Before using these scripts, verify you have:

- ✅ Lab environment deployed and running
- ✅ Network access to Supervisor IP (typically 10.1.7.2)
- ✅ Terminal access on your lab workstation
- ✅ `vcf` CLI installed (`vcf version` should work)
- ✅ `kubectl` installed (`kubectl version` should work)

---

## File Structure

```
2 node vks scripts/
├── Configuration
│   └── vks-lab.conf                         # ⚙️  Central config (passwords, IPs, names)
│
├── Main Scripts (Run these)
│   ├── ns1-login                            # 🔐 Login to Supervisor namespace
│   ├── vks1-login                           # 🔐 Login to VKS cluster
│   ├── vks1-create                          # ✨ Create a new VKS cluster
│   ├── vks1-delete                          # 🗑️  Delete VKS cluster
│   ├── select-context                       # 🔄 Interactive context manager
│   ├── vks1-install-cert-manager            # 📦 Install cert-manager addon
│   └── vks1-delete-cert-manager             # 📦 Remove cert-manager addon
│
├── Utilities
│   ├── vks-help                             # 📖 Quick reference guide
│   └── fix-storage-quota-certificates.sh    # 🔧 Certificate maintenance
│
└── Documentation (Read these)
    ├── README.md                            # 📖 This file
    ├── ARCHITECTURE.md                      # 🏗️  System design and workflows
    ├── SETUP_INSTRUCTIONS.md                # 📋 Step-by-step setup
    ├── README_VKS_SCRIPTS.md                # 📚 Detailed script documentation
    ├── README_CERTIFICATE_FIX.md            # 📚 Certificate fix guide
    ├── README_CERT_MANAGER_INSTALL.md       # 📚 Cert-manager install guide
    └── README_DELETE_CERT_MANAGER.md        # 📚 Cert-manager delete guide
```

---

## Main Scripts

### ns1-login

**Login to the Supervisor namespace**

```bash
./ns1-login
```

**What it does:**
1. Creates a Supervisor context (if it doesn't exist)
2. Selects the Supervisor context and namespace
3. Validates the connection
4. Displays cluster nodes

**Prerequisites:**
- Network access to Supervisor IP
- Valid credentials in `vks-lab.conf`

---

### vks1-login

**Login to the VKS cluster**

```bash
./vks1-login
```

**What it does:**
1. Creates a VKS cluster context (if it doesn't exist)
2. Selects the VKS cluster context
3. Validates the connection
4. Displays cluster nodes

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- VKS cluster must be in Ready state
- Cluster must exist

---

### vks1-create

**Create a new VKS cluster**

```bash
./vks1-create
```

**What it does:**
1. Verifies Supervisor context is active
2. Generates a VKS cluster manifest from configuration
3. Applies the manifest to create the cluster
4. Monitors cluster creation progress
5. Displays configuration summary

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- Supervisor namespace must exist and be configured
- Storage policies must be configured in the namespace

**Expected Time:**
- ~10-15 minutes in nested environments
- ~2-5 minutes in physical environments

**Output:**
- Creates `vks.yaml` manifest file
- Shows live progress of cluster creation
- Press Ctrl+C to stop monitoring (cluster continues creating)

---

### vks1-delete

**Delete a VKS cluster**

```bash
./vks1-delete
```

**What it does:**
1. Verifies Supervisor context is active
2. Confirms cluster exists
3. Prompts for confirmation before deletion
4. Sends delete command
5. Monitors deletion progress

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- VKS cluster must exist
- User must confirm deletion

**Expected Time:**
- ~5-10 minutes for deletion to complete

---

### select-context

**Interactive context management**

```bash
./select-context
```

**What it does:**
1. Lists all available VCF contexts
2. Presents menu of operations:
   - Select a different context
   - Refresh the current context
   - Exit

**Menu Options:**
- **Option 1:** Interactively select a context from the list
- **Option 2:** Refresh the current context with new credentials
- **Option 3:** Exit the tool

**Use Cases:**
- Switching between Supervisor and VKS cluster contexts
- Refreshing expired authentication tokens (context expires after ~10 hours)
- Viewing current context status

---

### vks1-install-cert-manager

**Install cert-manager addon on VKS cluster**

```bash
./vks1-install-cert-manager
```

**What it does:**
1. Verifies Supervisor context is valid
2. Verifies VKS cluster exists and is ready
3. Fetches the latest available cert-manager version
4. Generates the data values configuration
5. Installs the cert-manager addon
6. Monitors the installation progress
7. Verifies the installation in the cluster

**Prerequisites:**
- Must be logged into the Supervisor: `./ns1-login`
- VKS cluster must exist: `./vks1-create`
- VKS cluster should be in Ready state

**Typical Installation Time:**
- ~2-5 minutes

---

### vks1-delete-cert-manager

**Remove cert-manager addon from VKS cluster**

```bash
./vks1-delete-cert-manager
```

**What it does:**
1. Verifies Supervisor context is valid
2. Verifies VKS cluster exists
3. Checks if cert-manager addon is installed
4. Requests confirmation before deletion
5. Deletes the cert-manager addon
6. Monitors the deletion progress
7. Verifies the removal in the cluster

**Prerequisites:**
- Must be logged into the Supervisor: `./ns1-login`
- VKS cluster must exist
- Cert-manager addon must be installed on the cluster

**Typical Deletion Time:**
- ~1-5 minutes

---

## Configuration

### Configuration File: `vks-lab.conf`

All scripts use a shared configuration file. Edit this file to customize your lab environment.

```bash
# Supervisor Access
SUPERVISOR_IP="10.1.7.2"
SUPERVISOR_USERNAME="administrator@vsphere.local"
SUPERVISOR_PASSWORD="VMware123!VMware123!"

# Namespace
SUPERVISOR_NAMESPACE_NAME="ns1"
SUPERVISOR_CONTEXT="ns1-ctx"

# VKS Cluster
CLUSTER_NAME="vks1"
CLUSTER_CONTEXT="vks1-ctx"
CLUSTER_KUBERNETES_VERSION="v1.35"

# Cluster Resources
CLUSTER_CONTROL_PLANE_REPLICAS=1
CLUSTER_WORKER_REPLICAS=2
CLUSTER_VM_CLASS_CONTROL_PLANE="best-effort-medium"
CLUSTER_VM_CLASS_WORKER="best-effort-xlarge"
CLUSTER_STORAGE_CLASS="wcp-storage"
STORAGE_SIZE="60Gi"

# Timeouts and Retries
KUBECTL_TIMEOUT=300
CONTEXT_WAIT_TIMEOUT=60
CLUSTER_CREATION_TIMEOUT=900  # 15 minutes
```

### Modifying Configuration

Edit `vks-lab.conf` to customize:
- Supervisor IP address and credentials
- Cluster naming
- Kubernetes version
- Resource sizes (VM classes, storage)
- Timeout values

```bash
nano vks-lab.conf
# or
vi vks-lab.conf
```

---

## Typical Workflows

### Workflow 1: Create and Access a VKS Cluster

```bash
# 1. Login to Supervisor
./ns1-login

# 2. Create a new VKS cluster (10-15 minutes)
./vks1-create

# 3. When cluster is ready, login to it
./vks1-login

# 4. Deploy applications
kubectl apply -f my-app.yaml
kubectl get pods

# 5. When done, switch back to Supervisor
./ns1-login

# 6. Delete the cluster
./vks1-delete
```

### Workflow 2: Install and Use Cert Manager

```bash
# 1. Create a VKS cluster
./ns1-login
./vks1-create
./vks1-login

# 2. Install cert-manager
./ns1-login
./vks1-install-cert-manager

# 3. Login to cluster and use cert-manager
./vks1-login
kubectl get all -n cert-manager

# 4. When done, delete cert-manager
./ns1-login
./vks1-delete-cert-manager
```

### Workflow 3: Refresh Expired Context

```bash
# If you see "401 Unauthorized" error:
./select-context
# Choose option 2: Refresh the current context

# Or simply re-login:
./ns1-login
./vks1-login
```

### Workflow 4: Just Explore the Supervisor

```bash
# 1. Login to Supervisor
./ns1-login

# 2. Explore with kubectl
kubectl get nodes
kubectl get namespaces
kubectl get storageclass
kubectl get cluster

# 3. View cluster details
kubectl describe cluster vks1
```

---

## Troubleshooting

### Connection Issues

**Error: "Not connected to Supervisor"**
- Run: `./ns1-login` first to establish the connection
- Verify the Supervisor IP in `vks-lab.conf`

**Error: "VKS cluster not found"**
- Verify the cluster name in `vks-lab.conf`
- Check if the cluster has already been deleted
- Use `kubectl get cluster` to see available clusters

**Error: "SSH connection failed" or "Network unreachable"**
- Check if the Supervisor IP in `vks-lab.conf` is correct
- Verify you have network access to that IP
- Test connectivity: `ping 10.1.7.2`

---

### Authentication Issues

**Error: "401 Unauthorized"**
- Your context token has expired
- Run: `./select-context` and choose option 2 to refresh
- Or re-run: `./ns1-login` or `./vks1-login`

**Error: "Authentication failed"**
- Verify credentials in `vks-lab.conf`
- Confirm password hasn't changed
- Check network access to Supervisor IP

---

### VKS Cluster Issues

**Cluster not becoming Ready**
- VKS creation in nested environments takes 10-15 minutes
- Check resource availability: `kubectl describe cluster vks1`
- Monitor pod creation: `kubectl get pods -A`

**Cluster deletion hangs**
- Some resources may need manual cleanup
- Check for stuck finalizers: `kubectl get cluster vks1 -o yaml`
- Check events: `kubectl get events -A | grep vks1`

**Cert-manager installation timeout**
- Addon is still installing; check status later with:
  ```bash
  vcf addon install list --cluster-name vks1 -n ns1
  ```

---

### Script Issues

**Error: "command not found: vcf" or "command not found: kubectl"**
- These tools aren't installed or not in your PATH
- Verify installation: `which vcf` and `which kubectl`
- Install missing tools or contact lab administrator

**Error: "Configuration file not found"**
- You're not running scripts from the correct directory
- Run: `cd "/Users/charlesl/Documents/Development/2 node vks scripts"`

**Error: "Scripts are not executable"**
- Fix permissions:
  ```bash
  chmod +x ns1-login vks1-login vks1-create vks1-delete select-context
  chmod +x vks1-install-cert-manager vks1-delete-cert-manager vks-help
  ```

---

### Debug a Script

Run any script with verbose output to see what's happening:

```bash
bash -x ./ns1-login
```

This shows every command being executed, helpful for troubleshooting.

---

## Advanced Usage

### Creating Multiple Clusters

To create additional VKS clusters with different configurations:

1. Copy the config file:
   ```bash
   cp vks-lab.conf vks-lab-2.conf
   ```

2. Edit the new config:
   ```bash
   nano vks-lab-2.conf
   ```

3. Change cluster names and context names:
   ```bash
   CLUSTER_NAME="vks2"
   CLUSTER_CONTEXT="vks2-ctx"
   ```

4. Modify scripts to accept a config file parameter (or source the appropriate config):
   ```bash
   export CONFIG_FILE="vks-lab-2.conf"
   ./vks1-create  # (if modified to use $CONFIG_FILE)
   ```

### Customizing Cluster Resources

Edit `vks-lab.conf` to change:

- **Kubernetes version:**
  ```bash
  CLUSTER_KUBERNETES_VERSION="v1.36"
  ```

- **Control plane replicas:**
  ```bash
  CLUSTER_CONTROL_PLANE_REPLICAS=3
  ```

- **Worker node replicas:**
  ```bash
  CLUSTER_WORKER_REPLICAS=5
  ```

- **VM classes:**
  ```bash
  CLUSTER_VM_CLASS_CONTROL_PLANE="best-effort-small"
  CLUSTER_VM_CLASS_WORKER="best-effort-xlarge"
  ```

- **Storage:**
  ```bash
  CLUSTER_STORAGE_CLASS="different-storage"
  STORAGE_SIZE="100Gi"
  ```

---

## File Reference

### Main Documentation

| File | Purpose |
|------|---------|
| `README.md` | This comprehensive guide (start here) |
| `ARCHITECTURE.md` | System design, workflow diagrams, and context model |
| `SETUP_INSTRUCTIONS.md` | Step-by-step setup and first-time usage |
| `README_VKS_SCRIPTS.md` | Detailed script documentation and reference |

### Specialized Documentation

| File | Purpose |
|------|---------|
| `README_CERTIFICATE_FIX.md` | Guide for fixing expired storage quota certificates |
| `README_CERT_MANAGER_INSTALL.md` | Detailed cert-manager installation guide |
| `README_DELETE_CERT_MANAGER.md` | Detailed cert-manager deletion guide |

### Quick Reference

| File | Purpose |
|------|---------|
| `vks-help` | Quick reference guide (run: `./vks-help`) |
| `vks-lab.conf` | Configuration file for all scripts |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    vSphere Environment                        │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Supervisor Cluster (vCenter SSO: administrator)     │  │
│  │                                                      │  │
│  │  ┌─────────────────────────────────────────────┐   │  │
│  │  │ Supervisor Namespace (ns1)                 │   │  │
│  │  │  - Contains VKS clusters                   │   │  │
│  │  │  - Isolated resources                      │   │  │
│  │  │                                            │   │  │
│  │  │  ┌──────────────────────────────────┐     │   │  │
│  │  │  │ VKS Cluster (vks1)              │     │   │  │
│  │  │  │  - 1 Control Plane Node         │     │   │  │
│  │  │  │  - 2 Worker Nodes               │     │   │  │
│  │  │  │  - Full Kubernetes Environment  │     │   │  │
│  │  │  └──────────────────────────────────┘     │   │  │
│  │  │                                            │   │  │
│  │  └─────────────────────────────────────────────┘   │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Color Output Reference

Scripts use colored output for clarity:

- **[INFO]** - Green - Informational messages
- **[STEP]** - Blue - Progress steps
- **[WARN]** - Yellow - Warnings
- **[ERROR]** - Red - Errors (before exit)

---

## Security Note

⚠️ **For Lab Environments Only:** Passwords are stored in plain text in `vks-lab.conf` for convenience in learning environments.

**Never use this practice in production environments.** For production:
- Use SSH key-based authentication
- Use environment variables or secure vaults for credentials
- Use RBAC and service accounts instead of user passwords
- Implement proper secret management

---

## Getting Help

### View Quick Reference
```bash
./vks-help
```

### Read Full Documentation
```bash
cat README_VKS_SCRIPTS.md
```

### View Architecture & Workflow
```bash
cat ARCHITECTURE.md
```

### Debug a Script
```bash
bash -x ./ns1-login
```

---

## Learning Path

### Day 1: Get Familiar
1. Read `SETUP_INSTRUCTIONS.md`
2. Run `./vks-help`
3. Run `./ns1-login`
4. Explore: `kubectl get nodes`, `kubectl get namespaces`

### Day 2: Create Your First Cluster
1. `./ns1-login`
2. `./vks1-create` (wait for completion)
3. `./vks1-login`
4. Try: `kubectl get pods -A`

### Day 3: Deploy Applications
1. `./vks1-login`
2. `kubectl apply -f my-app.yaml`
3. Monitor your application

### Day 4: Advanced Topics
1. Read `ARCHITECTURE.md` to understand the design
2. Read `README_VKS_SCRIPTS.md` section on "Advanced Usage"
3. Customize `vks-lab.conf` for different cluster sizes

### Day 5: Cleanup
1. `./ns1-login`
2. `./vks1-delete`

---

## Version Information

- **Lab Environment:** VKS 3.6.2
- **vSphere:** 9.0.2
- **Kubernetes Version:** v1.35 (configurable)
- **Script Version:** 1.0
- **Last Updated:** May 2026

---

## Related Resources

- **Lab Manual:** TMM-2026-2-NODE-VKS-3.6.2-LAB-DEPLOYED
- **VMware VKS Documentation:** https://docs.vmware.com/
- **Kubernetes kubectl:** https://kubernetes.io/docs/reference/kubectl/
- **Cert-manager:** https://cert-manager.io/
- **VCF CLI:** Consult your VMware documentation
- **Broadcom KB 424055 (Storage Quota Certificates):** https://knowledge.broadcom.com/external/article/424055

---

## Support

For issues with the lab environment, consult the lab manual: **TMM-2026-2-NODE-VKS-3.6.2-LAB-DEPLOYED**

The scripts automate the manual procedures, but understanding the concepts in the lab guide is important for troubleshooting.

For script-specific issues:
1. Check `Troubleshooting` section above
2. Review the relevant README file
3. Run the script with `bash -x` for detailed output
4. Contact your lab administrator

---

**Created for the VMware VKS Learning Lab** | **For Lab Use Only**
