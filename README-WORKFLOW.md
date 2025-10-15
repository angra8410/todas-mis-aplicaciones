# CI/CD Workflow para Copiar Hojas de Vida (CV) PDF

Este repositorio contiene la configuración necesaria para recibir automáticamente hojas de vida en formato PDF desde el repositorio `aplicaciones_laborales`.

## 📋 Descripción General

El flujo de trabajo automatizado copia la hoja de vida generada en PDF desde el repositorio fuente (`aplicaciones_laborales`) a este repositorio público (`todas-mis-aplicaciones`), organizando los archivos por fecha de aplicación en carpetas con formato `YYYY-MM-DD`.

## 🏗️ Arquitectura del Workflow

### Flujo de Trabajo

```
aplicaciones_laborales                     todas-mis-aplicaciones
(Repositorio Fuente)                       (Repositorio Destino)
        |                                          |
        | 1. Genera PDF                           |
        |    de CV                                 |
        v                                          |
  [Workflow CI/CD]                                 |
        |                                          |
        | 2. Encuentra PDF                        |
        | 3. Valida archivo                       |
        |                                          |
        | 4. Checkout destino                     |
        |    con PAT token                        |
        v                                          |
  [Copia PDF] -------------------------------->    |
        |                                          v
        | 5. Crea carpeta                    [aplicaciones/]
        |    YYYY-MM-DD                            |
        |                                          v
        | 6. Commit & Push                   [YYYY-MM-DD/]
        v                                          |
  [Logs y auditoría]                               v
                                           [CV_PDF + README]
```

## 📁 Estructura de Carpetas

Los archivos se organizan de la siguiente manera:

```
todas-mis-aplicaciones/
├── .github/
│   └── workflows/
│       ├── receive-cv-from-aplicaciones-laborales.yml  # Workflow receptor (opcional)
│       └── TEMPLATE-copy-cv-from-source.yml            # Plantilla para repo fuente
├── aplicaciones/
│   ├── 2025-01-15/
│   │   ├── cv_nombre.pdf
│   │   └── README.md
│   ├── 2025-01-20/
│   │   ├── cv_nombre.pdf
│   │   └── README.md
│   └── YYYY-MM-DD/
│       ├── *.pdf                # Solo la hoja de vida en PDF
│       └── README.md            # Metadata de la aplicación
└── README-WORKFLOW.md           # Este archivo
```

## 🔧 Configuración

### Requisitos Previos

1. **Personal Access Token (PAT)**: Necesario para que el workflow del repositorio fuente pueda escribir en este repositorio.

### Paso 1: Crear Personal Access Token (PAT)

1. Ve a GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click en "Generate new token (classic)"
3. Configuración del token:
   - **Note**: `PAT_TODAS_MIS_APLICACIONES`
   - **Expiration**: Configura según tus necesidades (ej: 90 días, 1 año)
   - **Scopes**: Selecciona:
     - ✅ `repo` (Full control of private repositories)
       - ✅ `repo:status`
       - ✅ `repo_deployment`
       - ✅ `public_repo`
       - ✅ `repo:invite`
     - ✅ `workflow` (Update GitHub Action workflows)

4. Genera el token y **cópialo** (solo se muestra una vez)

### Paso 2: Configurar Secret en Repositorio Fuente

1. Ve al repositorio `aplicaciones_laborales`
2. Settings → Secrets and variables → Actions → New repository secret
3. Configura:
   - **Name**: `PAT_TODAS_MIS_APLICACIONES`
   - **Value**: Pega el token generado en el paso anterior
4. Click en "Add secret"

### Paso 3: Implementar Workflow en Repositorio Fuente

1. En el repositorio `aplicaciones_laborales`, crea el archivo:
   ```
   .github/workflows/copy-cv-to-todas-mis-aplicaciones.yml
   ```

2. Copia el contenido de `TEMPLATE-copy-cv-from-source.yml` (disponible en este repositorio)

3. Ajusta los siguientes parámetros según tu caso:

   ```yaml
   on:
     workflow_run:
       workflows: ["Generate CV PDF"]  # ⚠️ Cambiar al nombre real de tu workflow
       types:
         - completed
   ```

4. Ajusta el patrón de búsqueda del PDF si es necesario:
   
   ```yaml
   # En el step "Find generated CV PDF"
   PDF_FILE=$(find . -type f -name "*cv*.pdf" -o -name "*hoja_de_vida*.pdf" -o -name "*resume*.pdf" | sort -r | head -n 1)
   ```

## 🎯 Funcionalidades Clave

### ✅ Validaciones Implementadas

1. **Validación de formato de fecha**: Asegura que la fecha esté en formato `YYYY-MM-DD`
2. **Validación de existencia del PDF**: Verifica que el archivo PDF exista y tenga contenido
3. **Validación de copia exitosa**: Confirma que el archivo se copió correctamente antes del commit
4. **Solo PDF de CV**: El workflow busca específicamente archivos de hoja de vida, no otros documentos

### 📊 Logs y Auditoría

El workflow incluye logs detallados en cada paso:

- 🔍 Búsqueda de archivos PDF
- ✅ Validación de archivos
- 📁 Creación de carpetas
- 📄 Copia de archivos
- ⬆️ Operaciones de Git (add, commit, push)
- ✅/❌ Estado final del proceso

### 📝 Metadata Automática

Cada carpeta de aplicación incluye un `README.md` con:
- Fecha de aplicación
- Nombre del archivo PDF
- Fecha de generación
- ID del workflow ejecutado
- Enlace directo al archivo

## 🚀 Uso

### Ejecución Automática

El workflow se ejecuta automáticamente cuando:
1. Se completa exitosamente el workflow de generación de PDF en `aplicaciones_laborales`

### Ejecución Manual

Para ejecutar manualmente:

1. Ve al repositorio `aplicaciones_laborales`
2. Actions → "Copy CV PDF to Todas Mis Aplicaciones"
3. Click en "Run workflow"
4. Ingresa la fecha de aplicación (formato: `YYYY-MM-DD`)
5. Click en "Run workflow"

## 🔍 Diagnóstico y Solución de Problemas

### El workflow no encuentra el PDF

**Problema**: `❌ Error: No se encontró ningún archivo PDF de hoja de vida`

**Solución**:
1. Verifica que el workflow de generación de PDF se completó exitosamente
2. Revisa el patrón de búsqueda en el step "Find generated CV PDF"
3. Ajusta el patrón según el nombre de tus archivos:
   ```bash
   PDF_FILE=$(find . -type f -name "TU_PATRON*.pdf" | sort -r | head -n 1)
   ```

### Error de permisos al hacer push

**Problema**: `❌ Error: Permission denied (publickey)`

**Solución**:
1. Verifica que el PAT token está configurado correctamente en los secrets
2. Confirma que el token tiene los permisos necesarios (`repo` y `workflow`)
3. Verifica que el token no ha expirado

### El archivo se copia pero no se hace commit

**Problema**: `ℹ️ No hay cambios para guardar (el archivo ya existe)`

**Solución**:
- Esto es normal si el archivo ya existe en el destino
- Si necesitas sobrescribir, modifica el workflow para eliminar el archivo anterior antes de copiar

### Formato de fecha incorrecto

**Problema**: `❌ Error: La fecha debe estar en formato YYYY-MM-DD`

**Solución**:
1. Verifica que la fecha de entrada cumple el formato `YYYY-MM-DD`
2. Ejemplo correcto: `2025-01-15`
3. Ejemplos incorrectos: `15-01-2025`, `2025/01/15`, `15/1/2025`

## 🛡️ Seguridad

### Buenas Prácticas Implementadas

1. **PAT Token en Secrets**: El token nunca se expone en los logs
2. **Validación de archivos**: Solo se transfieren archivos PDF validados
3. **Scope mínimo necesario**: El token solo tiene permisos de repositorio
4. **Auditoría completa**: Cada operación se registra en los logs del workflow
5. **Commits firmados**: Los commits incluyen metadata del workflow

### Recomendaciones de Seguridad

- 🔄 Rota el PAT token periódicamente
- 🔒 Usa tokens con la menor duración práctica
- 📊 Revisa regularmente los logs de ejecución
- 🚫 No compartas el PAT token
- ✅ Usa tokens específicos por repositorio cuando sea posible

## 📚 Referencias

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Creating a Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Using Secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## 🤝 Contribuciones

Si encuentras algún problema o tienes sugerencias de mejora, por favor:
1. Revisa los logs del workflow
2. Consulta la sección de "Diagnóstico y Solución de Problemas"
3. Abre un issue en el repositorio si el problema persiste

## 📄 Licencia

Este workflow es parte del proyecto `todas-mis-aplicaciones` y está disponible para uso personal.
