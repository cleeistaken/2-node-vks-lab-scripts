# VKS Lab Automation Scripts - Complete Documentation

A comprehensive collection of shell scripts that automate common tasks in the VMware vSphere Kubernetes Services (VKS) learning lab environment. These scripts streamline repetitive tasks and help students focus on learning VKS concepts rather than typing commands.

**Lab Environment:** TMM-2026-2-NODE-VKS-3.6.2-LAB-DEPLOYED | **Kubernetes Version:** v1.35 | **vSphere:** 9.0.2

---

## 🚀 Quick Start (30 seconds)

```bash
cd "/Users/charlesl/Documents/Development/2 node vks scripts"
./ns1-login                      # Login to Supervisor
./ns1-create-vks1-1.35          # Create a VKS cluster with v1.35
./ns1-login-vks1                # Login to VKS cluster
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

1. **Test connectivity:** `./ns1-login`
2. **Create your first cluster:** `./ns1-create-vks1-1.35`
3. **Login to cluster:** `./ns1-login-vks1`

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
│   └── vks-lab.conf                              # ⚙️  Central config (passwords, IPs, names)
│
├── Main Scripts (Run these)
│   ├── ns1-login                                 # 🔐 Login to Supervisor namespace
│   ├── ns1-login-vks1                            # 🔐 Login to VKS cluster
│   ├── ns1-check-namespace                       # 🔍 Check Supervisor namespace configuration
│   ├── ns1-create-vks1-1.35                      # ✨ Create/upgrade VKS cluster to v1.35 (v3.6.0)
│   ├── ns1-create-vks1-1.34                      # ✨ Create/upgrade VKS cluster to v1.34 (v3.5.0)
│   ├── ns1-create-vks1-1.33                      # ✨ Create/upgrade VKS cluster to v1.33 (v3.4.0)
│   ├── ns1-delete-vks1                           # 🗑️  Delete VKS cluster (with context cleanup)
│   ├── ns1-delete-contexts                       # 🗑️  Delete all VCF and kubectl contexts
│   ├── ns1-check-vks1 [--watch]                  # 🔍 Check VKS cluster readiness (with optional watch)
│   ├── ns1-create-cert-manager                   # 📦 Install cert-manager addon
│   ├── ns1-check-cert-manager                    # 🔍 Check cert-manager status
│   └── ns1-delete-cert-manager-vks1              # 📦 Remove cert-manager addon
│
└── Documentation (Read this)
    └── README.md                                  # 📖 Complete documentation (this file)
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

### ns1-login-vks1

**Login to the VKS cluster**

```bash
./ns1-login-vks1
```

**What it does:**
1. Creates a VKS cluster context (if it doesn't exist)
2. Selects the VKS cluster context
3. Validates the connection with retry logic (handles upgrades)
4. Displays cluster nodes

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- VKS cluster must be in Ready state
- Cluster must exist

**Resilience:**
- Automatically retries connection validation up to 60 seconds
- Handles temporary cluster unavailability during upgrades
- No manual retry needed if cluster is upgrading

---

### ns1-check-namespace

**Check Supervisor namespace configuration and readiness**

```bash
./ns1-check-namespace
```

**What it does:**
1. Verifies Supervisor context is active
2. Checks if namespace exists
3. Verifies storage classes are available
4. Verifies VM classes (control plane and worker) are available
5. Displays namespace labels and annotations
6. Checks content library configuration (optional)
7. Checks network policies
8. Verifies RBAC configuration
9. Provides readiness summary

**Prerequisites:**
- Must be logged into the Supervisor: `./ns1-login`
- Supervisor namespace must exist

**Output:**
- Namespace configuration status
- Available storage classes
- Available VM classes
- RBAC configuration
- Overall readiness assessment
- Guidance for creating VKS clusters

**Use Cases:**
- Verify namespace is properly configured before creating clusters
- Troubleshoot missing storage or VM classes
- Validate lab environment setup

---

### ns1-create-vks1-1.35

**Create a new VKS cluster with Kubernetes v1.35 (builtin-generic-v3.6.0)**

```bash
./ns1-create-vks1-1.35
```

**What it does:**
1. Verifies Supervisor context is active
2. Checks if a cluster already exists (detects upgrade scenarios)
3. Validates cluster readiness for upgrade (if upgrading)
4. Generates a VKS cluster manifest for v1.35 with builtin-generic-v3.6.0
5. Applies the manifest to create or upgrade the cluster
6. Monitors cluster creation/upgrade progress
7. Displays configuration summary

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- Supervisor namespace must exist and be configured
- Storage policies must be configured in the namespace

**Upgrade Support:**
- If a cluster with the same name exists, this script detects it and performs an upgrade
- Supports upgrading from v1.33 (v3.4.0), v1.34 (v3.5.0), or fresh creation
- Displays "UPGRADE SCENARIO DETECTED" when upgrading an existing cluster
- Prevents downgrades (v1.35 → v1.34) automatically
- **Prevents direct v1.33 → v1.35 upgrades** - requires intermediate v1.34 upgrade first

**Upgrade Constraints:**
- ❌ Direct v1.33 → v1.35 **NOT SUPPORTED** (must go through v1.34)
- ✅ v1.33 → v1.34 (allowed)
- ✅ v1.34 → v1.35 (allowed)
- ❌ Downgrade any version (e.g., v1.35 → v1.34) **NOT SUPPORTED**

**Expected Time:**
- ~10-15 minutes in nested environments (fresh creation)
- ~10-15 minutes for upgrades (may be faster depending on load)
- ~2-5 minutes in physical environments

**Output:**
- Creates `vks.yaml` manifest file
- Shows live progress of cluster creation or upgrade
- Press Ctrl+C to stop monitoring (cluster continues creating/upgrading)

---

### ns1-create-vks1-1.34

**Create a new VKS cluster with Kubernetes v1.34 (builtin-generic-v3.5.0)**

```bash
./ns1-create-vks1-1.34
```

**What it does:**
1. Verifies Supervisor context is active
2. Checks if a cluster already exists (detects upgrade scenarios)
3. Validates cluster readiness for upgrade (if upgrading)
4. Generates a VKS cluster manifest for v1.34 with builtin-generic-v3.5.0
5. Applies the manifest to create or upgrade the cluster
6. Monitors cluster creation/upgrade progress
7. Displays configuration summary

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- Supervisor namespace must exist and be configured
- Storage policies must be configured in the namespace

**Upgrade Support:**
- If a cluster with the same name exists, this script detects it and performs an upgrade
- Supports upgrading from v1.33 (v3.4.0) or fresh creation
- Displays "UPGRADE SCENARIO DETECTED" when upgrading an existing cluster
- Prevents downgrades automatically

**Expected Time:**
- ~10-15 minutes in nested environments (fresh creation)
- ~10-15 minutes for upgrades
- ~2-5 minutes in physical environments

**Output:**
- Creates `vks.yaml` manifest file
- Shows live progress of cluster creation or upgrade
- Press Ctrl+C to stop monitoring (cluster continues creating/upgrading)

---

### ns1-create-vks1-1.33

**Create a new VKS cluster with Kubernetes v1.33 (builtin-generic-v3.4.0)**

```bash
./ns1-create-vks1-1.33
```

**What it does:**
1. Verifies Supervisor context is active
2. Checks if a cluster already exists (detects upgrade scenarios)
3. Validates cluster readiness for upgrade (if upgrading)
4. Generates a VKS cluster manifest for v1.33 with builtin-generic-v3.4.0
5. Applies the manifest to create or upgrade the cluster
6. Monitors cluster creation/upgrade progress
7. Displays configuration summary

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- Supervisor namespace must exist and be configured
- Storage policies must be configured in the namespace

**Upgrade Support:**
- If a cluster with the same name exists, this script detects it and performs an upgrade
- Designed as the base/oldest version for upgrade testing
- Can be used as a starting point to upgrade to v1.34 or v1.35
- Displays "UPGRADE SCENARIO DETECTED" when upgrading an existing cluster

**Expected Time:**
- ~10-15 minutes in nested environments (fresh creation)
- ~10-15 minutes for upgrades
- ~2-5 minutes in physical environments

**Output:**
- Creates `vks.yaml` manifest file
- Shows live progress of cluster creation or upgrade
- Press Ctrl+C to stop monitoring (cluster continues creating/upgrading)

---

### Upgrade Path and Version Matrix

Choose the appropriate script based on your needs:

| Starting Version | Script to Use | Target Version | Use Case |
|---|---|---|---|
| Fresh Start | `ns1-create-vks1-1.33` | v1.33 (v3.4.0) | Initial cluster, baseline for upgrades |
| v1.33 | `ns1-create-vks1-1.34` | v1.34 (v3.5.0) | Upgrade to intermediate version |
| v1.34 | `ns1-create-vks1-1.35` | v1.35 (v3.6.0) | Upgrade to latest version |
| v1.33 | ❌ Cannot use 1.35 directly | ❌ Not supported | Must upgrade through v1.34 first |
| Any Version | Same Script | Same Version | Idempotent - re-applying same version |

**Upgrade Workflow Example (v1.33 → v1.34 → v1.35):**

```bash
# Step 1: Create cluster with v1.33
./ns1-create-vks1-1.33
# Wait for cluster to be ready

# Step 2: Upgrade to v1.34
./ns1-create-vks1-1.34
# Script detects existing cluster and performs upgrade
# Wait for upgrade to complete

# Step 3: Upgrade to v1.35
./ns1-create-vks1-1.35
# Script detects existing cluster and performs upgrade
# Wait for upgrade to complete
```

---

### ns1-delete-vks1

**Delete a VKS cluster**

```bash
./ns1-delete-vks1
```

**What it does:**
1. Verifies Supervisor context is active
2. Confirms cluster exists
3. Prompts for confirmation before deletion
4. Sends delete command
5. Monitors deletion progress
6. Automatically cleans up VKS cluster contexts (kubectl and VCF)

**Prerequisites:**
- Must be logged into the Supervisor first: `./ns1-login`
- VKS cluster must exist
- User must confirm deletion

**Context Cleanup:**
- Automatically deletes kubectl context: `vks1-ctx:vks1`
- Automatically deletes VCF context: `vks1-ctx:vks1`
- Prevents connection issues when recreating cluster with same name

**Expected Time:**
- ~5-10 minutes for deletion to complete

---

### ns1-delete-contexts

**Delete all Supervisor and VKS contexts**

```bash
./ns1-delete-contexts
```

**What it does:**
1. Lists all VCF contexts
2. Lists all kubectl contexts
3. Prompts for confirmation
4. Optionally deletes the VKS cluster if it exists
5. Deletes all VCF contexts
6. Deletes all kubectl contexts
7. Verifies deletion was successful
8. Cleans up kubeconfig

**Features:**
- ✅ Lists contexts before deletion
- ✅ Optional VKS cluster deletion (prompted)
- ✅ Requires confirmation before any deletion
- ✅ Reports what was deleted
- ✅ Handles errors gracefully

**Prerequisites:**
- None (can be run anytime)

**Use Cases:**
- Clean slate before switching to different lab environment
- Troubleshoot context-related connection issues
- Full cleanup after labs complete
- Reset all contexts and clusters

**Typical Usage:**
```bash
./ns1-delete-contexts
# Lists contexts
# Asks: "Also delete the VKS cluster? (yes/no)"
# Deletes all contexts (VCF and kubectl)
# Shows summary of deleted items
```

---

### ns1-check-vks1

**Check VKS cluster readiness status (with optional watch mode)**

```bash
# Single check
./ns1-check-vks1

# Continuous watch mode (20 minute timeout)
./ns1-check-vks1 --watch
```

**Flags:**
- `--watch` - Continuously monitor cluster until ready (timeout: 20 min, Ctrl+C to stop)

**What it does:**
1. Verifies Supervisor context is active
2. Checks cluster existence
3. Displays cluster condition status:
   - Available (Ready)
   - TopologyReconciled
   - AddonsReconciled
   - Paused status
4. Shows node readiness
5. Verifies critical addon pods (Antrea, vSphere-CSI)
6. Determines overall cluster readiness
7. **[Watch mode only]** Continuously updates status until ready or timeout

**Prerequisites:**
- Must be logged into the Supervisor: `./ns1-login`
- VKS cluster must exist

**Exit Codes:**
- 0: ✅ Cluster is ready
- 1: ⚠️ Cluster partially ready (addons reconciling) or watch timeout
- 2: ❓ Unknown status

**Watch Mode Behavior:**
- Updates display every 5 seconds
- Shows: Available, TopologyReconciled, AddonsReconciled, Phase, Elapsed time
- Waits up to 20 minutes (1200 seconds)
- Exits immediately when cluster becomes ready
- User can press Ctrl+C to stop anytime
- Provides diagnostic commands on timeout

**Use Cases:**
- Monitor cluster creation/upgrade progress
- Verify cluster is ready before deployments
- Troubleshoot cluster issues
- Automate operational checks
- Real-time monitoring during creation/upgrades

**Diagnostic Commands:**
Script includes 8 diagnostic commands for troubleshooting addon issues:
- Check detailed cluster status
- View all cluster conditions
- Check AddonsReconciled condition specifically
- Check addon resources
- Check Antrea (CNI) plugin status
- Check vSphere-CSI (Storage) status
- Check for reconciliation events
- Monitor addon reconciliation in real-time

---

### ns1-create-cert-manager

**Install cert-manager addon on VKS cluster**

```bash
./ns1-create-cert-manager
```

**What it does:**
1. Verifies Supervisor context is valid
2. Verifies VKS cluster exists and is fully ready
3. **Checks if cert-manager is already installed** (skips if exists)
4. Fetches the latest available cert-manager version
5. Generates the data values configuration
6. Installs the cert-manager addon
7. Monitors the installation progress
8. Verifies the installation in the cluster

**Prerequisites:**
- Must be logged into the Supervisor: `./ns1-login`
- VKS cluster must exist: `./ns1-create-vks1-*`
- VKS cluster must be fully ready (Available=True, AddonsReconciled=True)
- Cluster must not already have cert-manager (script will skip if it does)

**Cluster Readiness Check:**
- ✅ Validates Available condition is True
- ✅ Validates AddonsReconciled condition is True
- ⚠️ Fails if cluster not fully ready (suggests waiting)

**Existing Installation Handling:**
- ✅ Detects if cert-manager is already installed
- ✅ Skips installation if addon exists
- ✅ Shows current addon status
- ✅ Suggests using `ns1-check-cert-manager` for detailed status

**Typical Installation Time:**
- ~2-5 minutes (if installing)
- ~5 seconds (if already installed)

---

### ns1-check-cert-manager

**Check cert-manager status and health**

```bash
./ns1-check-cert-manager
```

**What it does:**
1. Verifies Supervisor context is valid
2. Verifies VKS cluster exists
3. Checks if cert-manager addon is installed
4. Displays addon installation status
5. Verifies cert-manager pods are running
6. Shows services and pod health summary

**Prerequisites:**
- Must be logged into the Supervisor: `./ns1-login`
- VKS cluster must exist

**Output:**
- Addon installation status (Ready/not installed)
- Pod deployment status
- Service configuration
- Pod health summary

---

### ns1-delete-cert-manager-vks1

**Remove cert-manager addon from VKS cluster**

```bash
./ns1-delete-cert-manager-vks1
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

### Workflow 1: Create and Access a VKS Cluster (Latest v1.35)

```bash
# 1. Login to Supervisor
./ns1-login

# 2. Create a new VKS cluster with v1.35 (10-15 minutes)
./ns1-create-vks1-1.35

# 3. Monitor cluster creation with watch mode (in separate terminal optional)
./ns1-check-vks1 --watch
# Wait until cluster is ready or timeout

# 4. Single status check (alternative to watch)
./ns1-check-vks1

# 5. When cluster is ready, login to it
./ns1-login-vks1

# 6. Deploy applications
kubectl apply -f my-app.yaml
kubectl get pods

# 7. When done, switch back to Supervisor
./ns1-login

# 8. Delete the cluster
./ns1-delete-vks1
# Automatically cleans up contexts

# 9. (Optional) Clean up all contexts
./ns1-delete-contexts
```

### Workflow 2: Test Kubernetes Version Upgrades (v1.33 → v1.34 → v1.35)

```bash
# Phase 1: Create baseline cluster with v1.33
./ns1-login
./ns1-create-vks1-1.33

# Monitor creation with watch mode (alternative: run in separate terminal)
./ns1-check-vks1 --watch
# Wait for cluster to be ready (10-15 minutes)

# Phase 2: Upgrade to v1.34 (REQUIRED intermediate step)
./ns1-create-vks1-1.34
# Script detects existing cluster and performs upgrade
# Wait for upgrade to complete (10-15 minutes)

# Phase 3: Upgrade to v1.35 (final version)
./ns1-create-vks1-1.35
# Script detects existing cluster and performs upgrade
# Wait for upgrade to complete

# Monitor final status
./ns1-check-vks1 --watch
# Wait for cluster to fully reconcile

# Access and test
./ns1-login-vks1
kubectl get nodes
```

**Important:** Direct upgrade from v1.33 to v1.35 is not supported. You must upgrade through v1.34 first.

### Workflow 3: Install and Use Cert Manager

```bash
# 1. Create a VKS cluster
./ns1-login
./ns1-create-vks1-1.35

# 2. Monitor creation (optional, in separate terminal)
./ns1-check-vks1 --watch

# 3. Check namespace configuration (optional verification)
./ns1-check-namespace

# 4. Install cert-manager (waits for cluster to be fully ready)
./ns1-create-cert-manager

# 5. Check cert-manager status and health
./ns1-check-cert-manager

# 6. Login to cluster and use cert-manager
./ns1-login-vks1
kubectl get all -n cert-manager
kubectl get certificates -A

# 7. When done, delete cert-manager
./ns1-login
./ns1-delete-cert-manager-vks1

# 8. Verify deletion
./ns1-check-cert-manager
```

### Workflow 4: Just Explore the Supervisor

```bash
# 1. Login to Supervisor
./ns1-login

# 2. Explore with kubectl
kubectl get nodes
kubectl get namespaces
kubectl get storageclass
kubectl get clusters

# 3. View cluster details
kubectl describe cluster vks1
```

---

## Script Flags and Options

### Available Script Flags

| Script | Flag | Description | Example |
|--------|------|-------------|---------|
| `ns1-check-vks1` | `--watch` | Watch mode: monitor until ready or timeout (20 min) | `./ns1-check-vks1 --watch` |
| `ns1-delete-contexts` | N/A | Includes interactive prompts for deletion | `./ns1-delete-contexts` |

### Flag Details

#### `ns1-check-vks1 --watch`

Continuously monitors cluster readiness with automatic updates every 5 seconds:

```bash
./ns1-check-vks1 --watch
```

**Behavior:**
- Shows real-time status: Available, TopologyReconciled, AddonsReconciled
- Updates every 5 seconds with timestamp
- Shows elapsed time vs 20-minute timeout
- Exits automatically when cluster is ready
- User can press Ctrl+C to stop anytime
- Provides diagnostic commands if timeout reached

**Display Example:**
```
[14:32:15] A:True R:False AD:False | Phase:Provisioned | Elapsed:  15s/1200s
[14:32:20] A:True R:True  AD:False | Phase:Provisioned | Elapsed:  20s/1200s
[14:32:25] A:True R:True  AD:True  | Phase:Provisioned | Elapsed:  25s/1200s

✅ CLUSTER IS READY!
Cluster became ready after 25 seconds
```

---

## Cluster Readiness Validation

The cluster creation and cert-manager scripts perform comprehensive readiness checks:

### Before Cluster Upgrade

The `ns1-create-vks1-*` scripts validate:
- ✅ Cluster exists
- ✅ Cluster is Available (infrastructure ready)
- ✅ Cluster is TopologyReconciled (cluster objects reconciled)
- ✅ Cluster AddonsReconciled (CNI, storage, all addons ready)
- ✅ Not in Paused state
- ✅ No downgrade attempts (prevents data loss)
- ✅ Upgrade path is supported (v1.33→v1.35 blocked, must use v1.34 intermediate)

### Before Cert-Manager Installation

The `ns1-create-cert-manager` script validates:
- ✅ Supervisor context is valid
- ✅ Cluster exists
- ✅ Cluster is Available (Available=True)
- ✅ Cluster AddonsReconciled (AddonsReconciled=True)
- ✅ Cert-manager not already installed

If validation fails, script provides clear guidance on what to do next.

---

## Troubleshooting

### Connection Issues

**Error: "Not connected to Supervisor"**
- Run: `./ns1-login` first to establish the connection
- Verify the Supervisor IP in `vks-lab.conf`

**Error: "VKS cluster not found"**
- Verify the cluster name in `vks-lab.conf`
- Check if the cluster has already been deleted
- Use `kubectl get clusters` to see available clusters

**Error: "SSH connection failed" or "Network unreachable"**
- Check if the Supervisor IP in `vks-lab.conf` is correct
- Verify you have network access to that IP
- Test connectivity: `ping 10.1.7.2`

---

### Authentication Issues

**Error: "401 Unauthorized"**
- Your context token has expired
- Run: `./ns1-login` or `./ns1-login-vks1` to refresh

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
  ./ns1-check-cert-manager
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
  chmod +x ns1-login ns1-login-vks1 ns1-create-vks1-* ns1-delete-vks1 ns1-check-vks1
  chmod +x ns1-create-cert-manager ns1-check-cert-manager ns1-delete-cert-manager-vks1
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
   ./ns1-create-vks1-1.35  # (if modified to use $CONFIG_FILE)
   ```

### Customizing Cluster Resources

Edit `vks-lab.conf` to change:

- **Kubernetes version:**
  ```bash
  CLUSTER_KUBERNETES_VERSION="v1.35"
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

### Read Full Documentation
```bash
cat README.md
```

### Debug a Script
```bash
bash -x ./ns1-login
```

---

## Learning Path

### Day 1: Get Familiar
1. Read this README.md
2. Run `./ns1-login`
3. Explore: `kubectl get nodes`, `kubectl get namespaces`

### Day 2: Create Your First Cluster
1. `./ns1-login`
2. `./ns1-create-vks1-1.35` (wait for completion)
3. `./ns1-login-vks1`
4. Try: `kubectl get pods -A`

### Day 3: Deploy Applications
1. `./ns1-login-vks1`
2. `kubectl apply -f my-app.yaml`
3. Monitor your application

### Day 4: Advanced Topics
1. Test cluster upgrades: `ns1-create-vks1-1.34` then `ns1-create-vks1-1.35`
2. Install cert-manager: `ns1-create-cert-manager`
3. Customize `vks-lab.conf` for different cluster sizes

### Day 5: Cleanup
1. `./ns1-login`
2. `./ns1-delete-vks1`

---

## Version Information

- **Lab Environment:** VKS 3.6.2
- **vSphere:** 9.0.2
- **Kubernetes Version:** v1.35 (configurable)
- **Script Version:** 2.0
- **Last Updated:** May 2026

---

## Related Resources

- **Lab Manual:** TMM-2026-2-NODE-VKS-3.6.2-LAB-DEPLOYED
- **VMware VKS Documentation:** https://docs.vmware.com/
- **Kubernetes kubectl:** https://kubernetes.io/docs/reference/kubectl/
- **Cert-manager:** https://cert-manager.io/
- **VCF CLI:** Consult your VMware documentation

---

## Support

For issues with the lab environment, consult the lab manual: **TMM-2026-2-NODE-VKS-3.6.2-LAB-DEPLOYED**

The scripts automate the manual procedures, but understanding the concepts in the lab guide is important for troubleshooting.

For script-specific issues:
1. Check `Troubleshooting` section above
2. Run the script with `bash -x` for detailed output
3. Contact your lab administrator

---

**Created for the VMware VKS Learning Lab** | **For Lab Use Only**
