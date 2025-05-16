# Mealie Helm Chart

This is a Helm chart for deploying [Mealie](https://github.com/mealie-recipes/mealie), a self-hosted recipe manager and meal planner.

It includes support for:

- Custom configuration via `values.yaml`
- Secret and config management
- Persistent storage
- Ingress and service configuration
- CNPG PostgreSQL secret referencing

---

## Installation

Add your chart repository (if remote) or install directly from local path:

```bash
helm upgrade --install mealie ./helm/mealie -f ./helm/mealie/values.yaml
```

## Configuration

You can configure the chart by editing `values.yaml`. Below is an example:

```yaml
image:
  repository: hkotel/mealie
  tag: v1.5.0
  pullPolicy: IfNotPresent

replicaCount: 1

postgres:
  fullnameOverride: cnpg-cluster

pvc:
  - name: mealie-data
    size: 10Gi
    storageClass: default

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: mealie.local
      paths:
        - path: /
          pathType: Prefix

mealie:
  email:
    host: smtp.example.com
    port: 587
    from_name: Mealie
    from_email: mealie@example.com
    user: smtp-user
    password: smtp-password
    auth_strategy: TLS
```

---

## Secrets and Config

This chart creates:

- A ConfigMap: `mealie-config`
- A Secret: `mealie-secrets`

Sensitive values like SMTP credentials should be placed under `.Values.mealie.email`.

---

## Database Integration

By default, this chart expects a CNPG (CloudNativePG) PostgreSQL cluster.

It references the CNPG-generated URI secret:

```yaml
env:
  - name: POSTGRES_URL_OVERRIDE
    valueFrom:
      secretKeyRef:
        name: cnpg-cluster-app
        key: uri
```

No manual configuration is needed if CNPG is installed correctly and the cluster is named appropriately.

---

## Persistent Volumes

The chart provisions a PVC for application data.

To configure:

```yaml
pvc:
  - name: mealie-data
    size: 10Gi
    storageClass: default
```

To disable PVCs, set the list to empty:

```yaml
pvc: []
```

---

## Security Context

This chart sets a default `fsGroup` of `1000` unless otherwise specified.

To override:

```yaml
podSecurityContext:
  fsGroup: 1000
  runAsUser: 1000
  runAsGroup: 1000
```

---

## Development

Test the chart locally:

```bash
helm template ./helm/mealie
```

Lint:

```bash
helm lint ./helm/mealie
```

## License

This Helm chart is provided under the MIT License.
