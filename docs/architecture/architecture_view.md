# Arquitectura de Análisis de Ataque PDF (ISO 42010 / ARCADIA)

## 1. Identificación de la Arquitectura (ISO 42010)

| Stakeholder | Concern | Role |
| :--- | :--- | :--- |
| **SOC Manager** | Impacto de negocio y tiempo de respuesta. | Auditor / Decisor |
| **Arquitecto de Seguridad** | Identificación de fallos en controles preventivos. | Diseñador |
| **Operaciones TI (N1/N2)** | Procedimientos de remediación y aislamiento. | Ejecutor |
| **Compliance / Auditoría** | Trazabilidad del ataque y cumplimiento de políticas. | Verificador |

---

## 2. Niveles de Análisis ARCADIA

### A. Operational Analysis (OA)
**Objetivo:** Comprender las actividades de los usuarios y las necesidades operativas.
- **Actores:** Empleado (Víctima), Atacante Externo, Analista SOC.
- **Proceso Operativo:** recepción de correo electrónico -> apertura de documento PDF "legítimo" -> activación de respuesta a incidentes.

### B. System Analysis (SA)
**Objetivo:** Definir qué hace el sistema para mitigar o permitir el ataque.
- **Capacidades del Sistema:** 
    - Explotación de vulnerabilidad en Adobe Reader (T1203).
    - Ejecución de comandos en memoria para evitar firmas de disco.
    - Detección de sandboxes para evadir análisis automatizado.

### C. Logical Architecture (LA)
**Objetivo:** Descomposición funcional de la cadena de ataque.
- **Componentes Lógicos:**
    - **Exploit Engine:** Gatillado por el PDF malicioso.
    - **Reconnaissance Module:** Consultas WMI para perfilado del sistema.
    - **Persistence Manager:** DLL Side-loading para supervivencia tras reinicio.
    - **Evasion Module:** Rootkit y limpieza de artefactos.

### D. Physical Architecture (PA)
**Objetivo:** Implementación física y recursos específicos.
- **Indicadores Físicos (IoCs):**
    - **Archivos:** `Tmp26C.tmp`, `SOPHIA.json`.
    - **Directorios:** `C:\Users\user\.ms-ad`.
    - **Red:** `54.227.187.23`, `dns.google` (DoH).
    - **Procesos:** Inyección en procesos legítimos de Adobe.

---

## 3. Decisiones Arquitectónicas (ADR)

### ADR-001: Aislamiento Inmediato de Host
- **Contexto:** El malware presenta capacidades de persistencia (Bootkit) y robo de credenciales (LSASS).
- **Decisión:** Priorizar el aislamiento total del host sobre la limpieza parcial.
- **Consecuencia:** Previene el movimiento lateral (T1021).

### ADR-002: Deshabilitación de JavaScript en PDF
- **Contexto:** El vector de entrada es un exploit de Adobe via PDF.
- **Decisión:** Implementar política de GPO para deshabilitar JS en lectores de PDF corporativos.
- **Consecuencia:** Reduce la superficie de ataque inicial.
