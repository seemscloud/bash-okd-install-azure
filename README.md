# OKD Install on Azure

## Requirements

### Subscription Roles

|   Subscription Roles      |
|---------------------------|
| User Access Administrator |
| Owner                     |

### Resource Quotas

| Type      | Type             | Count            | Cores            |
|-----------|------------------|------------------|------------------|
| Bootstrap | Same like Master | Same like Master | Same like Master |
| Master    | Compute          | 3                | 8                |
| Worker    | General          | 8                | 8                |

----

## Azure Resources

### Application

```bash
az ad sp create-for-rbac \
  --role Contributor \
  --name OKD \
  --scopes /subscriptions/Aaaa-Aaaa
```

```json
{
  "appId": "Bbbbb-Bbbbb",
  "displayName": "App-Name",
  "password": "Ccccc-Ccccc",
  "tenant": "Ddddd-Ddddd"
}
```

#### Get SP ID

```bash
az ad sp show \
  --id "Bbbbb-Bbbbb" \
  --query id \
  --output tsv
```

```
Eeeee-Eeeee
```

#### Role Assignment to SP

```bash
az role assignment create \
  --role "User Access Administrator" \
  --assignee-object-id Eeeee-Eeeee
```

----

## Install

### Create `install-config.yaml` file
```bash
./openshift-install create install-config \
  --dir install_dir/ \
  --log-level=debug
```

### Install

```yaml
apiVersion: v1
metadata:
  name: prod
baseDomain: mydomain.local
controlPlane:
  name: master
  replicas: 3
  hyperthreading: Enabled
  architecture: amd64
  platform:
    azure:
      encryptionAtHost: false
      ultraSSDCapability: Enabled
      type: Standard_D8ds_v5
      osDisk:
        diskSizeGB: 150
        diskType: Premium_LRS
compute:
  - name: worker
    replicas: 3
    hyperthreading: Enabled
    architecture: amd64
    platform:
      azure:
        encryptionAtHost: false
        ultraSSDCapability: Enabled
        type: Standard_D8ds_v5
        osDisk:
          diskSizeGB: 150
          diskType: Premium_LRS
        zones:
          - "1"
          - "2"
          - "3"
networking:
  networkType: OVNKubernetes
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  serviceNetwork:
    - 172.30.0.0/16
  machineNetwork:
    - cidr: 10.100.0.0/28
    - cidr: 10.100.2.0/23
platform:
  azure:
    cloudName: AzurePublicCloud
    region: westeurope
    resourceGroupName: prod-okd
    baseDomainResourceGroupName: prod-okd-dns
    networkResourceGroupName: prod-okd-network
    virtualNetwork: prod-okd
    controlPlaneSubnet: master
    computeSubnet: worker
    outboundType: Loadbalancer
    defaultMachinePlatform:
      ultraSSDCapability: Enabled
publish: Internal
pullSecret: '{"auths":{"fake":{"auth": "bar"}}}'
sshKey: ssh-rsa AAAAB3NzaC1y
```

```bash
unset OPENSHIFT_INSTALL_OS_IMAGE_OVERRIDE
export OPENSHIFT_INSTALL_OS_IMAGE_OVERRIDE="https://XXXXXX.blob.core.windows.net/XXXXXX/fedora-coreos-azure.x86_64.vhd

echo "${OPENSHIFT_INSTALL_OS_IMAGE_OVERRIDE}"
```

```bash
./openshift-install create cluster \
  --dir install_dir/ \
  --log-level=debug
```
