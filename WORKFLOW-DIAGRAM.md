# 🔄 Diagrama de Flujo del Workflow CI/CD

## Flujo Completo del Sistema

```
┌─────────────────────────────────────────────────────────────────────┐
│                   REPOSITORIO: aplicaciones_laborales               │
│                          (Repositorio Fuente)                       │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   │ 1. Genera PDF de CV
                                   ▼
                    ┌──────────────────────────┐
                    │  Workflow: Generate CV   │
                    │     PDF completo         │
                    └──────────────────────────┘
                                   │
                                   │ 2. Trigger automático
                                   ▼
    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    ┃       WORKFLOW: Copy CV PDF to Todas Mis Aplicaciones       ┃
    ┃              (TEMPLATE-copy-cv-from-source.yml)             ┃
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                                   │
            ┌──────────────────────┴──────────────────────┐
            │                                              │
            ▼                                              ▼
    ┌────────────────┐                           ┌────────────────┐
    │ Step 1:        │                           │ Step 2:        │
    │ Checkout repo  │                           │ Buscar PDF     │
    │ fuente         │                           │ generado       │
    └────────────────┘                           └────────────────┘
            │                                              │
            │                                              │ 🔍 Busca en:
            │                                              │ - output/
            │                                              │ - dist/
            │                                              │ - build/
            │                                              │
            └──────────────────────┬──────────────────────┘
                                   │
                                   ▼
                         ┌────────────────┐
                         │ ✅ Validación  │
                         │ - Existe PDF?  │
                         │ - Tamaño > 0?  │
                         │ - Es PDF?      │
                         └────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │ ❌ Error                    │ ✅ OK
                    │                             │
                    ▼                             ▼
            ┌────────────────┐           ┌────────────────┐
            │ Detener con    │           │ Step 3:        │
            │ mensaje claro  │           │ Extraer fecha  │
            └────────────────┘           └────────────────┘
                                                  │
                                    ┌─────────────┼─────────────┐
                                    │             │             │
                                    ▼             ▼             ▼
                            ┌──────────┐  ┌──────────┐  ┌──────────┐
                            │ Nombre   │  │ Commit   │  │ Fecha    │
                            │ archivo  │  │ message  │  │ actual   │
                            └──────────┘  └──────────┘  └──────────┘
                                    │             │             │
                                    └─────────────┼─────────────┘
                                                  │
                                                  ▼
                                         ┌────────────────┐
                                         │ ✅ Validar     │
                                         │ YYYY-MM-DD     │
                                         └────────────────┘
                                                  │
                                                  ▼
                                         ┌────────────────┐
                                         │ Step 4:        │
                                         │ Checkout repo  │
                                         │ destino con    │
                                         │ PAT token      │
                                         └────────────────┘
                                                  │
┌─────────────────────────────────────────────────┼─────────────────────────────┐
│              REPOSITORIO: todas-mis-aplicaciones│                             │
│                  (Repositorio Destino)          │                             │
└─────────────────────────────────────────────────┼─────────────────────────────┘
                                                  │
                                                  ▼
                                         ┌────────────────┐
                                         │ Step 5:        │
                                         │ Crear carpeta  │
                                         │ YYYY-MM-DD/    │
                                         └────────────────┘
                                                  │
                                                  ▼
                                    ┌─────────────────────┐
                                    │ ¿Archivo ya existe? │
                                    └─────────────────────┘
                                    │                     │
                            ✅ Sí │                     │ ❌ No
                                    │                     │
                                    ▼                     ▼
                        ┌───────────────────┐   ┌────────────────┐
                        │ Comparar checksum │   │ Copiar archivo │
                        └───────────────────┘   │ PDF            │
                                    │           └────────────────┘
                        ┌───────────┴───────────┐       │
                        │                       │       │
                ▼                       ▼               │
        ┌──────────┐           ┌──────────┐            │
        │ Idéntico │           │ Diferente│            │
        │ = Skip   │           │ = Backup │            │
        └──────────┘           └──────────┘            │
                │                       │               │
                │                       └───────────────┘
                │                                       │
                │                                       ▼
                │                              ┌────────────────┐
                │                              │ ✅ Verificar   │
                │                              │ integridad MD5 │
                │                              └────────────────┘
                │                                       │
                │                                       ▼
                │                              ┌────────────────┐
                │                              │ Crear README   │
                │                              │ con metadata   │
                │                              └────────────────┘
                │                                       │
                └───────────────┬───────────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │ Step 6:        │
                       │ git add        │
                       │ git commit     │
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │ git push       │
                       │ (con reintentos)│
                       └────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
            ❌ Falla│                       │ ✅ Éxito
                    │                       │
                    ▼                       ▼
            ┌──────────────┐       ┌────────────────┐
            │ Reintento    │       │ Step 7:        │
            │ (hasta 3x)   │       │ Log success    │
            │              │       │ + Resumen      │
            └──────────────┘       └────────────────┘
                    │                       │
                    │                       ▼
                    │              ┌────────────────┐
                    │              │ 📊 GitHub      │
                    │              │ Actions        │
                    │              │ Summary        │
                    │              └────────────────┘
                    │                       │
                    ▼                       ▼
            ┌──────────────┐       ┌────────────────┐
            │ ❌ Error     │       │ ✅ Completado  │
            │ final +      │       │                │
            │ logs         │       │ aplicaciones/  │
            └──────────────┘       │ YYYY-MM-DD/    │
                                   │ ├─ CV.pdf      │
                                   │ └─ README.md   │
                                   └────────────────┘
```

## Leyenda

| Símbolo | Significado |
|---------|-------------|
| ┏━━┓    | Workflow principal |
| ┌──┐    | Paso/Acción |
| ▼       | Flujo secuencial |
| ┼       | Bifurcación/Opciones |
| ✅      | Validación exitosa |
| ❌      | Error o validación fallida |
| 🔍      | Búsqueda/Detección |
| 📊      | Reporte/Resumen |

## Puntos Clave del Flujo

### 1. Detección Automática
- El workflow se activa al completar la generación del PDF
- No requiere intervención manual (también soporta ejecución manual)

### 2. Validaciones Múltiples
- Existencia del archivo
- Tamaño mayor a 0
- Formato PDF válido
- Formato de fecha correcto
- Integridad del archivo (checksum)

### 3. Manejo Inteligente de Duplicados
- Compara checksums
- Skip si es idéntico
- Backup automático si es diferente
- Preserva historial

### 4. Recuperación de Errores
- Reintentos automáticos en push (hasta 3)
- Pull rebase en caso de conflicto
- Mensajes de error claros
- Logs detallados para debugging

### 5. Auditoría Completa
- Logs en cada paso
- Referencias a commits de origen
- Checksums para verificación
- Metadata en README
- Resumen visual en GitHub Actions

## Ejemplo de Ejecución Exitosa

```
🔍 Buscando archivo PDF de la hoja de vida generada...
✅ PDF encontrado: ./output/cv_juan_perez.pdf
📊 Tamaño del archivo: 245678 bytes
✅ Archivo PDF validado correctamente

📅 Fecha extraída del nombre del archivo: 2025-01-15
✅ Fecha validada: 2025-01-15

📁 Creando carpeta para la fecha: 2025-01-15
📄 Copiando PDF: cv_juan_perez.pdf
✅ Archivo copiado y verificado exitosamente
📍 Ubicación: todas-mis-aplicaciones/aplicaciones/2025-01-15/cv_juan_perez.pdf
🔐 Checksum: a1b2c3d4e5f6...
📝 README.md creado

📊 Estado del repositorio:
On branch main
Changes to be committed:
  new file:   aplicaciones/2025-01-15/cv_juan_perez.pdf
  new file:   aplicaciones/2025-01-15/README.md

📝 Archivos a commitear:
aplicaciones/2025-01-15/cv_juan_perez.pdf
aplicaciones/2025-01-15/README.md

⬆️ Enviando cambios al repositorio todas-mis-aplicaciones...
✅ Push exitoso
✅ Proceso completado exitosamente

====================================
✅ PROCESO COMPLETADO EXITOSAMENTE
====================================
Archivo: cv_juan_perez.pdf
Fecha: 2025-01-15
Destino: aplicaciones/2025-01-15/
Estado: Archivo nuevo copiado y commiteado
====================================
```

## Tiempo de Ejecución Estimado

| Fase | Tiempo |
|------|--------|
| Checkout repositorio fuente | ~5s |
| Búsqueda de PDF | ~2s |
| Validaciones | ~1s |
| Checkout repositorio destino | ~5s |
| Copia de archivo | ~1s |
| Commit y Push | ~3s |
| **Total** | **~17s** |

*Nota: Los tiempos pueden variar según el tamaño del archivo y la carga de GitHub Actions.*

## Referencias

- Documentación completa: [README-WORKFLOW.md](README-WORKFLOW.md)
- Guía rápida: [QUICKSTART.md](QUICKSTART.md)
- Resumen de implementación: [IMPLEMENTATION-SUMMARY.md](IMPLEMENTATION-SUMMARY.md)
