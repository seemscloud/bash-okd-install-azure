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
  --scopes /subscriptions/XXXX-XXXXX
```

#### Get SP ID

```bash
az ad sp show \
  --id "XXXX-XXXXX" \
  --query id \
  --output tsv
```

#### Role Assignment to SP

```bash
az role assignment create \
  --role "User Access Administrator" \
  --assignee-object-id 4f0af042-ead4-4e4e-8a0e-0cee862dee1f
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
