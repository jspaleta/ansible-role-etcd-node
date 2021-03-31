# Etcd Node Ansible Role
Configures an etcd node, either as a standalone node or as part of a cluster.
For a cluster, the role must be executed for all cluster members in a single play to ensure all nodes are located.

 
SSL is enabled by default and requires certificates to be configured.
See the [SSL Configuration](#ssl-configuration) section for more information.

Based on the `jumperfly.etcd_node` role developed by jumperfly.  

Long term goal is to be able to support Sensu embedded etcd deployments as well as standalone etcd clusters.

## Installation Modes
Etcd can be installed in the following modes:
* Package - etcd is installed with the package manager
* Local service - etcd binaries are downloaded and configured as a service.
* Kubernetes pod - etcd is deployed as static pod, requires kubelet to be installed, eg. through the  ``jumperfly.kubelet`` role.

## Base Configuration
| Key | Description |
|-----|-------------|
| ``etcd_version``               | The version of etcd to install. |
| ``etcd_ansible_group``         | The Ansible group that defines all etcd nodes in the cluster, default ``etc_nodes``. Clustering is enabled if more than one host is found in the group. |
| ``etcd_ssl_enabled``           | Determines whether SSL is used (default ``yes``). See the [SSL Configuration](#ssl-configuration) section for more detail. |
| ``etcd_install_mode``          | Determines the installation method. Supported modes are ``package`` (default), ``service`` or ``pod``. |
| ``etcd_enable_service``        | Determines whether to start/enable the etcd service. Defaults to ``yes`` and cannot be modified for ``pod`` installation mode. |
| ``etcd_auto_recover_degraded`` | Determines whether to attempt to auto-recover if this cluster is degraded. This involves deleting and replacing the missing nodes. |

## Network Configuration
| Key | Description |
|-----|-------------|
| ``etcd_iface``      | The network interface used to listen for client requests. Defaults to the Ansible default as described by the ``default_ipv4`` fact. |
| ``etcd_peer_iface`` | The network interface used to listen for peer requests. Only applicable if running in a cluster. Defaults to the same as the ``etcd_iface``. |

## Installation Configuration
### Service Installation Configuration
| Key | Description |
|-----|-------------|
| ``etcd_archive_checksum`` | The checksum of the etcd archive. |
| ``etcd_base_name``        | The base name of etcd archive to download (excluding file extension). Default ``etcd-v{{ etcd_version }}-linux-amd64`` |

### Kubernetes Pod Installation Configuration
| Key | Description |
|-----|-------------|
| ``kube_docker_registry`` | The docker registry used to pull the etcd image. Default ``k8s.gcr.io``. |
| ``etcd_docker_revision`` | The revision of the docker image to pull, appended to the version. |
| ``etcd_cpulimit``        | The CPU limit applied to the etcd pod. |

## SSL Configuration
Keys and certificates are loaded from the locations shown below.
The ``jumperfly.ssl_cert`` role can be used to generate them if required. For example, the following will work with the default configuration:
```
  - role: jumperfly.ssl_cert
    ssl_cert_type: peer
    ssl_cert_file_base_dir: /etc/etcd
    ssl_cert_name: "{{ ansible_hostname }}"
    ssl_cert_ca_delegate_name: etcd-ca
    ssl_cert_subject_common_name: "{{ ansible_hostname }}"
    ssl_cert_hosts:
      - "{{ ansible_hostname }}"
      - "{{ ansible_default_ipv4.address }}"
```

| Key | Description |
|-----|-------------|
| ``etcd_private_key`` | The location of the etcd private key. Defaults to ``/etc/etcd/{{ ansible_hostname }}-key.pem``. |
| ``etcd_cert``        | The location of the etcd certificate. Defaults to ``/etc/etcd/{{ ansible_hostname }}.pem``. |
| ``etcd_ca_cert``     | The location of the etcd CA certificate chain. Defaults to ``/etc/etcd/{{ ansible_hostname }}-chain.pem`` |
