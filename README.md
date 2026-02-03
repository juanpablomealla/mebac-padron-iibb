# MEBAC Padrón IIBB Jujuy

Repositorio de descarga automática del Padrón de Percepciones IIBB de Jujuy.

## Propósito

Este repositorio descarga automáticamente el padrón de percepciones de Ingresos Brutos desde Rentas Jujuy y lo publica como Release.

El servidor WordPress descarga desde aquí en lugar de directo a Rentas Jujuy, evitando problemas de:
- IP baneada por WAF
- Rate limiting
- Bloqueos geográficos

## Funcionamiento

```
┌─────────────────┐    ┌───────────────┐    ┌─────────────────┐
│ GitHub Actions  │ -> │ Rentas Jujuy  │ -> │ GitHub Release  │
│ (cron mensual)  │    │ (descarga RAR)│    │ (publica RAR)   │
└─────────────────┘    └───────────────┘    └─────────────────┘
                                                    │
                                                    v
                                           ┌─────────────────┐
                                           │ WordPress       │
                                           │ (descarga RAR)  │
                                           └─────────────────┘
```

## Ejecución automática

El workflow se ejecuta automáticamente:
- **Cuándo:** Día 1 de cada mes a las 3:00 AM (Argentina)
- **Qué hace:** Descarga el padrón del mes actual y crea un Release

## Ejecución manual

Para descargar un período específico:

1. Ir a **Actions** → **Descarga Padrón IIBB Jujuy**
2. Click en **Run workflow**
3. Opcionalmente especificar año/mes
4. Click en **Run workflow** (botón verde)

## Releases

Cada release contiene:
- **Tag:** `padron-YYYY-MM` (ej: `padron-2026-02`)
- **Archivo:** `PADRONRETPERYYMM.rar` (ej: `PADRONRETPER202602.rar`)

### URL de descarga directa

```
https://github.com/USUARIO/REPO/releases/download/padron-YYYY-MM/PADRONRETPERYYMM.rar
```

Ejemplo:
```
https://github.com/juanpablomealla/mebac-padron-iibb/releases/download/padron-2026-02/PADRONRETPER202602.rar
```

## Configuración inicial

1. Crear este repo como **público**
2. Copiar el archivo `.github/workflows/padron-iibb-download.yml`
3. Ejecutar el workflow manualmente para el mes actual

## Integración con WordPress

El plugin `mebac-step-checkout` está configurado para:
1. Primero intentar descargar desde este repo
2. Si falla, fallback a Rentas Jujuy directo

Configuración en `class-padron-downloader.php`:
```php
private const GITHUB_REPO = 'juanpablomealla/mebac-padron-iibb';
```

---

Mantenido por [MEBAC](https://mebac.com.ar)
