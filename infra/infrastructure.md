---
title: 0 - Sommaire de mon infrastructure
description: Documentation de la configuration de mon infrastructure personnelle : réseau, pare-feu, reverse proxy, virtualisation, sauvegarde, automatisation et supervision.
published: true
date: 2026-10-08T14:01:06.475Z
tags: infra
editor: markdown
dateCreated: 2026-10-08T13:59:14.141Z
---

# Sommaire de mon infrastructure
Documentation de la configuration de mon infrastructure personnelle : réseau, pare-feu, reverse proxy, virtualisation, sauvegarde, automatisation et supervision. Les adresses, noms de domaine, identifiants et numéros de série sont volontairement omis.

```mermaid
graph TB
  F1["Fibre 1"]
  F2["Fibre 2"]
  SL["Starlink (secours)"]
  RA["Accès distant WireGuard"]
  FW["Pare-feu OPNsense"]
  RP["Reverse proxy NGINX"]
  HV["Hyperviseur Hyper-V<br>9 machines virtuelles"]
  BK["Serveur de sauvegarde<br>Veeam"]
  CK["Supervision Checkmk"]
  AN["Ansible + Semaphore"]
  CL["Backblaze B2"]
  F1 --> FW
  F2 --> FW
  SL -.-> FW
  RA --> FW
  FW --> RP
  RP --> HV
  HV --> BK
  BK --> CL
  CK -.-> HV
  AN -.-> HV
  classDef core fill:#008FA6,stroke:#005F6E,color:#ffffff;
  classDef ext fill:#FFF8D6,stroke:#FCD200,color:#333333;
  classDef svc fill:#F6F8DC,stroke:#B4C000,color:#333333;
  class FW,RP,HV core;
  class F1,F2,SL,RA,CL ext;
  class BK,CK,AN svc;
```

- [Réseau et pare-feu](/fr/infra/infrastructure-network)
- [Reverse proxy NGINX](/fr/infra/infrastructure-reverse-proxy)
- [Virtualisation Hyper-V](/fr/infra/infrastructure-virtualization)
- [Sauvegarde Veeam](/fr/infra/infrastructure-backup)
- [Automatisation : Ansible et Semaphore](/fr/infra/infrastructure-automation)
- [Supervision Checkmk](/fr/infra/infrastructure-monitoring)
