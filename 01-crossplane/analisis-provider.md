# =====================================================================
# Análisis del Provider PostgreSQL
# =====================================================================

## Provider: tages/provider-postgresql v0.1.0

### 1. Managed Resources disponibles

## 1. Managed Resources

  - **Database:** crea y administra una base de datos en PostgreSQL.
  - **Role:** crea y administra roles o usuarios de PostgreSQL.
  - **Grant:** concede permisos sobre objetos de la base de datos.
  - **Schema:** crea y administra esquemas.
  - **Extension:** habilita extensiones de PostgreSQL en una base de datos.
  - **Function:** administra funciones SQL.
  - **Publication:** configura publicaciones para replicación lógica.
  - **Subscription:** configura suscripciones para replicación lógica.
  - **ReplicationSlot / PhysicalReplicationSlot:** administra slots de replicación.
  - **RoleGrant:** gestiona la pertenencia de un rol a otro.
  - **UserMapping:** configura el mapeo de usuarios para servidores externos.
  - **DefaultPrivilege:** define permisos predeterminados para objetos futuros.
  - **Server:** administra servidores externos de PostgreSQL.


### 2. Campos requeridos del recurso Database

El esquema de `spec.forProvider` no marca campos obligatorios. Por tanto,
  todos son opcionales desde la validación OpenAPI.

  Los campos disponibles son: `allowConnections`, `connectionLimit`, `encoding`,
  `isTemplate`, `lcCollate`, `lcCtype`, `name`, `owner`, `tablespaceName` y
  `template`.

  En esta PoC usaré `name` para indicar el nombre de la base de datos y `owner`
  para el dueño.


### 3. Información requerida por el ProviderConfig

El ProviderConfig necesita la información de conexión a PostgreSQL: host,
  puerto, usuario, contraseña, base de datos y modo SSL.

  En esta ocación las credenciales provienen de un Secret de Kubernetes. El campo
  `credentials.source` es obligatorio y se configura como `Secret`; luego
  `secretRef` identifica el namespace, nombre y clave del Secret que contiene
  la conexión.

