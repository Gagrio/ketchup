# 🍅 Ketchup - Kubernetes Config Collector

> **Catch up** on your cluster configurations! 🏃‍♂️💨

A blazingly fast 🦀 Rust-powered tool that collects and archives **all** Kubernetes cluster configurations for backup, analysis, and troubleshooting by support engineers.

## ✨ Features

🔐 **Secure & Explicit** - Requires explicit kubeconfig path (no magic auto-discovery)  
📦 **Comprehensive Collection** - Collects ALL cluster and namespaced resources dynamically  
🧹 **Smart Sanitization** - Resources sanitized by default for kubectl apply readiness  
🎯 **Selective Collection** - Opt-in flags for sensitive or high-volume resources  
🗂️ **Organized Structure** - Cluster and namespace resources in separate directories  
📊 **Detailed Summaries** - Complete metadata about what was collected  
🗜️ **Compressed Archives** - Creates `.tar.gz` archives for easy storage and sharing  
🐳 **Container Ready** - Perfect for containerized environments with SELinux support  
⚡ **Fast & Reliable** - Built with Rust for maximum performance and safety  

## 🚀 Quick Start

### Prerequisites

- ☸️ **Kubernetes cluster** with accessible kubeconfig
- 🐳 **Podman or Docker** installed
- 📁 **Write access** to output directory

### Quick Start with Container

The easiest way to use Ketchup is with the pre-built container image.

**Available Images:**
- `ghcr.io/gagrio/ketchup:latest` - Latest stable release
- `ghcr.io/gagrio/ketchup:2.0.0` - Specific version (e.g., 2.0.0)
- See all available versions: [GitHub Releases](https://github.com/gagrio/ketchup/releases)

```bash
# Basic collection from all namespaces (using latest)
podman run -v ~/.kube/config:/kubeconfig:ro,Z \
           -v /tmp:/tmp:Z \
           ghcr.io/gagrio/ketchup:latest \
           --kubeconfig /kubeconfig --output /tmp --verbose

# Collect with secrets and custom resources
podman run -v ~/.kube/config:/kubeconfig:ro,Z \
           -v /tmp:/tmp:Z \
           ghcr.io/gagrio/ketchup:latest \
           --kubeconfig /kubeconfig --output /tmp -s -C --verbose

# Collect from specific namespaces only
podman run -v ~/.kube/config:/kubeconfig:ro,Z \
           -v /tmp:/tmp:Z \
           ghcr.io/gagrio/ketchup:latest \
           --kubeconfig /kubeconfig --output /tmp -n "kube-system,default" --verbose

# Use a specific version (recommended for production)
podman run -v ~/.kube/config:/kubeconfig:ro,Z \
           -v /tmp:/tmp:Z \
           ghcr.io/gagrio/ketchup:2.0.0 \
           --kubeconfig /kubeconfig --output /tmp --verbose
```

**Note:** The `:Z` flag is required for SELinux systems (RHEL, CentOS, Fedora, SUSE). On non-SELinux systems, it's safely ignored.

### 🐳 Building Custom Container Image

If you need to build your own container image, a multi-stage Dockerfile using SUSE BCI images is included in the repository.

```bash
# Build the container image
podman build -t ketchup:custom .

# Run your custom image
podman run -v ~/.kube/config:/kubeconfig:ro,Z \
           -v /tmp:/tmp:Z \
           ketchup:custom \
           --kubeconfig /kubeconfig --output /tmp --verbose
```

**Notes:**
- **Container Images:**
  - `latest` tag: Points to the most recent stable release
  - Version tags: Pin to specific versions (e.g., `2.0.0`, `2.1.0`)
  - All releases: https://github.com/gagrio/ketchup/releases
- Custom builds are useful for testing unreleased features or modifications
- If your kubeconfig points to `localhost` or `127.0.0.1`, add `--net=host` to the run command
- **Recommendation:** Use version tags (e.g., `:2.0.0`) in production for reproducibility

### Installation from Source

If you want to build from source instead of using the container image:

**Prerequisites:**
- 🦀 **Rust 1.70+** (install via [rustup](https://rustup.rs/))

```bash
# Clone the repository
git clone https://github.com/gagrio/ketchup.git
cd ketchup

# Build the tool
cargo build --release

# Run it!
cargo run -- --kubeconfig ~/.kube/config --verbose
```

## 📖 Usage

### Basic Usage

```bash
# Collect from all namespaces with defaults
cargo run -- --kubeconfig ~/.kube/config

# Collect from specific namespaces
cargo run -- --kubeconfig ~/.kube/config --namespaces "kube-system,default"

# Verbose output with progress
cargo run -- --kubeconfig ~/.kube/config --verbose

# Debug output with HTTP traces
cargo run -- --kubeconfig ~/.kube/config --debug

# Include secrets (disabled by default for security)
cargo run -- --kubeconfig ~/.kube/config --include-secrets

# Include custom resources (may show API errors, can be ignored)
cargo run -- --kubeconfig ~/.kube/config --include-custom-resources

# Collect raw unsanitized resources
cargo run -- --kubeconfig ~/.kube/config --raw
```

### Command Line Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| `--kubeconfig` | `-k` | **Required** Path to kubeconfig file | - |
| `--namespaces` | `-n` | Comma-separated list of namespaces | All namespaces |
| `--output` | `-o` | Output directory for archives | `/tmp` |
| `--format` | `-f` | Output format: json, yaml, or both | `yaml` |
| `--compression` | `-c` | Compression: compressed, uncompressed, or both | `compressed` |
| `--include-secrets` | `-s` | Include Secrets (disabled by default for security) | `false` |
| `--include-custom-resources` | `-C` | Include CRD instances (disabled by default) | `false` |
| `--include-events` | `-E` | Include Events (disabled by default, high volume) | `false` |
| `--include-replicasets` | `-R` | Include ReplicaSets (disabled by default) | `false` |
| `--include-endpoints` | `-P` | Include Endpoints/EndpointSlices (disabled by default) | `false` |
| `--include-leases` | `-L` | Include Leases (disabled by default) | `false` |
| `--crds` | - | Collect specific CRD instances only (comma-separated) | - |
| `--raw` | `-r` | Skip sanitization, collect raw resources | `false` |
| `--verbose` | `-v` | Verbose logging (progress and summaries) | `false` |
| `--debug` | `-d` | Debug logging (HTTP requests and traces) | `false` |
| `--help` | `-h` | Show help message | - |

## 📦 What Gets Collected?

### Core Resources (Always Collected)

**Cluster-Scoped Resources:**
- ✅ Nodes - Cluster infrastructure
- ✅ Namespaces - Namespace definitions
- ✅ PersistentVolumes - Storage resources
- ✅ StorageClasses - Storage types
- ✅ ClusterRoles - Cluster-wide permissions
- ✅ ClusterRoleBindings - Cluster permission assignments
- ✅ CustomResourceDefinitions - CRD definitions

**Namespaced Resources:**
- ✅ Pods - Running workloads
- ✅ Services - Network services
- ✅ Deployments - Deployment configurations
- ✅ StatefulSets - Stateful workloads
- ✅ DaemonSets - Node-level workloads
- ✅ Jobs - Batch jobs
- ✅ CronJobs - Scheduled jobs
- ✅ ConfigMaps - Configuration data
- ✅ PersistentVolumeClaims - Storage claims
- ✅ Ingresses - Ingress rules
- ✅ NetworkPolicies - Network policies
- ✅ ServiceAccounts - Service identities
- ✅ Roles - Namespace permissions
- ✅ RoleBindings - Permission assignments

### Optional Resources (Opt-In with Flags)

| Resource | Flag | Why Opt-In? |
|----------|------|-------------|
| **Secrets** | `-s, --include-secrets` | Contains sensitive data (passwords, tokens, certificates) |
| **Custom Resources** | `-C, --include-custom-resources` | Can be large, may cause API errors in resource-constrained clusters |
| **Specific CRDs** | `--crds <list>` | Collect only specified CRD instances |
| **Events** | `-E, --include-events` | High volume, temporary data, typically not useful for config analysis |
| **ReplicaSets** | `-R, --include-replicasets` | Auto-created by Deployments, redundant information |
| **Endpoints** | `-P, --include-endpoints` | Auto-managed by Services, redundant information |
| **Leases** | `-L, --include-leases` | High churn leader election data, not configuration |

## 📁 Output Structure

Ketchup creates organized, timestamped output with cluster and namespace separation:

```
/tmp/ketchup-2025-11-12-14-30-00/
├── 📄 collection-summary.yaml          # Complete collection metadata
├── 📂 cluster/                          # Cluster-scoped resources
│   ├── nodes/
│   │   ├── node-1.yaml
│   │   └── node-2.yaml
│   ├── namespaces/
│   │   ├── default.yaml
│   │   └── kube-system.yaml
│   ├── persistentvolumes/
│   ├── storageclasses/
│   ├── clusterroles/
│   └── clusterrolebindings/
├── 📂 default/                          # Namespace: default
│   ├── pods/
│   ├── services/
│   ├── deployments/
│   ├── configmaps/
│   └── ...
└── 📂 kube-system/                      # Namespace: kube-system
    ├── pods/
    ├── services/
    ├── deployments/
    └── ...

# Plus compressed archive:
/tmp/ketchup-2025-11-12-14-30-00.tar.gz  🗜️
```

### Collection Summary Example

```yaml
collection_info:
  timestamp: "2025-11-12T14:30:00Z"
  tool: "ketchup"
  version: "2.0.0"
  sanitized: true
  optional_resources_included:
    - secrets
    - custom_resources

cluster_summary:
  total_namespaces: 5
  total_cluster_resources: 42
  total_namespaced_resources: 387
  total_resources: 429
  resource_type_counts:
    Node: 3
    Pod: 156
    Service: 48
    Deployment: 23
    # ... etc

cluster_resources:
  total_resources: 42
  resource_types:
    Node: 3
    PersistentVolume: 5
    StorageClass: 4
    # ... etc

namespace_details:
  kube-system:
    total_resources: 89
    resource_types:
      Pod: 34
      Service: 12
      # ... etc
```

## 🧹 Resource Sanitization

By default, Ketchup **sanitizes** all resources to make them ready for `kubectl apply`. This removes:

- `metadata.uid`
- `metadata.resourceVersion`
- `metadata.selfLink`
- `metadata.creationTimestamp`
- `metadata.generation`
- `metadata.managedFields`
- `status` (entire section)

**Why?** Support engineers can directly apply these resources to recreate cluster state without manual cleanup.

**To collect raw unsanitized resources:** Use the `--raw` or `-r` flag.

## 💡 Common Use Cases

### Support Engineers: Complete Cluster State
```bash
# Collect everything for troubleshooting (including secrets)
ketchup --kubeconfig customer.kubeconfig -s -C --verbose
```

### Backup Without Secrets
```bash
# Safe backup without sensitive data
ketchup --kubeconfig ~/.kube/config --compression both
```

### Specific Namespace Analysis
```bash
# Focus on production namespace with custom resources
ketchup --kubeconfig ~/.kube/config -n production -C --verbose
```

### Raw Resources for Comparison
```bash
# Get unmodified resources with all metadata
ketchup --kubeconfig ~/.kube/config --raw --format both
```

### Specific CRD Collection
```bash
# Only collect specific custom resources
ketchup --kubeconfig ~/.kube/config --crds "mycrd.example.com,anothercrd.io"
```

## 🔧 Development

### Building from Source

```bash
# Debug build
cargo build

# Release build (optimized)
cargo build --release

# Run tests
cargo test

# Format code
cargo fmt

# Check for issues
cargo clippy
```

### Project Structure

```
src/
├── main.rs          # 🚪 CLI interface and orchestration
├── k8s.rs           # ☸️ Dynamic Kubernetes resource collection
└── output.rs        # 📁 File output and archive management
```

## 🐛 Troubleshooting

### Common Issues

**🚫 "Failed to load kubeconfig"**
- Check that the kubeconfig file exists and is readable
- Verify the file format is valid YAML
- Ensure you have network access to the cluster

**📁 "Permission denied" on output**
- Make sure the output directory is writable
- Try using `/tmp` as output directory
- Check file permissions with `ls -la`

**☸️ "Failed to connect to cluster"**
- Verify cluster is accessible: `kubectl cluster-info`
- Check if kubeconfig context is correct
- Ensure cluster certificates are valid

**⚠️ API errors when collecting custom resources**
- This is normal in resource-constrained clusters
- Use `--verbose` to see which resources cause errors
- Errors can be safely ignored - collection will continue
- To skip custom resources entirely, don't use `-C` flag

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

This project is part of the SUSE Support Material collection.

## 🙏 Acknowledgments

- 🦀 Built with **Rust** for performance and safety
- ☸️ Uses **kube-rs** with dynamic discovery for comprehensive resource collection
- 🎨 Designed for support engineers who need complete cluster visibility
- ☕ Powered by lots of coffee and determination

---

**Made with ❤️ and 🦀 by the SUSE Support Team**

*Catch up on your cluster configs with Ketchup!* 🍅✨
