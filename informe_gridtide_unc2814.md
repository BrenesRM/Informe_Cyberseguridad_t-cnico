# Informe Técnico de Arquitectura de Ciberataque

## Malware GRIDTIDE asociado al actor de amenazas UNC2814

**Autor del informe:** Análisis arquitectónico de ciberseguridad\
**Enfoque metodológico:** ISO/IEC/IEEE 42010, Arcadia, ISO/IEC 15288\
**Formato:** Descripción de arquitectura de sistema malicioso

------------------------------------------------------------------------

# 1. Introducción

El presente documento describe la arquitectura técnica del malware
**GRIDTIDE**, asociado al actor de amenazas **UNC2814**. El análisis se
enfoca exclusivamente en comprender el sistema de ataque desde una
perspectiva de **arquitectura de sistemas**, aplicando marcos
metodológicos utilizados en ingeniería de sistemas y arquitectura
empresarial.

El objetivo del documento es modelar el malware como **un sistema
distribuido de operación ofensiva**, analizando sus componentes,
interacciones, flujos de ejecución y dependencias de infraestructura.

Para ello se emplean los siguientes estándares y metodologías:

-   **ISO/IEC/IEEE 42010**: Descripción de arquitecturas de sistemas.
-   **Metodología Arcadia**: Modelado arquitectónico en capas
    (Operational, System, Logical, Physical).
-   **ISO/IEC 15288**: Conceptualización del malware como sistema dentro
    del ciclo de vida de sistemas.

GRIDTIDE representa un ejemplo relevante de **Command and Control
encubierto utilizando servicios cloud legítimos**, específicamente APIs
de servicios de hojas de cálculo en la nube.

Este modelo de arquitectura permite ocultar las comunicaciones de mando
y control dentro de tráfico legítimo, reduciendo la probabilidad de
detección en redes empresariales.

------------------------------------------------------------------------

# 2. Contexto del actor de amenazas UNC2814

UNC2814 es un actor de amenazas identificado en investigaciones de
inteligencia de amenazas. Se caracteriza por operar campañas de
intrusión dirigidas que incluyen:

-   operaciones de espionaje digital
-   acceso persistente en redes corporativas
-   uso de infraestructura cloud como canal de comando y control
-   desarrollo o adaptación de malware personalizado

El malware **GRIDTIDE** se integra dentro de esta estrategia como un
**implante de control remoto** diseñado para operar con bajo perfil y
aprovechar servicios legítimos para coordinar operaciones.

## Características operativas del actor

  Característica   Descripción
  ---------------- ---------------------------------------
  Tipo de actor    Amenaza persistente avanzada (PRC-nexus)
  Objetivo         Espionaje global y acceso persistente
  Alcance          53 víctimas confirmadas en 42 países (>70 países impactados)
  Estrategia       Uso de infraestructura cloud legítima (Google Sheets API)
  Técnica clave    C2 basado en API para mimetizar tráfico benigno

------------------------------------------------------------------------

# 3. Descripción técnica del malware GRIDTIDE

GRIDTIDE es un implante sofisticado desarrollado en **lenguaje C**, diseñado para ejecutar comandos remotos, subir y bajar archivos dentro de sistemas comprometidos.

El elemento distintivo del malware es el uso de **Google Sheets como plataforma C2 de alta disponibilidad**, tratando la hoja de cálculo como un canal de comunicación para transferir datos binarios y comandos shell.

### Especificaciones técnicas de ejecución
- **Cifrado:** Utiliza AES-128 en modo Cipher Block Chaining (CBC).
- **Llave de cifrado:** Requiere una llave criptográfica de 16 bytes presente en un archivo local (ej. `xapt.cfg`) para desencriptar la configuración almacenada en Google Drive.
- **Autenticación:** Emplea Google Service Accounts para autenticarse contra las APIs de Google.
- **Sanitización:** Al iniciar, utiliza el método `batchClear` para limpiar las primeras 1000 filas (A1:Z1000) de la hoja de cálculo, eliminando residuos de sesiones previas.

Este mecanismo permite que el malware consulte periódicamente el documento remoto, interprete los datos como comandos y escriba resultados en el mismo recurso, generando tráfico aparentemente legítimo hacia servicios cloud ampliamente utilizados.

## Funciones principales del malware

  Función                 Descripción
  ----------------------- ------------------------------------------
  Inicialización          Verificación del entorno de ejecución
  Persistencia            Mantenimiento del implante en el sistema
  Comunicación C2         Interacción con servicio cloud
  Ejecución de comandos   Interpretación de instrucciones remotas
  Exfiltración            Transmisión de resultados al atacante

------------------------------------------------------------------------

# 4. Arquitectura del ataque

La arquitectura del ataque puede entenderse como un sistema distribuido
compuesto por los siguientes elementos:

-   operador atacante
-   infraestructura cloud legítima
-   hosts comprometidos
-   implante malware

## Flujo general del ataque

``` mermaid
flowchart LR
A[Operador atacante] --> B[Documento de control en servicio cloud]
B --> C[Malware GRIDTIDE]
C --> D[Host comprometido]
D --> C
C --> B
B --> A
```

Este modelo permite desacoplar el operador atacante de la
infraestructura comprometida.

------------------------------------------------------------------------

# 5. Modelado con metodología Arcadia

Arcadia divide el análisis arquitectónico en cuatro niveles principales:

1.  Operational Architecture
2.  System Architecture
3.  Logical Architecture
4.  Physical Architecture

------------------------------------------------------------------------

# 5.1 Operational Architecture

La arquitectura operacional describe **quién interactúa con el sistema y
con qué objetivos**.

## Actores

  Actor               Rol
  ------------------- -------------------------------
  Operador atacante   Controla la operación
  Malware GRIDTIDE    Agente de ejecución
  Host comprometido   Entorno de ejecución
  Servicio cloud      Intermediario de comunicación

## Objetivos operacionales

-   establecer control remoto encubierto
-   mantener persistencia en sistemas críticos
-   ejecutar comandos arbitrarios de shell
-   recolectar Información de Identificación Personal (PII): nombres, teléfonos, fechas de nacimiento, IDs nacionales y números de votante.

## Interacciones operacionales

``` mermaid
flowchart TD

Attacker[Operador atacante]
Cloud[Servicio Cloud]
Malware[GRIDTIDE]
Host[Host comprometido]

Attacker --> Cloud
Cloud --> Malware
Malware --> Host
Host --> Malware
Malware --> Cloud
Cloud --> Attacker
```

------------------------------------------------------------------------

# Diagrama de flujo operacional

``` mermaid
sequenceDiagram
participant A as Operador
participant C as Servicio Cloud
participant M as GRIDTIDE
participant H as Host

A->>C: Inserta comandos
M->>C: Consulta documento
C->>M: Devuelve comandos
M->>H: Ejecuta instrucciones
H->>M: Resultados
M->>C: Escribe resultados
A->>C: Recupera información
```

------------------------------------------------------------------------

# 5.2 System Architecture

La arquitectura de sistema modela el malware como **un sistema
funcional**.

## Funciones del sistema

  Función               Descripción
  --------------------- ---------------------------------
  Masquerading          Binario `/var/tmp/xapt` (mimetiza herramienta de Debian)
  Polling C2            Consulta a celdas específicas en Google Sheets
  Parsing de comandos   Interpretación de sintaxis estructurada
  Ejecución local       Comandos shell con privilegios escalados
  Persistencia          Servicio systemd en `/etc/systemd/system/xapt.service`
  Lateral Movement      Uso de SoftEther VPN Bridge para túneles cifrados
  Exfiltración          Escritura en fragmentos de 45-KB en celdas de Sheets

## Cadena de ataque

``` mermaid
flowchart LR

Initial[Acceso inicial]
Install[Instalación malware]
Persist[Persistencia]
C2[Conexión C2]
Execute[Ejecución comandos]
Exfil[Exfiltración datos]

Initial --> Install
Install --> Persist
Persist --> C2
C2 --> Execute
Execute --> Exfil
```

------------------------------------------------------------------------

# Flujo de ejecución del malware

``` mermaid
flowchart TD

Start[Inicio malware]
Check[Verificación entorno]
Connect[Conexión servicio cloud]
Fetch[Lectura comandos]
Parse[Interpretación]
Exec[Ejecución]
Send[Enviar resultados]

Start --> Check
Check --> Connect
Connect --> Fetch
Fetch --> Parse
Parse --> Exec
Exec --> Send
Send --> Fetch
```

------------------------------------------------------------------------

# 5.3 Logical Architecture

La arquitectura lógica describe **los módulos internos del malware**.

## Componentes internos

  Módulo                Función
  --------------------- ---------------------------------
  Core Engine           Control de flujo
  C2 Client             Comunicación con servicio cloud
  Command Processor     Interpretación de comandos
  Persistence Manager   Mecanismo de persistencia
  Data Collector        Recolección de información

## Arquitectura de componentes

``` mermaid
flowchart TD

Core[Core Engine]
C2[C2 Client]
Cmd[Command Processor]
Persist[Persistence Module]
Data[Data Collector]

Core --> C2
Core --> Cmd
Core --> Persist
Cmd --> Data
C2 --> Core
```

------------------------------------------------------------------------

# Motor de comandos

El motor de comandos analiza datos obtenidos del documento remoto mediante una sintaxis de cuatro partes: `<type>-<command_id>-<arg_1>-<arg_2>`.

**Tipos de comandos:**
- **C (Command):** Ejecuta comandos Bash codificados en Base64.
- **U (Upload):** Sube datos desde celdas A2:An al host víctima.
- **D (Download):** Lee archivos locales y los transfiere a la hoja de cálculo en fragmentos.

**Mecanismo de Polling:**
- **Celda A1:** El malware sondea esta celda por nuevos comandos y escribe el estado de respuesta (ej. `S-C-R`).
- **Control de ruido:** Realiza 120 intentos con intervalos de 1s; si no hay comandos, cambia a un sleep aleatorio de 5 a 10 minutos para evadir detecciones basadas en beaconing constante.
- **Celda V1:** Almacena metadatos del sistema (hostname, usuario, IP local, OS, timezone).
- **Celdas A2-An:** Canal de transferencia de datos crudos (upload/download).

------------------------------------------------------------------------

# Módulo de comunicación con API cloud

El cliente C2 implementa:

-   autenticación con API
-   consultas periódicas
-   lectura de datos
-   escritura de resultados

Las comunicaciones se realizan mediante solicitudes HTTPS legítimas.

------------------------------------------------------------------------

# Módulo de persistencia

El módulo de persistencia garantiza que el malware continúe ejecutándose
tras reinicios del sistema.

Su responsabilidad dentro de la arquitectura es mantener la
disponibilidad del implante.

------------------------------------------------------------------------

# Módulo de exfiltración

La exfiltración se realiza reutilizando el mismo canal de comunicación
utilizado para recibir comandos.

Los resultados se insertan en el documento remoto y posteriormente son
recuperados por el operador.

------------------------------------------------------------------------

# 5.4 Physical Architecture

La arquitectura física describe la infraestructura real utilizada.

## Componentes físicos

  Componente              Descripción
  ----------------------- --------------------------
  Estación del atacante   Interfaz de operación
  Servicio cloud          Infraestructura legítima
  Hosts comprometidos     Sistemas víctimas
  Internet                Canal de transporte

## Arquitectura de infraestructura

``` mermaid
flowchart LR

Operator[Operador atacante]
Cloud[Servicio Cloud]
Internet[Internet]
Victim[Host comprometido]

Operator --> Cloud
Cloud --> Internet
Internet --> Victim
Victim --> Internet
Internet --> Cloud
```

------------------------------------------------------------------------

# Arquitectura de comando y control

``` mermaid
flowchart LR

Attacker[Operador]
Sheet[Documento cloud]
Malware[GRIDTIDE]

Attacker --> Sheet
Malware --> Sheet
Sheet --> Malware
Malware --> Sheet
Sheet --> Attacker
```

------------------------------------------------------------------------

# 6. Protocolo de Command and Control

El protocolo C2 se basa en un mecanismo simple:

1.  el operador inserta instrucciones
2.  el malware consulta periódicamente el documento
3.  el malware interpreta el contenido
4.  el malware ejecuta acciones
5.  los resultados se escriben nuevamente en el documento

Este diseño tiene varias propiedades arquitectónicas:

-   desacoplamiento entre operador y víctima
-   uso de infraestructura legítima
-   tráfico cifrado estándar

------------------------------------------------------------------------

# 7. Mecanismos de evasión de detección

La arquitectura del malware incluye elementos diseñados para reducir su
visibilidad:

-   uso de tráfico HTTPS legítimo
-   dependencia de servicios cloud ampliamente utilizados
-   patrón de comunicación similar a aplicaciones legítimas
-   ausencia de infraestructura C2 dedicada visible

Estos factores dificultan diferenciar el tráfico del malware del tráfico
normal de aplicaciones cloud.

------------------------------------------------------------------------

# 8. Mapeo MITRE ATT&CK

Para estandarizar el análisis desde una perspectiva de ciberseguridad, se mapean las técnicas observadas en GRIDTIDE con el marco MITRE ATT&CK.

| Táctica | Técnica | ID | Descripción en GRIDTIDE |
| :--- | :--- | :--- | :--- |
| **Persistencia** | Systemd Service | T1543.002 | Crea el archivo `/etc/systemd/system/xapt.service` para persistencia. |
| **Evasión de Defensa** | Web Service | T1102 | Utiliza Google Sheets API para ocultar tráfico C2. |
| **Evasión de Defensa** | Masquerading | T1036.005 | El binario se nombra `xapt` para imitar una herramienta legítima de Debian. |
| **Movimiento Lateral** | Remote Services (SSH) | T1021.004 | Uso de SSH y cuentas de servicio para moverse lateralmente. |
| **Comando y Control** | Application Layer Protocol | T1071.001 | Solicitudes HTTPS HTTPS a APIs cloud. |
| **Comando y Control** | Data Encoding | T1132 | Codificación URL-safe Base64 para datos en celdas. |
| **Exfiltración** | Exfiltration Over C2 Channel | T1041 | Los datos robados se suben a las celdas A2:An del documento C2. |

------------------------------------------------------------------------

# 10. Detección y Respuesta

La respuesta de Google Threat Intelligence Group (GTIG) incluyó la terminación de proyectos de Google Cloud controlados por el atacante y la desactivación de cuentas de servicio.

## Consultas de Detección (Google SecOps UDM)

### Conexiones de API sospechosas
```udm
target.url = /sheets\.googleapis\.com/ ( target.url = /batchClear/ OR target.url = /batchUpdate/ OR target.url = /valueRenderOption=FORMULA/ ) principal.process.file.full_path != /chrome|firefox|safari|msedge/
```

### Ejecución de Shell en /var/tmp/
```udm
principal.process.file.full_path = /^\/var\/tmp\/[a-z0-9]{1,10}$/ nocase AND target.process.file.full_path = /\b(ba)?sh$/ nocase
```

------------------------------------------------------------------------

# 11. Monitoreo con Machine Learning (ML)

Para organizaciones con capacidades de análisis avanzado, se recomienda implementar modelos de ML enfocados en dos áreas críticas:

## 11.1 Detección de Anomalías en Tráfico de APIs Cloud
- **Baseline:** Modelar el comportamiento normal de uso de APIs (Google Sheets, Drive) por usuario y aplicación.
- **Anomalía:** Detectar picos de uso del método `batchUpdate` o `batchClear` desde procesos que no son navegadores o herramientas conocidas de BI/ETL.
- **Patrón temporal:** Identificar el patrón de polling (1s de intervalo seguido de silencios de 5-10m) que difiere del uso humano típico.

## 11.2 Análisis Comportamental del Endpoint
- **Anomalía de Proceso:** Detectar binarios con nombres cortos y aleatorios ejecutando comandos de reconocimiento (`sh -c id`).
- **Relaciones Padre-Hijo:** Alertar sobre servicios `systemd` recién creados que activan shells con privilegios de root.

------------------------------------------------------------------------

# 12. Reglas de Detección de Red e Implementación

## 12.1 Reglas IPS (Snort/Suricata)
```snort
alert tcp $HOME_NET any -> $EXTERNAL_NET 443 (msg:"GRIDTIDE C2 Check-in (Sheets API)"; content:"sheets.googleapis.com"; content:"valueRenderOption=FORMULA"; content:"A1"; sid:1000001; rev:1;)
alert tcp $HOME_NET any -> $EXTERNAL_NET 443 (msg:"GRIDTIDE Data Exfiltration (Sheets API)"; content:"sheets.googleapis.com"; content:"batchUpdate"; pcre:/(A2|V1)/; sid:1000002; rev:1;)
```

## 12.2 Regla YARA de Google (GTIG)
```yara
rule G_APT_Backdoor_GRIDTIDE_1 {
    meta:
        author = "Google Threat Intelligence Group (GTIG)"
    strings:
        $s1 = { 7B 22 61 6C 67 22 3A 22 52 53 32 35 36 22 2C 22 6B 69 64 22 3A 22 25 73 22 2C 22 74 79 70 22 3A 22 4A 57 54 22 7D 00 }
        $s2 = { 2F 70 72 6F 63 2F 73 65 6C 66 2F 65 78 65 00 }
        $s3 = { 7B 22 72 61 6E 67 65 73 22 3A 5B 22 61 31 3A 7A 31 30 30 30 22 5D 7D 00 }
        $s4 = { 53 2D 55 2D 25 73 2D 31 00 }
        $s5 = { 53 2D 55 2D 52 2D 31 00 }
        $s6 = { 53 2D 44 2D 25 73 2D 30 00 }
        $s7 = { 53 2D 44 2D 52 2D 25 64 00 }
    condition:
        (uint32(0) == 0x464c457f) and 6 of ($*)
}
```

------------------------------------------------------------------------

# 13. Guía de Implementación Organizacional

Para proteger una organización contra GRIDTIDE y actores similares (UNC2814):

1.  **Restricción de APIs:** Limitar el uso de Google Service Accounts a proyectos y usuarios autorizados. Implementar políticas de "Context-Aware Access".
2.  **Hardening de Endpoints:** Monitorear la creación de servicios en `/etc/systemd/system/` y el uso de directorios `/var/tmp/` para ejecución.
3.  **Filtrado de Red:** Inspeccionar tráfico HTTPS (SSL Decryption) para buscar User-Agents sospechosos como `Google-API-Java-Client/2.0.0`.
4.  **Monitoreo de VPNs:** Alertar sobre despliegues no autorizados de SoftEther VPN o cualquier puente VPN que inicie conexiones salientes persistentes.

------------------------------------------------------------------------

# 14. Indicadores de Compromiso (IoCs)

| Tipo | Descripción | Valor |
| :--- | :--- | :--- |
| **Hash (SHA256)** | Malware GRIDTIDE | `ce36a5fc44cbd7de947130b67be9e732a7b4086fb1df98a5afd724087c973b47` |
| **Hash (SHA256)** | Key file (`xapt.cfg`) | `01fc3bd5a78cd59255a867ffb3dfdd6e0b7713ee90098ea96cc01c640c6495eb` |
| **IP** | C2 Infra (Hosting archives) | `130.94.6.228` |
| **IP** | SoftEtherVPN Server | `38.60.194.21` |
| **Dominio** | C2 Domain | `1cv2f3d5s6a9w.ddnsfree.com` |
| **Dominio** | C2 Domain | `admina.freeddns.org` |
| **User-Agent** | GRIDTIDE UA | `Google-HTTP-Java-Client/1.42.3 (gzip)` |

---

# 9. Secuencia completa del ataque

``` mermaid
flowchart TD

A[Compromiso inicial]
B[Instalación GRIDTIDE]
C[Establecimiento persistencia]
D[Consulta servicio cloud]
E[Recepción comandos]
F[Ejecución local]
G[Recolección datos]
H[Escritura resultados]

A --> B
B --> C
C --> D
D --> E
E --> F
F --> G
G --> H
H --> D
```

------------------------------------------------------------------------

# Conclusión

GRIDTIDE representa una arquitectura de malware moderna y altamente eficaz basada en el uso de servicios cloud legítimos como canal de comando y control. 

Desde la perspectiva de arquitectura de sistemas, el implante es un **sistema distribuido resiliente** que aprovecha la confianza implícita en infraestructuras de terceros. El uso de metodologías como **Arcadia** y el mapeo con **MITRE ATT&CK** permite diseccionar la amenaza no solo como un archivo malicioso, sino como una pieza de ingeniería diseñada para objetivos estratégicos de espionaje.

El resultado es una comprensión clara del sistema de ataque, su flujo de operación y su arquitectura interna, facilitando el diseño de defensas más robustas basadas en el comportamiento del sistema.
