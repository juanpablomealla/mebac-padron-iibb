# MEBAC Padron IIBB Jujuy

Repositorio de descarga automatica del Padron de Percepciones IIBB de Jujuy.

## Proposito

Este repositorio sirve como **proxy de descarga** entre Rentas Jujuy y el servidor WordPress.

GitHub Actions descarga automaticamente el padron desde Rentas Jujuy y lo publica como Release. El servidor WordPress descarga desde aqui en lugar de directo a Rentas, evitando:

- **IP baneada** por WAF de Rentas
- **Rate limiting** en descargas repetidas
- **Bloqueos geograficos** por IP del servidor

## Arquitectura

```
┌─────────────────────┐    ┌───────────────────┐    ┌─────────────────────┐
│ GitHub Actions      │ -> │ Rentas Jujuy      │ -> │ GitHub Release      │
│ (cron 2:30 AM ARG)  │    │ (descarga RAR)    │    │ (RAR + metadata)    │
└─────────────────────┘    └───────────────────┘    └─────────────────────┘
                                                             │
                                                             v
                                                    ┌─────────────────────┐
                                                    │ WordPress Server    │
                                                    │ (cron 3:00 AM ARG)  │
                                                    └─────────────────────┘
```

## Ejecucion Automatica

| Campo | Valor |
|-------|-------|
| **Cuando** | Dia 1 de cada mes a las 2:30 AM (Argentina) |
| **Que hace** | Descarga el padron del mes actual desde Rentas |
| **Resultado** | Crea un Release con el RAR y metadata.json |

El cron de WordPress esta configurado para las 3:00 AM, 30 minutos despues, asegurando que el Release ya este disponible.

## Ejecucion Manual

Para descargar un periodo especifico:

1. Ir a **Actions** > **Descarga Padron IIBB Jujuy**
2. Click en **Run workflow**
3. Opcionalmente especificar ano/mes (dejar vacio para mes actual)
4. Click en **Run workflow** (boton verde)

## Releases

Cada release contiene dos archivos:

### 1. Archivo RAR del Padron
- **Nombre**: `PADRONRETPERYYMM.rar` (ej: `PADRONRETPER202602.rar`)
- **Contenido**: Padron de percepciones IIBB en formato RAR

### 2. Metadata JSON
- **Nombre**: `metadata.json`
- **Contenido**: Informacion sobre la descarga

```json
{
  "periodo": "2026-02",
  "archivo": "PADRONRETPER202602.rar",
  "tamano_bytes": 12345678,
  "descarga_utc": "2026-02-01T05:30:00Z",
  "descarga_argentina": "2026-02-01 02:30:00",
  "fuente": "rentasjujuyonline.gob.ar",
  "workflow_run": "123456789",
  "commit": "abc123def456..."
}
```

### Tags
- **Formato**: `padron-YYYY-MM` (ej: `padron-2026-02`)

## URLs de Descarga Directa

### RAR del Padron
```
https://github.com/juanpablomealla/mebac-padron-iibb/releases/download/padron-YYYY-MM/PADRONRETPERYYMM.rar
```

### Metadata
```
https://github.com/juanpablomealla/mebac-padron-iibb/releases/download/padron-YYYY-MM/metadata.json
```

### Ejemplo (Febrero 2026)
```bash
# RAR
https://github.com/juanpablomealla/mebac-padron-iibb/releases/download/padron-2026-02/PADRONRETPER202602.rar

# Metadata
https://github.com/juanpablomealla/mebac-padron-iibb/releases/download/padron-2026-02/metadata.json
```

## Integracion con WordPress

El plugin `mebac-step-checkout` usa este repositorio como fuente primaria:

```php
// En class-padron-downloader.php
private const GITHUB_REPO = 'juanpablomealla/mebac-padron-iibb';

// Flujo de descarga:
// 1. Intenta GitHub Releases
// 2. Si falla, fallback a Rentas Jujuy directo
// 3. Descarga metadata.json para fecha original
```

## Seguridad

- **Solo lectura**: Repositorio publico, pero solo el owner puede modificar
- **Releases protegidos**: Solo el workflow puede crear releases
- **Sin datos sensibles**: Solo contiene el padron publico de Rentas

## Mantenimiento

### Verificar Estado
```bash
# Ver ultimo workflow
gh run list --limit 5

# Ver releases
gh release list --limit 5
```

### Re-ejecutar Descarga Fallida
```bash
# Ejecutar para un periodo especifico
gh workflow run padron-iibb-download.yml -f year=2026 -f month=02
```

### Borrar Release (para re-descargar)
```bash
gh release delete padron-2026-02 --yes
```

---

Mantenido por [MEBAC](https://mebac.com.ar)
