# Integración entre Jira Cloud y GitHub Cloud
## Resumen Técnico Completo

---

## 1. Requisitos y Pasos para Conectar Jira Cloud con GitHub Cloud

### 1.1 Requisitos Previos

| Componente | Requisito |
|------------|-----------|
| **Jira Cloud** | Permiso de administrador del sitio (Site Administrator) |
| **GitHub Cloud** | Permiso de propietario de la organización (Organization Owner) |

> **Nota:** En algunas organizaciones, la conexión puede requerir colaboración entre equipos: un administrador de Jira instala la app y un propietario de GitHub conecta la organización.

### 1.2 Instalación de la App GitHub for Atlassian

1. En Jira, seleccionar **Apps** → **Explore more apps**
2. Buscar **"GitHub for Atlassian"** y seleccionarla
3. Hacer clic en **Get app** → **Get it now**

### 1.3 Conexión de una Organización de GitHub

1. Después de instalar la app, seleccionar **Get started**
   - Si ya está instalada: **Apps** → **Manage your apps** → **GitHub for Atlassian**
2. Seleccionar **Continue**
3. Elegir **GitHub Cloud** → **Next**
4. Ingresar credenciales de GitHub y hacer **Sign in**
5. Localizar la organización a conectar:
   - Si aparece listada y eres propietario: hacer clic en **Connect**
   - Si no tienes permisos: seleccionar **"Send them a link and ask them to connect"** para solicitar al propietario que complete la conexión

### 1.4 Agregar la App de Jira a una Nueva Organización de GitHub

Si la organización no aparece disponible:

1. Seleccionar la organización en GitHub
2. Elegir el acceso a repositorios:
   - **All repositories**: Jira accederá a todos los repositorios (incluyendo futuros)
   - **Only select repositories**: Acceso solo a repositorios específicos
3. Hacer clic en **Install** (solo si eres propietario de la organización)
4. Si no eres propietario: seleccionar **Request** para enviar solicitud al propietario

### 1.5 Conectar Nuevos Repositorios

Si se seleccionó "Only select repositories":

1. En Jira: **Apps** → **Manage apps** → **GitHub for Jira** → **Configure**
2. Localizar la organización y seleccionar el menú de tres puntos (…) → **Configure**
3. En GitHub: bajo **Repository access**, usar el dropdown **Select repositories**
4. Seleccionar los repositorios deseados y guardar

---

## 2. Funcionalidades Principales de la Integración

### 2.1 Sincronización de Datos de Desarrollo

La integración sincroniza automáticamente hacia Jira:

| Elemento | Descripción |
|----------|-------------|
| **Branches** | Ramas de Git creadas con claves de issue |
| **Commits** | Mensajes de commit que contienen claves de issue |
| **Pull Requests** | PRs con claves de issue en el título |
| **Builds** | Ejecuciones de GitHub Actions (workflows) |
| **Deployments** | Despliegues creados mediante GitHub Actions |

### 2.2 Asociación Automática mediante Claves de Issue

Para vincular actividad de desarrollo a un issue de Jira, se debe incluir la clave del issue (ej. `ABC-123`) en:

- **Nombre de rama**: `git checkout -b ABC-123-feature-name`
- **Mensaje de commit**: `git commit -m "ABC-123 Descripción del cambio"`
- **Título del Pull Request**: `ABC-123 Implementación de nueva funcionalidad`

### 2.3 Visualización de Información DevOps en Jira

Los datos de desarrollo se muestran en múltiples ubicaciones:

| Ubicación | Información Mostrada |
|-----------|---------------------|
| **Panel de Desarrollo** (en issues) | Branches, commits, PRs, builds, deployments vinculados |
| **Tablero (Board)** | Iconos en las tarjetas indicando actividad de desarrollo |
| **Pestaña Code** | Pull requests vinculados en los últimos 30 días |
| **Pestaña Releases** | PRs, builds y deployments asociados a la versión |
| **Pestaña Deployments** | Timeline de despliegues vinculados a issues |
| **Development Page** | Vista consolidada de PRs, métricas y datos de seguridad |

### 2.4 Enlaces en Comentarios de GitHub

Para crear enlaces a issues de Jira desde comentarios de GitHub, usar corchetes:
```
[ABC-123]
```
La app convierte automáticamente la clave en un enlace clickeable al issue de Jira.

---

## 3. Enlaces entre GitHub Actions / Workflows / Deployments y los Issues de Jira

### 3.1 Visualización de Builds (Workflows) en Jira

Los workflows de GitHub Actions se muestran como "builds" en Jira:

- Aparecen en el **panel de desarrollo** de los issues
- Se muestran en las **tarjetas del tablero**
- Se incluyen en el **Releases hub**

**Asociación de Workflows:**
- Un build se vincula automáticamente a un issue si la clave se encuentra en cualquier mensaje de commit relacionado con el PR
- Si se fusiona una rama sin PR, solo se revisa el último mensaje de commit de esa rama

**Configuración de Workflows para PRs:**
```yaml
on:
  pull_request:
    branches:
      - main
      - feature**
```

### 3.2 Visualización de Deployments en Jira

Los deployments se muestran en:
- Panel de desarrollo de issues
- Tarjetas del tablero
- Releases hub
- Timeline de deployments

**Requisitos para que los deployments aparezcan:**

1. Crear deployments usando:
   - [GitHub's Create Deployment API](https://docs.github.com/en/rest/deployments/deployments)
   - Acción `chrnorm/deployment-action@releases/v1`

2. Actualizar el estado del deployment al menos una vez usando:
   - [Create Deployment Status API](https://docs.github.com/en/rest/deployments/statuses)
   - Acción `chrnorm/deployment-status@releases/v1`

> **Importante:** La app GitHub for Atlassian solo escucha eventos `deployment_status`.

### 3.3 Mapeo de Ambientes de Despliegue

Jira reconoce automáticamente nombres de ambientes estándar en inglés. Para nombres personalizados (ej. otros idiomas), crear archivo `.jira/config.yml` en la rama principal:

```yaml
deployments:
  environmentMapping:
    development:
      - "dev*"
      - "Entwicklung"
      - "desenvolvimento"
      - "desarrollo"
    testing:
      - "testes"
      - "Test"
      - "TST-*"
      - "pruebas"
    staging:
      - "Pre-Prod"
      - "STG-*"
      - "staging"
      - "preproducción"
    production:
      - "Produktion"
      - "produção"
      - "PROD-*"
      - "producción"
```

> Se pueden especificar hasta 10 patrones glob por cada uno de los 4 tipos de ambiente válidos.

---

## 4. Automatizaciones Disponibles (DevOps Triggers)

### 4.1 Requisitos Previos

- Herramienta de control de código fuente conectada a Jira Cloud
- Las herramientas self-hosted (Bitbucket Server, GitLab On-Premise, GitHub Enterprise Server) **no** soportan triggers de automatización, aunque sí permiten otras funcionalidades de integración

### 4.2 Triggers de DevOps Disponibles

#### Triggers de Código Fuente

| Trigger | Smart Value | Descripción |
|---------|-------------|-------------|
| **Branch created** | `{{branch}}` | Se dispara cuando se crea una rama |
| **Commit created** | `{{commit}}` | Se dispara cuando se crea un commit |
| **Pull request created** | `{{pullRequest}}` | Se dispara cuando se crea un PR |
| **Pull request merged** | `{{pullRequest}}` | Se dispara cuando se fusiona un PR |
| **Pull request declined** | `{{pullRequest}}` | Se dispara cuando se rechaza un PR |

#### Triggers de Build

| Trigger | Smart Values | Descripción |
|---------|--------------|-------------|
| **Build successful** | `{{development}}` | Se dispara cuando un build es exitoso |
| **Build failed** | `{{development}}` | Se dispara cuando un build falla |
| **Build status changed** | `{{development}}` | Se dispara cuando cambia el estado de un build |

#### Triggers de Deployment

| Trigger | Smart Values | Descripción |
|---------|--------------|-------------|
| **Deployment successful** | `{{development}}` | Se dispara cuando un deployment es exitoso |
| **Deployment failed** | `{{development}}` | Se dispara cuando un deployment falla |
| **Deployment status changed** | `{{development}}` | Se dispara cuando cambia el estado de un deployment |

### 4.3 Ejemplos de Automatizaciones Comunes

#### Ejemplo 1: Mover issue a "En Progreso" cuando se crea una rama
```
Trigger: Branch created
Condición: Branch name contains issue key
Acción: Transition issue to "In Progress"
```

#### Ejemplo 2: Mover issue a "En Revisión" cuando se crea un PR
```
Trigger: Pull request created
Condición: PR title contains issue key
Acción: Transition issue to "In Review"
```

#### Ejemplo 3: Cerrar issue automáticamente cuando se fusiona el PR
```
Trigger: Pull request merged
Condición: PR title contains issue key
Acción: Transition issue to "Done"
```

#### Ejemplo 4: Notificar al equipo cuando falla un deployment
```
Trigger: Deployment failed
Acción: Send Slack message to #devops-alerts
Acción: Add comment to linked issues
```

#### Ejemplo 5: Actualizar issue cuando deployment llega a producción
```
Trigger: Deployment successful
Condición: Environment type = "production"
Acción: Add label "deployed-to-prod"
Acción: Log work on issue
```

---

## 5. Permisos Necesarios

### 5.1 Permisos en Jira

| Ámbito | Permisos Requeridos |
|--------|---------------------|
| **Development Information** | Lectura, escritura y administración (branches, commits, pull requests) |
| **View Development Tools** | Permiso de proyecto necesario para ver el panel de desarrollo en issues |

### 5.2 Permisos en GitHub (Repository Permissions)

| Permiso | Propósito |
|---------|-----------|
| **Read-only: Actions** | Acceso a eventos `workflow_run` para información de builds |
| **Read-only: Code scanning alerts** | Recibir alertas de escaneo de código en Jira |
| **Read-only: Deployments** | Ver información de builds y deployments |
| **Read-only: Metadata** | Requisito obligatorio de GitHub para todas las apps |
| **Read & Write: Issues y PRs** | Smart Commits y unfurling de URLs de Jira en comentarios |
| **Read & Write: Contents (code)** | Sincronizar información de desarrollo; creación de branches desde Jira |

### 5.3 Permisos en GitHub (Organization Permissions)

| Permiso | Propósito |
|---------|-----------|
| **Read-only: Members** | Determinar si el usuario tiene acceso de admin a la organización |

### 5.4 Eventos de Webhook Suscritos

| Evento | Ocurrencia |
|--------|------------|
| Code scanning alert | Alerta creada, corregida o cerrada |
| Commit comment | Comentario de commit creado |
| Create | Branch o tag de Git creado |
| Delete | Branch o tag de Git eliminado |
| Deployment status | Deployment creado |
| Issue comment | Actividad en comentarios de issues/PRs |
| Issues | Actividad relacionada con issues |
| Pull request | Actividad relacionada con PRs |
| Pull request review | Actividad en revisiones de PR |
| Push | Commits enviados a branch o tag |
| Repository | Actividad relacionada con el repositorio |
| Workflow run | Workflow de GitHub Actions solicitado o completado |

### 5.5 Consideraciones de Seguridad y Gobernanza

- **Principio de mínimo privilegio**: Seleccionar "Only select repositories" para limitar el acceso solo a repositorios necesarios
- **IP Allowlists**: Si se usan allowlists de IP en GitHub, consultar la [documentación de configuración](https://github.com/atlassian/github-for-jira/blob/main/docs/ip-allowlist.md)
- **Auditoría de permisos**: Revisar periódicamente las organizaciones y repositorios conectados
- **Roles de proyecto**: Usar esquemas de permisos en Jira para controlar quién puede ver datos de desarrollo

---

## 6. Consultas Avanzadas con JQL Aplicadas a DevOps

### 6.1 Búsquedas de Código Fuente

| Consulta | Descripción |
|----------|-------------|
| `development[pullrequests].all > 0` | Issues con cualquier PR |
| `development[pullrequests].open > 0` | Issues con PRs abiertos |
| `development[commits].all > 0` | Issues con commits asociados |
| `development[reviews].all > 0` | Issues con reviews de PR |
| `development[reviews].open > 0` | Issues con reviews pendientes |
| `development[builds].failing > 0` | Issues con builds fallidos |

### 6.2 Búsquedas de Builds

| Consulta | Descripción |
|----------|-------------|
| `buildState ~ "failed"` | Issues donde el último build falló |
| `buildState ~ "successful"` | Issues donde el último build fue exitoso |
| `buildState ~ "in progress"` | Issues con builds en progreso |
| `buildState ~ "pending"` | Issues con builds pendientes |
| `buildName ~ "my-pipeline"` | Issues con un build específico por nombre |

**Ejemplo combinado:**
```jql
buildState ~ "FAILED" AND development[pullrequests].open > 0
```
> Muestra issues con PRs abiertos donde el último build falló

### 6.3 Búsquedas de Deployments

| Consulta | Descripción |
|----------|-------------|
| `deploymentEnvironmentType ~ "production"` | Issues desplegados a producción |
| `deploymentEnvironmentType ~ "staging"` | Issues desplegados a staging |
| `deploymentEnvironmentType ~ "testing"` | Issues desplegados a testing |
| `deploymentEnvironmentType ~ "development"` | Issues desplegados a desarrollo |
| `deploymentEnvironmentType !~ "production"` | Issues NO desplegados a producción |
| `deploymentEnvironmentName ~ "prod-east"` | Issues desplegados a ambiente específico |
| `deploymentState ~ "successful"` | Issues con deployment exitoso |
| `deploymentState ~ "failed"` | Issues con deployment fallido |
| `deploymentState ~ "in_progress"` | Issues con deployment en progreso |
| `deploymentState ~ "rolled_back"` | Issues con deployment revertido |

### 6.4 Ejemplos Prácticos de JQL para DevOps

#### Issues completados sin desplegar a producción:
```jql
status = Done AND deploymentEnvironmentType !~ "production"
```

#### Issues con PRs abiertos y ya en producción:
```jql
deploymentEnvironmentType ~ "production" AND development[pullrequests].open > 0
```

#### Issues sin actividad de desarrollo:
```jql
project = "MYPROJECT" AND development[pullrequests].all = 0 AND development[commits].all = 0
```

#### Issues con deploy reciente a producción:
```jql
deploymentEnvironmentType ~ "production" AND deploymentState ~ "successful" AND updated >= -7d
```

#### Issues con builds fallidos en el sprint actual:
```jql
sprint in openSprints() AND buildState ~ "failed"
```

#### Issues listos para deploy (PRs mergeados, no en producción):
```jql
status = "Ready for Deploy" AND development[pullrequests].open = 0 AND deploymentEnvironmentType !~ "production"
```

### 6.5 Búsquedas de Feature Flags (si se usa integración con herramientas de feature flags)

| Consulta | Descripción |
|----------|-------------|
| `flagEnabled ~ "true"` | Issues con feature flag habilitada |
| `flagEnabled ~ "false"` | Issues con feature flag deshabilitada |
| `flagEnabledRollout ~ "partial"` | Feature flag ON con rollout parcial (>0% y <100%) |
| `flagEnabledRollout ~ "full"` | Feature flag ON al 100% |
| `flagName ~ "MyFeature"` | Issues relacionados con un flag específico |

---

## 7. Buenas Prácticas Generales

### 7.1 Convenciones para Referenciar Issues

#### Nomenclatura de Branches
```
<CLAVE-ISSUE>-<descripcion-breve>
```
**Ejemplos:**
- `ABC-123-add-login-feature`
- `DEV-456-fix-payment-bug`
- `PROJ-789-refactor-api-module`

#### Mensajes de Commit
```
<CLAVE-ISSUE> <Descripción del cambio>
```
**Ejemplos:**
- `ABC-123 Add user authentication endpoint`
- `DEV-456 Fix null pointer exception in payment service`
- `PROJ-789 Refactor API module for better performance`

#### Títulos de Pull Request
```
<CLAVE-ISSUE> <Descripción concisa del cambio>
```
**Ejemplos:**
- `ABC-123 Implement user login feature`
- `DEV-456 Fix payment processing bug`

### 7.2 Recomendaciones para Maximizar Trazabilidad

#### Estructura del Flujo DevOps

1. **Crear Issue en Jira** antes de iniciar cualquier trabajo
2. **Crear Branch** con la clave del issue en el nombre
3. **Commits atómicos** con claves de issue en cada mensaje
4. **Pull Request** con clave en el título
5. **Revisión de código** con comentarios que referencien issues `[ABC-123]`
6. **Merge** tras aprobación
7. **Deploy automatizado** vinculado al PR/commit

#### Configuración de Workflows de GitHub Actions

Para máxima visibilidad en Jira:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop, feature/**]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # ... pasos de build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Create Deployment
        uses: chrnorm/deployment-action@releases/v1
        id: deployment
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          environment: production

      # ... pasos de deploy

      - name: Update Deployment Status
        uses: chrnorm/deployment-status@releases/v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          state: "success"
          deployment_id: ${{ steps.deployment.outputs.deployment_id }}
```

### 7.3 Automatizaciones Recomendadas

| Evento | Automatización Sugerida |
|--------|------------------------|
| Branch creada | Mover issue a "En Desarrollo" |
| PR creado | Mover issue a "En Revisión" |
| PR mergeado | Mover issue a "Listo para Deploy" o "Hecho" |
| Build fallido | Notificar al equipo, agregar etiqueta "build-failed" |
| Deploy a staging | Mover issue a "En QA" |
| Deploy a producción | Cerrar issue, agregar etiqueta "released" |

### 7.4 Mantenimiento y Gobernanza

- **Revisar conexiones periódicamente**: Verificar que solo las organizaciones y repositorios necesarios estén conectados
- **Auditar permisos**: Asegurar que los usuarios correctos tengan acceso a ver datos de desarrollo
- **Monitorear automatizaciones**: Revisar el historial de ejecución de reglas de automatización
- **Documentar flujos**: Mantener documentación actualizada de los flujos DevOps y su integración con Jira
- **Capacitar al equipo**: Asegurar que todos los desarrolladores conozcan las convenciones de nomenclatura

---

## Referencias

- [Connect GitHub Cloud to Jira](https://support.atlassian.com/jira-cloud-administration/docs/integrate-with-github-cloud/)
- [Link GitHub workflows and deployments to Jira issues](https://support.atlassian.com/jira-cloud-administration/docs/link-github-workflows-and-deployments-to-jira-issues/)
- [Use the GitHub for Jira app](https://support.atlassian.com/jira-cloud-administration/docs/use-the-github-for-jira-app/)
- [Jira Automation – DevOps Triggers](https://support.atlassian.com/cloud-automation/docs/jira-automation-triggers/#DevOps-triggers)
- [Manage project permissions](https://support.atlassian.com/jira-cloud-administration/docs/manage-project-permissions/)
- [JQL Developer Status](https://support.atlassian.com/jira-software-cloud/docs/jql-developer-status/)
- [Permissions required for GitHub for Atlassian](https://support.atlassian.com/jira-cloud-administration/docs/permissions-required-for-github-for-jira/)

---

*Documento generado: Diciembre 2025*
