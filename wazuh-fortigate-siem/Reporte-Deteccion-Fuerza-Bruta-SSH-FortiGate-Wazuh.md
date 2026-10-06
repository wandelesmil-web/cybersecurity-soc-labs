# Reporte de Detección: Fuerza Bruta SSH contra FortiGate

**Autor:** Wandel Esmil Luna Victorino
**Proyecto:** Laboratorio SIEM — Integración Wazuh + FortiGate (GNS3)
**Fecha del ejercicio:** Octubre 2026
**Categoría MITRE ATT&CK:** T1110 — Brute Force

---

## 1. Objetivo

Simular un ataque de fuerza bruta SSH contra un firewall FortiGate dentro de un laboratorio aislado, con el fin de validar la capacidad de detección de un SIEM (Wazuh) correctamente integrado vía syslog, y documentar el comportamiento defensivo nativo del dispositivo atacado.

## 2. Arquitectura del laboratorio

| Componente | Rol | Dirección IP |
|---|---|---|
| Router Cisco IOSv | Gateway del segmento | 192.168.150.1 |
| FortiGate-VM64-KVM | Firewall / objetivo del ataque | 192.168.150.2 |
| Wazuh Manager (Ubuntu 24.04) | SIEM — recepción y análisis de logs | 192.168.150.10 |
| Kali Linux | Atacante simulado | 192.168.150.20 |

Todos los nodos conectados a un segmento aislado (`192.168.150.0/24`) sobre GNS3, con salida controlada hacia VMware (VMnet host-only).

**Integración FortiGate → Wazuh:** configurada vía syslog (UDP/514), formato nativo Fortinet, confirmada mediante captura de tráfico (`tcpdump`) y verificación en el índice `wazuh-alerts-*`.

## 3. Herramienta y metodología del ataque

- **Herramienta:** Hydra v9.7
- **Vector:** SSH (puerto 22)
- **Usuario objetivo:** `admin`
- **Diccionario:** lista de 7 contraseñas comunes, incluyendo la contraseña real como última entrada, para forzar un resultado positivo tras varios intentos fallidos

**Comando ejecutado:**
```bash
hydra -l admin -P /home/kali/passwords.txt ssh://192.168.150.2
```

**Resultado del ataque:**
```
[22][ssh] host: 192.168.150.2   login: admin   password: ****
1 of 1 target successfully completed, 1 valid password found
```

## 4. Hallazgos en Wazuh

### 4.1 Eventos individuales de intentos fallidos

Cada intento incorrecto quedó registrado de forma individual, con origen correctamente identificado:

```
data.msg: Administrator admin login failed from ssh(192.168.150.20) because of invalid password
data.reason: passwd_invalid
data.srcip: 192.168.150.20
data.devname: FortiGate-VM64-KVM
```

### 4.2 Respuesta automática del FortiGate (control nativo)

Tras 3 intentos fallidos consecutivos, el FortiGate activó un bloqueo temporal sobre la IP de origen:

```
data.msg: Login disabled from IP 192.168.150.20 for 60 seconds because of 3 bad attempts
data.reason: exceed_limit
data.level: alert
data.action: login
```

Este evento fue clasificado por Wazuh con nivel **`alert`**, a diferencia de los intentos individuales (nivel `information`/`low`), lo que confirma una correlación efectiva: el sistema no solo registra cada intento, sino que distingue cuándo un patrón cruza el umbral de un comportamiento malicioso.

## 5. Análisis como analista SOC

| Pregunta | Respuesta |
|---|---|
| ¿El ataque fue detectado? | Sí, en tiempo real, vía syslog hacia Wazuh |
| ¿Hubo respuesta automática? | Sí — bloqueo temporal de 60s por parte del FortiGate (control nativo de fuerza bruta) |
| ¿El ataque tuvo éxito final? | Sí, la contraseña correcta fue encontrada tras el período de bloqueo |
| ¿Qué acción recomendaría un analista? | Bloqueo permanente o de mayor duración de la IP origen; forzar rotación de credenciales; evaluar MFA para acceso administrativo; crear regla de correlación en Wazuh para escalar automáticamente ante 3+ eventos `exceed_limit` en una ventana de tiempo corta |

## 6. Técnica MITRE ATT&CK

- **Técnica:** T1110 — Brute Force
- **Sub-técnica aplicable:** T1110.001 — Password Guessing

## 7. Conclusión

El ejercicio valida que la integración FortiGate → Wazuh vía syslog es funcional de punta a punta: desde la generación del evento en el dispositivo de borde, pasando por el transporte syslog, hasta la decodificación y clasificación de severidad en el SIEM. Esto demuestra capacidad práctica para configurar, operar e interpretar un pipeline de detección real, aplicable directamente a un rol de SOC Analyst Level 1 o Network Security Analyst.

---

*Nota: las contraseñas reales usadas en el ejercicio fueron omitidas de este reporte por motivos de seguridad del laboratorio.*
