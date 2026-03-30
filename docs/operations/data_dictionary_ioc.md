# Diccionario de Datos: Indicadores de Compromiso (IoC)

## 1. Definición de Variables Técnicas

Este diccionario tabula los indicadores detectados para su integración en sistemas de orquestación (SOAR/SIEM).

| Variable ID | Tipo | Valor / Ejemplo | Descripción | Regla de Normalización |
| :--- | :--- | :--- | :--- | :--- |
| `ioc_ip_dest` | Red | `54.227.187.23` | IP del servidor C2 principal. | Validar formato IPv4. |
| `ioc_domain_doh` | Red | `dns.google` | Canal de resolución DNS-over-HTTPS. | Convertir a minúsculas. |
| `ioc_file_dropped` | Archivo | `Tmp26C.tmp` | Archivo temporal soltado en disco. | Prefix `Tmp*` regex. |
| `ioc_path_hidden` | Ruta | `.ms-ad` | Directorio oculto de persistencia. | Absolute path inclusion. |
| `ioc_tls_fingerprint` | Red | `cd08e31494f9...` | JA3 Fingerprint de la comunicación. | 32 caracteres HEX. |
| `ioc_technique_id` | MITRE | `T1055` | ID de la técnica (Process Injection). | MITRE ATT&CK Schema. |

## 2. Mapa de Variables de Automatización

Para flujos de **VMware vRO / Aria Automation**, se proponen las siguientes llaves (Key/Value):

```json
{
  "attackMetadata": {
    "vector": "PDF_EXPLOIT",
    "criticality": "HIGH",
    "indicators": [
      { "key": "remote_host", "value": "54.227.187.23" },
      { "key": "temp_pattern", "value": "Tmp*.tmp" },
      { "key": "persistence_dir", "value": ".ms-ad" }
    ]
  }
}
```

## 3. Reglas de Validación (Gobernanza)

1. **Integridad:** Todos los IoCs deben tener un Hash SHA-256 (Pendiente de obtención en forense).
2. **Contexto:** Se debe asociar la IP con la geolocalización detectada (AWS EC2).
3. **Persistencia:** Cualquier archivo en `.ms-ad` debe ser marcado como `MALICIOUS` automáticamente.
