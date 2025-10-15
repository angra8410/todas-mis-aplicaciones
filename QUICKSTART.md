# 🚀 Guía Rápida de Configuración

Esta guía te ayudará a configurar el workflow CI/CD en **5 minutos**.

## ✅ Checklist de Configuración

- [ ] Crear Personal Access Token (PAT)
- [ ] Configurar secret en repositorio fuente
- [ ] Copiar workflow al repositorio fuente
- [ ] Ajustar configuración según tus necesidades
- [ ] Probar el workflow

## 📋 Paso a Paso

### 1️⃣ Crear Personal Access Token

1. Ve a: https://github.com/settings/tokens
2. Click en **"Generate new token (classic)"**
3. Configuración:
   - **Note**: `PAT_TODAS_MIS_APLICACIONES`
   - **Expiration**: 90 días (o según prefieras)
   - **Scopes**: 
     - ✅ `repo` (todos los sub-scopes)
     - ✅ `workflow`
4. Click en **"Generate token"**
5. **¡COPIA EL TOKEN!** (solo se muestra una vez)

### 2️⃣ Configurar Secret en `aplicaciones_laborales`

1. Ve a: https://github.com/angra8410/aplicaciones_laborales/settings/secrets/actions
2. Click en **"New repository secret"**
3. Configura:
   - **Name**: `PAT_TODAS_MIS_APLICACIONES`
   - **Secret**: Pega el token del paso anterior
4. Click en **"Add secret"**

### 3️⃣ Copiar Workflow

1. En el repositorio `aplicaciones_laborales`, crea:
   ```
   .github/workflows/copy-cv-to-todas-mis-aplicaciones.yml
   ```

2. Copia el contenido de: [TEMPLATE-copy-cv-from-source.yml](.github/workflows/TEMPLATE-copy-cv-from-source.yml)

3. Guarda el archivo (commit)

### 4️⃣ Ajustar Configuración

Abre el archivo copiado y ajusta:

#### A) Nombre del workflow que genera el PDF (línea ~7):

```yaml
on:
  workflow_run:
    workflows: ["Generate CV PDF"]  # 👈 CAMBIA ESTO al nombre real
```

**¿Cómo encontrar el nombre?**
- Ve a `.github/workflows/` en tu repo
- Busca el archivo del workflow que genera el PDF
- Copia el valor de `name:` al inicio del archivo

#### B) Patrón de búsqueda del PDF (línea ~30, opcional):

Si tus PDFs tienen un patrón de nombre diferente, ajusta:

```yaml
PDF_FILE=$(find . -type f \
  \( -name "*tu_patron*.pdf" \) \    # 👈 Ajusta el patrón
  -printf '%T@ %p\n' | sort -rn | head -1 | cut -d' ' -f2-)
```

**Ejemplos comunes:**
- `*cv*.pdf` → Busca archivos con "cv" en el nombre
- `*hoja_de_vida*.pdf` → Busca "hoja_de_vida"
- `*resume*.pdf` → Busca "resume"
- `cv_*.pdf` → Busca archivos que empiecen con "cv_"

### 5️⃣ Probar el Workflow

#### Opción A: Ejecución Manual

1. Ve a: https://github.com/angra8410/aplicaciones_laborales/actions
2. Selecciona **"Copy CV PDF to Todas Mis Aplicaciones"**
3. Click en **"Run workflow"**
4. Ingresa la fecha (formato: `2025-01-15`)
5. Click en **"Run workflow"**
6. Espera ~30 segundos
7. Revisa los logs

#### Opción B: Ejecución Automática

1. Genera un nuevo PDF usando tu workflow normal
2. El workflow debería ejecutarse automáticamente
3. Revisa: https://github.com/angra8410/aplicaciones_laborales/actions

## 🔍 Verificación

Después de ejecutar el workflow:

1. ✅ Ve a: https://github.com/angra8410/todas-mis-aplicaciones
2. ✅ Busca la carpeta `aplicaciones/YYYY-MM-DD/`
3. ✅ Verifica que el PDF esté ahí
4. ✅ Lee el `README.md` de la carpeta

## ❓ Preguntas Frecuentes

### ¿Qué pasa si el workflow falla?

1. Ve a los **Actions logs** en el repositorio fuente
2. Busca el ícono ❌ rojo
3. Lee el mensaje de error
4. Consulta la sección de **Solución de Problemas** en [README-WORKFLOW.md](README-WORKFLOW.md)

### ¿Cómo saber si funcionó?

Verás en los logs:

```
✅ PDF encontrado: ./output/cv_nombre.pdf
✅ Fecha validada: 2025-01-15
✅ Archivo copiado y verificado exitosamente
✅ Push exitoso
✅ PROCESO COMPLETADO EXITOSAMENTE
```

### ¿Puedo cambiar la estructura de carpetas?

Sí, modifica la línea que crea el directorio:

```yaml
DEST_DIR="todas-mis-aplicaciones/aplicaciones/$CV_DATE"
```

Cámbiala a:

```yaml
DEST_DIR="todas-mis-aplicaciones/TU_CARPETA/$CV_DATE"
```

### ¿Qué pasa si ejecuto el workflow dos veces con el mismo PDF?

El workflow detecta archivos duplicados:
- Si el archivo es **idéntico**: No hace nada (logs: "No hay cambios")
- Si el archivo es **diferente**: Crea un backup y copia el nuevo

## 🆘 Ayuda

Si tienes problemas:

1. **Lee los logs del workflow** - Tienen emojis y mensajes claros
2. **Consulta** [README-WORKFLOW.md](README-WORKFLOW.md) - Solución de problemas detallada
3. **Verifica**:
   - ✅ El PAT token está configurado correctamente
   - ✅ El token no ha expirado
   - ✅ El workflow que genera el PDF terminó exitosamente
   - ✅ El archivo PDF existe en el repositorio fuente

## 📚 Documentación Completa

Para más detalles, consulta [README-WORKFLOW.md](README-WORKFLOW.md)

---

**¿Todo listo?** → Pasa al siguiente paso o prueba el workflow 🚀
