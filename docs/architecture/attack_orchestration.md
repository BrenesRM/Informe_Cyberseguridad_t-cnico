# Orquestación del Ataque PDF (Visualización)

## 1. Cadena de Matanza (Cyber Kill Chain)

```mermaid
graph TD
    A[Inicio: Correo Phishing] -->|Vector: PDF| B(Malicious PDF Execution)
    B --> C{Exploit Adobe}
    C -->|Success| D[Memory Payload Injection]
    D --> E[Reconocimiento: WMI]
    E --> F[Persistencia: DLL Sideloading]
    F --> G[C2 Communication: 54.227.187.23]
    G --> H[Evasión: Cleanup Files]
```

## 2. Flujo Lógico de Ejecución de Malware

```mermaid
sequenceDiagram
    participant User
    participant PDF as Adobe Reader
    participant OS as Sistema Operativo
    participant C2 as Servidor C2 (Atacante)

    User->>PDF: Abre PDF malicioso
    PDF->>PDF: Ejecuta exploit en memoria
    PDF->>OS: Dropped: Tmp26C.tmp
    OS->>OS: Inyección en Proceso Legítimo
    OS->>OS: Consulta WMI (System Info)
    OS->>OS: Creación directorio .ms-ad
    OS->>C2: Conexión HTTPS (JA3: cd08e314...)
    C2-->>OS: Comandos/Exfiltración
    OS->>OS: Eliminación de Temporales
```

## 3. Matriz de Orquestación

| Etapa | Artefacto de Orquestación | Acción Técnica |
| :--- | :--- | :--- |
| **Entrada** | Malicious PDF | Desbordamiento de búfer / Scripting |
| **Profilado** | WMI Scripts | `Get-WmiObject -Class Win32_Process` |
| **Persistencia** | DLL Wrapper | Sideloading via `SOPHIA.json` |
| **Evasión** | Purge Action | `del Tmp*.tmp` |
