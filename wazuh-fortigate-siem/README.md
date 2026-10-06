# Wazuh SIEM + FortiGate Integration Lab

Laboratorio hands-on de integración de un SIEM (Wazuh) con un firewall FortiGate dentro de un entorno virtualizado con GNS3.

## Contenido

- `reporte-fuerza-bruta-ssh.md` — Detección de un ataque de fuerza bruta SSH simulado contra FortiGate, capturado y clasificado por Wazuh.
- `screenshots/` — Evidencia visual del laboratorio y la detección.

## Stack utilizado

- GNS3 (topología de red)
- FortiGate-VM
- Wazuh (manager + indexer + dashboard) sobre Ubuntu Server
- Kali Linux (simulación de ataque con Hydra)

## Técnica simulada

MITRE ATT&CK T1110 — Brute Force
