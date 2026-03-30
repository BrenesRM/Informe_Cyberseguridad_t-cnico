# SOP: Respuesta Operativa ante Ataque PDF (T1203/T1055)

## 1. Identificación y Alcance
Este Procedimiento Operativo Estándar (SOP) describe las acciones inmediatas para analistas de SOC de nivel N1 y N2 al detectar actividad relacionada con el informe de ataque PDF.

| Nivel de Riesgo | Criticidad | Tiempo de Respuesta (SLA) |
| :--- | :--- | :--- |
| **CRÍTICO** | Alta | < 30 minutos |

## 2. Flujo de Respuesta (Paso a Paso)

### Paso 1: Aislamiento de Red
- **Acción:** Isolar el host afectado inmediatamente.
- **Técnica:** Bloquear comunicación en el EDR o desconectar cable de red/vNIC.
- **Propósito:** Prevenir el movimiento lateral (Lateral Movement).

### Paso 2: Bloqueo de Infraestructura C2
- **Acción:** Bloquear los siguientes indicadores en firewalls perimetrales (Palo Alto/Fortinet/Azure FW).
- **Indicadores de Red:** 
  - `54.227.187.23`
  - `23.35.76.119`
  - `dns.google` (Si el DoH es inusual en el segmento).

### Paso 3: Kill de Procesos y Purga de Memoria
- **Acción:** Terminar procesos de Adobe Reader y cualquier proceso inyectado sospechoso (`RuntimeBroker.exe`, `svchost.exe` con alta actividad WMI).
- **Comando PowerShell (Audit-ready):**
  ```powershell
  Stop-Process -Name "AcroRd32" -Force
  Get-WmiObject Win32_Process | Where-Object { $_.CommandLine -like "*Tmp*" } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
  ```

### Paso 4: Evidencia y Forense
- **Acción:** Recolectar el directorio `.ms-ad` y archivos temporales `Tmp*.tmp`.
- **Acción:** Realizar volcado de memoria (Memory Dump) antes de reiniciar el equipo si es posible.

## 3. Guía de Prevención y Parcheo
- Aplicar parches de seguridad de Adobe Acrobat (CVE correspondiente al exploit T1203).
- Implementar Reglas de Reducción de Superficie de Ataque (ASR) en Defender: "Bloquear aplicaciones Office de crear procesos secundarios".

## 4. Registro de Auditoría
- Documentar todas las acciones en el ticket de incidencia (ServiceNow/Jira).
- Referenciar este SOP como la base del procedimiento de respuesta.
