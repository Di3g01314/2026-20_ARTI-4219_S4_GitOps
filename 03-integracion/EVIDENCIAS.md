# Evidencias de la PoC GitOps

Repositorio entregado: <https://github.com/Di3g01314/2026-20_ARTI-4219_S4_GitOps>

Este documento registra las verificaciones realizadas durante la PoC de Crossplane y Argo CD. No incluye credenciales, tokens ni secretos.

## 1. Argo CD sincronizado

Comando ejecutado:

```bash
argocd app get gitops-demo
```

Resultado observado:

```text
Sync Status:   Synced
Health Status: Healthy
Source:        rama main, ruta 01-crossplane
```

La Application `gitops-demo` usa este repositorio como fuente de verdad y tiene activadas las políticas `prune` y `selfHeal`.

## 2. Recursos administrados por Argo CD

Comando ejecutado:

```bash
argocd app resources gitops-demo
```

Resultado observado: Argo CD administra el Provider de PostgreSQL, la función `function-patch-and-transform`, el `ProviderConfig`, el XRD, la Composition, el Claim y su recurso compuesto. Ninguno apareció como huérfano.

## 3. Recursos de Crossplane y base de datos

Comandos ejecutados:

```bash
kubectl get postgresqlinstances -n default
kubectl get databases.postgresql.postgresql.upbound.io
```

Resultado observado:

```text
NAME               SYNCED   READY
databaseplatform   True     True

NAME                                  SYNCED   READY   EXTERNAL-NAME
databaseplatform-6wc8z-a723a87e1479   True     True    appdb-v2
```

También se verificó la base desde el contenedor PostgreSQL:

```bash
kubectl exec -n postgresql postgresql-0 -- \
  env PGPASSWORD=platform123 \
  psql -U postgres -d postgres -c '\l'
```

Resultado observado: la lista de bases contiene `appdb-v2`.

## 4. Actualización de infraestructura mediante Git

Se cambió el parámetro `dbName` en `01-crossplane/claim.yaml` de `appdb` a `appdb-v2`.

```yaml
spec:
  parameters:
    dbName: appdb-v2
    owner: postgres
```

El cambio se publicó en la rama `main` mediante el commit `f019c3b` (`feat: rename database to appdb-v2`). Argo CD sincronizó la nueva revisión y Crossplane reconcilió el recurso hasta crear `appdb-v2`.

## 5. Self-healing de Argo CD

Se simuló un cambio no autorizado directamente en el clúster:

```bash
kubectl patch postgresqlinstance databaseplatform -n default \
  --type=merge \
  -p '{"spec":{"parameters":{"dbName":"appdb-hacked"}}}'
```

Kubernetes confirmó que el Claim fue modificado. Sin realizar ningún commit con ese valor, se comprobó el estado final:

```bash
kubectl get postgresqlinstance -n default \
  -o jsonpath='{.items[0].spec.parameters.dbName}{"\n"}'
```

Resultado observado:

```text
appdb-v2
```

Argo CD revirtió automáticamente el cambio no autorizado y restauró el estado definido en Git.
