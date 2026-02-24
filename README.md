# Workflows de Push a GHCR

Repositorio con workflows automatizados para pushear **imágenes Docker** y **Helm Charts** a GitHub Container Registry (GHCR).

## 📁 Estructura

```
.github/workflows/
├── push-to-ghcr-linux.yml           # Push imágenes Docker (Linux)
├── push-to-ghcr-windows.yml         # Push imágenes Docker (Windows)
├── push-helm-charts-linux.yml       # Push Helm Charts (Linux)
└── push-helm-charts-windows.yml     # Push Helm Charts (Windows)

.
├── images-config.linux.json         # Config para imágenes Docker (Linux)
├── images-config.windows.json       # Config para imágenes Docker (Windows)
├── helm-charts-config.linux.json    # Config para Helm Charts (Linux)
├── helm-charts-config.windows.json  # Config para Helm Charts (Windows)
└── README.md                         # Esta documentación
```

---

## 🐳 Workflows de Imágenes Docker

### 📄 `images-config.linux.json` / `images-config.windows.json`

Estructura de configuración:

```json
[
  {
    "source_image": "nginx:alpine",
    "repository_name": "nginx",
    "tag": "v1"
  },
  {
    "source_image": "postgres:alpine3.22",
    "repository_name": "postgres",
    "tag": "v2"
  }
]
```

#### Campos:
- **source_image** ⚠️ (requerido): Imagen de origen a descargar (ej: `nginx:alpine`)
- **repository_name** ⚠️ (requerido): Nombre del repositorio en GHCR
- **tag** (opcional): Tag de la imagen en GHCR (por defecto: `latest`)

### 🔧 Características:

✅ **Validación automática** de configuración  
✅ **Manejo robusto de errores** - continúa si una imagen falla  
✅ **Reporte detallado** en el job summary  
✅ **Escaneo de vulnerabilidades** con Trivy (solo Linux)  
✅ **Limpieza automática** de imágenes locales  
✅ **Soporte para workflow_dispatch** - ejecutar manualmente con config personalizada  

### 🚀 Ejecución:

Los workflows se activan automáticamente cuando:
- Se modifica `images-config.linux.json` o `images-config.windows.json`
- Se modifica el workflow mismo
- Push a las ramas `main` o `develop`

O puedes ejecutar manualmente desde GitHub Actions.

---

## 📦 Workflows de Helm Charts

### 📄 `helm-charts-config.linux.json` / `helm-charts-config.windows.json`

Los workflows soportan **dos tipos de fuentes**:
1. **Charts locales** en tu repositorio
2. **Charts públicos de Artifact Hub** (espejo/upstream)

#### 📍 Charts Locales

Estructura de configuración:

```json
[
  {
    "type": "local",
    "chart_path": "./charts/mychart",
    "chart_name": "mychart",
    "chart_version": "1.0.0",
    "description": "Mi Helm Chart local"
  }
]
```

**Campos:**
- **type** ⚠️ (requerido): `"local"`
- **chart_path** ⚠️ (requerido): Ruta del directorio del chart (ej: `./charts/mychart`)
- **chart_name** ⚠️ (requerido): Nombre del chart
- **chart_version** (opcional): Versión (por defecto: `latest`)
- **description** (opcional): Descripción del chart

#### 🌐 Charts de Artifact Hub

Estructura de configuración:

```json
[
  {
    "type": "artifacthub",
    "artifacthub_chart": "bitnami/nginx",
    "chart_name": "nginx",
    "chart_version": "15.0.0",
    "description": "Nginx desde Artifact Hub"
  }
]
```

**Campos:**
- **type** ⚠️ (requerido): `"artifacthub"`
- **artifacthub_chart** ⚠️ (requerido): Identificador en Artifact Hub (ej: `bitnami/nginx`, `stable/mysql`)
- **chart_name** ⚠️ (requerido): Nombre con el que se guardará en GHCR
- **chart_version** (opcional): Versión (por defecto: `latest`)
- **description** (opcional): Descripción del chart

#### 📋 Ejemplo Combinado

```json
[
  {
    "type": "local",
    "chart_path": "./charts/myapp",
    "chart_name": "myapp",
    "chart_version": "1.2.0",
    "description": "Mi aplicación Kubernetes"
  },
  {
    "type": "artifacthub",
    "artifacthub_chart": "bitnami/postgresql",
    "chart_name": "postgresql",
    "chart_version": "12.0.0",
    "description": "PostgreSQL desde Artifact Hub"
  },
  {
    "type": "artifacthub",
    "artifacthub_chart": "bitnami/redis",
    "chart_name": "redis",
    "chart_version": "17.0.0",
    "description": "Redis desde Artifact Hub"
  }
]
```

### 🔧 Características:

✅ **Dual-source support** - charts locales + Artifact Hub en el mismo workflow  
✅ **Auto-detección de repositorio** - soporta bitnami, stable, incubator  
✅ **Validación automática** de configuración  
✅ **`helm lint`** para charts locales  
✅ **`helm pull`** para charts de Artifact Hub  
✅ **Empaquetamiento** automático  
✅ **Push a GHCR** con OCI registry support  
✅ **Indicadores visuales** (📂 local, 🌐 Artifact Hub)  
✅ **Manejo robusto de errores** - continúa si un chart falla  
✅ **Reporte detallado** con nombre y tipo  
✅ **Limpieza automática** de archivos temporales  

### 🚀 Ejecución:

Al igual que los workflows de Docker, se activan cuando:
- Se modifica `helm-charts-config.linux.json` o `helm-charts-config.windows.json`
- Se añaden/modifican archivos en `charts/`
- Se modifica el workflow mismo

O puedes ejecutar manualmente desde GitHub Actions con una config personalizada.

### 💡 Casos de Uso

#### 📂 Usar Charts Locales cuando:
- Tienes charts personalizados para tu aplicación
- Necesitas mantener versiones de charts internos
- Quieres un control total sobre las variables y valores

#### 🌐 Usar Artifact Hub cuando:
- Utilizas charts populares como PostgreSQL, Redis, Nginx, etc.
- Quieres mantener un espejo local de charts públicos en GHCR
- Necesitas versiones específicas de charts conocidos
- Quieres distribuir charts públicos a través de tu registry privado

#### 🔀 Usar ambos (Recomendado):
```json
[
  {
    "type": "local",
    "chart_path": "./charts/my-app",
    "chart_name": "my-app",
    "chart_version": "1.0.0",
    "description": "Mi aplicación custom"
  },
  {
    "type": "artifacthub",
    "artifacthub_chart": "bitnami/postgresql",
    "chart_name": "postgresql",
    "chart_version": "12.0.0",
    "description": "Base de datos PostgreSQL"
  },
  {
    "type": "artifacthub",
    "artifacthub_chart": "bitnami/redis",
    "chart_name": "redis",
    "chart_version": "17.0.0",
    "description": "Cache Redis"
  }
]
```

---

## 📊 Reportes y Resúmenes

Todos los workflows generan un **resumen ejecutivo** (Job Summary) con:

### ✅ Imágenes Docker - Caso exitoso:
```
# ✅ [Linux/Windows] Images Pushed Successfully

## 📦 Images Pushed Successfully (X/Y):
- ✅ source_image → ghcr.io/owner/repo:tag
- ✅ another_image → ghcr.io/owner/repo:tag

## ❌ Images Failed (Z/Y):
- ❌ FAILED - reason

## 🛡️ Security Report (Imágenes Linux):
- Escaneo de vulnerabilidades con Trivy
```

### ✅ Helm Charts - Caso exitoso:
```
# ✅ Helm Charts Pushed Successfully

## 📦 Helm Charts Pushed (3/3):
- 📂 myapp → ghcr.io/owner/myapp:1.2.0
  - Mi aplicación Kubernetes
- 🌐 postgresql → ghcr.io/owner/postgresql:12.0.0
  - PostgreSQL desde Artifact Hub
- 🌐 redis → ghcr.io/owner/redis:17.0.0
  - Redis desde Artifact Hub
```

### ⚠️ Caso sin elementos:
```
# ⚠️ No [Images/Charts] to Process

El archivo de configuración no contiene elementos válidos para procesar.
```

---

## 🔐 Permisos Requeridos

El workflow necesita estos permisos en `secrets.GITHUB_TOKEN`:

- `contents: read` - Leer el repositorio
- `packages: write` - Escribir en GHCR

✅ Estos permisos se conceden automáticamente en GitHub Actions.

---

## 🐛 Resolución de Problemas

### Workflows Docker

#### "No Images to Process"
**Causa:** El archivo de configuración está vacío o no tiene entradas válidas.  
**Solución:** Asegúrate de que cada objeto tenga `source_image` con valor no vacío.

#### "FAILED to pull: image:tag"
**Causa:** La imagen no existe en Docker Hub o no es accesible.  
**Solución:** Verifica que la imagen sea pública o tengas acceso.

#### "FAILED to push"
**Causa:** Problema con autenticación o permisos en GHCR.  
**Solución:** Verifica que `GITHUB_TOKEN` tenga permisos `packages: write`.

### Workflows Helm Charts

#### "No Helm Charts to Process"
**Causa:** El archivo de configuración está vacío o no tiene entradas válidas.  
**Solución:** Asegúrate de que cada objeto tenga:
- `type` con valor `"local"` o `"artifacthub"`
- `chart_name` con valor no vacío
- Si `type: "local"` → `chart_path` con valor no vacío
- Si `type: "artifacthub"` → `artifacthub_chart` con valor no vacío

#### "Helm lint error" (Charts locales)
**Causa:** El chart tiene errores de validación.  
**Solución:** Ejecuta `helm lint ./charts/mychart` localmente para más detalles.

#### "FAILED to pull: chart:version" (Artifact Hub)
**Causa:** El chart no existe en Artifact Hub o la versión es incorrecta.  
**Solución:** Verifica que el chart exista en https://artifacthub.io y que la versión sea correcta.

#### "Unknown chart type"
**Causa:** El campo `type` tiene un valor diferente a `"local"` o `"artifacthub"`.  
**Solución:** Usa solo `"local"` para charts locales y `"artifacthub"` para charts públicos.

---

## 📝 Notas

- **Linux (Imágenes):** Incluye escaneo de vulnerabilidades con Trivy
- **Windows (Imágenes):** No incluye Trivy (no está disponible en runners Windows)
- **Helm Charts:** Soporta charts locales Y públicos de Artifact Hub simultáneamente
- **OCI Support:** Los charts se pushean como OCI artifacts en GHCR
- **Tolerancia a fallos:** Si una imagen/chart falla, continúa con las siguientes
- **Repositorios públicos:** Auto-detecta y configura repos de bitnami, stable, incubator

---

## 📚 Referencias

- [Helm Documentation](https://helm.sh/docs/)
- [GHCR Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Trivy Scanner](https://github.com/aquasecurity/trivy)
- [GitHub Actions](https://docs.github.com/en/actions)
