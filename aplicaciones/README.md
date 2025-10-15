# Aplicaciones Laborales

Esta carpeta contiene las hojas de vida (CV) en formato PDF organizadas por fecha de aplicación.

## Estructura

Cada subcarpeta representa una aplicación laboral con el formato `YYYY-MM-DD`:

```
aplicaciones/
├── 2025-01-15/
│   ├── cv_nombre.pdf
│   └── README.md
├── 2025-01-20/
│   ├── cv_nombre.pdf
│   └── README.md
└── ...
```

## Contenido de cada carpeta

- **PDF de Hoja de Vida**: El documento principal de la aplicación
- **README.md**: Metadata de la aplicación (fecha, workflow ID, etc.)

## Automatización

Los archivos se agregan automáticamente mediante el workflow CI/CD desde el repositorio `aplicaciones_laborales`.

Para más información, consulta [README-WORKFLOW.md](../README-WORKFLOW.md).
