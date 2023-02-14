# BindPlane OP Chart

## Installing the Chart

### Add Repo

```bash
helm repo add bindplane \
    https://observiq.github.io/bindplane-op-helm

helm repo update
helm search repo
```

### Create Secret

BindPlane OP requires a secret for configuring sensative options. This secret should be managed outside of helm with your preferred secret management solution.

The secret should have the following keys:
- `username`: Basic auth username to use for the default admin user
- `password`: Basic auth password to use for the default admin user
- `secret_key`: Random UUIDv4 to use for authenticating OpAMP clients
- `sessions_secret`: Random UUIDv4 used to derive web interface session tokens

Example: Create secret with `kubectl`:

```shell
kubectl -n default create secret generic bindplane \
  --from-literal=username=myuser \
  --from-literal=password=mypassword \
  --from-literal=secret_key=353753ca-ae48-40f9-9588-28cf86430910 \
  --from-literal=sessions_secret=d9425db6-c4ee-4769-9c1f-a66987679e90
```

### Install Chart

```bash
helm upgrade --install bindplane bindplane/bindplane
```

## Configuration

The following table lists the configurable parameters of the BindPlaneOP chart.                                                                                                                                                   
| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `backend.type` | Backend to use for persistent storage. When set to `bbolt`, bindplane will be deployed as a single pod StatefulSet using a persistent volume. | `bbolt` |
| `backend.bbolt.volumeSize` | Persistent volume size. | `10Gi` |
| `image.repository` | Container repository to use. | `observiq/bindplane` |
| `image.tag`        | Image tag to use. | Derived from chart version. |
| `config.server_url` | The URI used by clients to communicate with bindplane-op's REST API. | `http://bindplane-op:3001` |
| `config.remote_url` | The URI used by clients to communicate with bindplane-op's OpAMP interface. | `ws://bindplane-op:3001` |
| `config.secret`     | See the [create secret](#create-secret) section. | `bindplane` |
| `resources.requests.cpu`    | [Requested CPU](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | `250m` |
| `resources.requests.memory` | [Requested Memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | `250Mi` |
| `resources.limits.cpu`      | [CPU limit](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | Not set (burstable qos) |
| `resources.memory.cpu`      | [Memory limit](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) | `500Mi` |

Detailed documentation for BindPlane OP's configuration can be found [here](../../docs/configuration.md)

## Connectivity

BindPlane can be reached using the clusterIP service deployed by the chart. By default the service
name is `bindplane` and the port is `3001`.

**Web Interface**

You can connect to BindPlane from your workstation using port forwarding:

```bash
kubectl -n default port-forward svc/bindplane 3001
```

You should now be able to access http://localhost:3001

**Command Line**

You can connect to BindPlane from your workstation using port forwarding:

```bash
kubectl -n default port-forward svc/bindplane 3001
```

Create and use a profile:

```bash
bindplanectl profile set myprofile \
  --username <username> \
  --password <password> \
  --server-url http://localhost:3001

bindplanectl profile use myprofile
```

You should be able to issue commands: `bindplanectl get agent`

**Collectors**

Collectors running within the cluster can connect with the following OpAMP URI:

```
ws://bindplane.default.svc.cluster.local:3001/v1/opamp
```