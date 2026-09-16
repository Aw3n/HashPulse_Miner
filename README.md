# HashPulse

**Mineur Xelis — CPU & GPU (OpenCL) — XelisHash V3**

![version](https://img.shields.io/badge/version-1.0.0-00c89b)
![support](https://img.shields.io/badge/CPU%20%2B%20GPU-00d4ff)
![platforms](https://img.shields.io/badge/Windows%20%2B%20Linux-9cf)
![dev tax](https://img.shields.io/badge/Dev%20Tax-1%25-ffa500)

HashPulse est un mineur Xelis performant supportant à la fois le **CPU** et le **GPU (OpenCL)**, l'algorithme **XelisHash V3**, le minage **solo** (daemon) et le minage en **pool** (Stratum). Disponible pour **Windows** et **Linux (toutes distributions)**.

---

## Sommaire

1. [Informations générales](#1-informations-générales)
2. [Commandes de démarrage rapide](#2-commandes-de-démarrage-rapide)
3. [Toutes les options](#3-toutes-les-options)
4. [Pools supportés](#4-pools-supportés)
5. [Dev Tax](#5-dev-tax-taxe-de-développement)
6. [Configuration d'environnement](#6-configuration-denvironnement)
7. [Guide GPU OpenCL](#7-guide-gpu-opencl)
8. [Linux (toutes distributions)](#8-linux-toutes-distributions)

---

## 1. Informations générales

### Fichier exécutable

**Windows**

| | |
|---|---|
| **Nom** | `HashPulse.exe` |
| **Taille** | ~2.68 MB |
| **Support GPU** | OpenCL inclus dans le binaire |

**Linux** (voir [section 8](#8-linux-toutes-distributions))

| | |
|---|---|
| **Nom** | `HashPulse-linux-x86_64` (CPU) · `HashPulse-linux-x86_64-gpu` (CPU+GPU) |
| **Taille** | ~4.1 MB · ~3.6 MB |
| **Compatibilité** | Toutes distributions (voir [section 8](#8-linux-toutes-distributions)) |

---

## 2. Commandes de démarrage rapide

> Remplacez `xel:VOTRE_ADRESSE` par votre adresse wallet Xelis.

### Solo Mining — Daemon local

```powershell
HashPulse.exe --daemon ws://127.0.0.1:8080 --address xel:VOTRE_ADRESSE
```

Minage solo via un daemon Xelis tournant localement sur le port 8080.

### Solo Mining — Daemon distant (TLS)

```powershell
HashPulse.exe --daemon wss://node.xelis.io/json_rpc --address xel:VOTRE_ADRESSE
```

Minage solo via un daemon Xelis distant avec chiffrement TLS.

### Pool Mining — Vipor (France)

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE
```

Connexion au pool Vipor en France pour le minage en équipe.

### Pool Mining — K1Pool (Europe)

```powershell
HashPulse.exe --pool stratum+tcp://eu.xel.k1pool.com:9351 --address xel:VOTRE_ADRESSE
```

### Pool Mining — HeroMiners (Global)

```powershell
HashPulse.exe --pool stratum+tcp://xelis.herominers.com:1225 --address xel:VOTRE_ADRESSE
```

Serveurs régionaux disponibles : `eu`, `us`, `ru`, `hk`.

### Pool Mining — Kryptex (Europe)

```powershell
HashPulse.exe --pool stratum+tcp://xel-eu.kryptex.network:7019 --address xel:VOTRE_ADRESSE
```

8 serveurs régionaux : Global, EU, US, BR, SG, HK, RU, AE.

### Pool Mining — GrandPool (France)

```powershell
HashPulse.exe --pool stratum+tcp://fr.grandpool.io:2025 --address xel:VOTRE_ADRESSE --user-agent "rigel/1.4.5"
```

⚠️ GrandPool nécessite un user-agent whitelisté. Utilisez `--user-agent "rigel/1.4.5"` pour contourner la restriction. Serveurs : FR, KZ, US.

### GPU Mining — Hybride CPU + GPU (défaut)

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu
```

Active le minage GPU OpenCL **en parallèle du CPU** (tous les cœurs par défaut) : XelisHash V3 favorise le CPU, l'hybride donne le meilleur hashrate total. Utilisez `--no-cpu` pour désactiver le CPU, ou `--threads N` pour limiter le nombre de threads CPU.

### GPU avec intensité personnalisée

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu --gpu-intensity 2048
```

Fixe le nombre de work-items GPU (chaque work-item utilise ~531 KB de VRAM). **2048 est le sweet spot** validé sur GTX 1060 (voir [section 7](#7-guide-gpu-opencl)).

### GPU multiples

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu --gpu-ids 0,1
```

Utilise les GPU 0 et 1 simultanément.

### Worker personnalisé

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --worker rig_01
```

Définit un nom de worker personnalisé pour identifier votre rig sur le pool.

---

## 3. Toutes les options

| Option | Description | Défaut |
|--------|-------------|--------|
| `--daemon <URL>` | URL WebSocket du daemon pour le solo mining | **Requis (solo)** |
| `--pool <URL>` | URL Stratum du pool pour le pool mining | **Requis (pool)** |
| `--address <ADDR>` | Adresse wallet Xelis pour recevoir les récompenses | **Obligatoire** |
| `--worker <NAME>` | Nom du worker visible sur le pool | `worker1` |
| `--threads <N>` | Nombre de threads CPU pour le minage | auto (tous les cœurs) |
| `--gpu` | Active le minage GPU OpenCL (le CPU reste actif par défaut) | désactivé |
| `--no-cpu` | Désactive complètement le minage CPU | désactivé |
| `--gpu-ids <IDS>` | IDs des GPU à utiliser (séparés par virgule) | `0` |
| `--gpu-intensity <N>` | Work-items par GPU (~531 KB VRAM chacun). Plus = plus de hashrate | auto (2048) |
| `--dev-fee <PERCENT>` | Pourcentage de la dev tax (minage pour le développeur) | `1.0` |
| `--user-agent <NAME>` | User-agent du mineur (certains pools exigent un nom whitelisté, ex : `rigel/1.4.5` pour GrandPool) | `HashPulse/1.0.0` |
| `-h, --help` | Affiche l'aide en ligne de commande | — |

---

## 4. Pools supportés

| Pool | Région | URL Stratum |
|------|--------|-------------|
| Vipor | France | `stratum+tcp://fr.vipor.net:5077` |
| K1Pool | Europe | `stratum+tcp://eu.xel.k1pool.com:9351` |
| HeroMiners | Global | `stratum+tcp://xelis.herominers.com:1225` |
| HeroMiners | Europe | `stratum+tcp://eu.xelis.herominers.com:1225` |
| HeroMiners | Amérique du Nord | `stratum+tcp://us.xelis.herominers.com:1225` |
| HeroMiners | Russie | `stratum+tcp://ru.xelis.herominers.com:1225` |
| HeroMiners | Asie (HK) | `stratum+tcp://hk.xelis.herominers.com:1225` |
| Kryptex | Global | `stratum+tcp://xel.kryptex.network:7019` |
| Kryptex | Europe | `stratum+tcp://xel-eu.kryptex.network:7019` |
| Kryptex | Amérique du Nord | `stratum+tcp://xel-us.kryptex.network:7019` |
| Kryptex | Amérique du Sud | `stratum+tcp://xel-br.kryptex.network:7019` |
| Kryptex | Singapour | `stratum+tcp://xel-sg.kryptex.network:7019` |
| Kryptex | Chine (HK) | `stratum+tcp://xel-hk.kryptex.network:7019` |
| Kryptex | Russie | `stratum+tcp://xel-ru.kryptex.network:7019` |
| Kryptex | Moyen-Orient | `stratum+tcp://xel-ae.kryptex.network:7019` |
| BaikalMine | Europe | `stratum+tcp://xel-eu.baikalmine.com:9030` |
| Suprnova | Global | `stratum+tcp://xel.suprnova.cc:3333` |
| 1Miner | Europe | `stratum+tcp://eu1.1miner.net:4073` |
| GrandPool | France (Paris) | `stratum+tcp://fr.grandpool.io:2025` |
| GrandPool | Kazakhstan (Almaty) | `stratum+tcp://kz.grandpool.io:2025` |
| GrandPool | USA (New York) | `stratum+tcp://us.grandpool.io:2025` |

---

## 5. Dev Tax (taxe de développement)

Une dev tax de **1 %** est appliquée par défaut. Elle est configurable entre **0.5 %** et **5 %**.

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --dev-fee 2.0
```

Définit la dev tax à 2 %. Le minimum est 0.5 % et le maximum 5 %.

---

## 6. Configuration d'environnement

### Niveau de log (PowerShell)

```powershell
$env:RUST_LOG="info"; HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu
```

Niveaux disponibles : `error`, `warn`, `info` (recommandé), `debug`, `trace`.

### Voir l'aide complète

```powershell
HashPulse.exe --help
```

Affiche toutes les options et exemples directement dans le terminal.

---

## 7. Guide GPU OpenCL

### Prérequis GPU

- Drivers GPU à jour (NVIDIA, AMD, ou Intel)
- Runtime OpenCL installé sur le système
- VRAM suffisante : ~531 KB par work-item (intensity)
- L'exécutable `HashPulse.exe` inclut déjà le support GPU

### Calcul de la VRAM nécessaire

| Intensité | VRAM |
|-----------|------|
| `--gpu-intensity 256` | ~133 MB |
| `--gpu-intensity 512` | ~266 MB |
| `--gpu-intensity 1024` | ~531 MB |
| `--gpu-intensity 2048` | ~1.06 GB |
| `--gpu-intensity 2560` | ~1.33 GB |
| Par défaut (auto) | calculé selon `MaxMemAllocSize` du GPU (plafonné à 2048) |

### Intensité optimale (retour terrain — GTX 1060 6 Go)

Un balayage complet de `--gpu-intensity` a montré que le débit réel monte avec l'occupation GPU jusqu'à ~2048 work-items, puis plafonne :

| Intensité | Débit réel | Gain vs 1024 |
|-----------|------------|--------------|
| 1024 | ~410 H/s | référence |
| 1536 | ~530 H/s | +29 % |
| **2048** (défaut) | **~614 H/s** | **+50 %** |
| 2560 | ~625 H/s | +2 % (plateau) |

**2048 est donc le défaut** : meilleur compromis débit / stabilité / marge VRAM. Sur les cartes plus petites, la valeur auto est automatiquement bornée par la VRAM disponible.

### Tester la détection GPU

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu
```

Au démarrage, HashPulse détecte et liste tous les GPU OpenCL disponibles avec leur VRAM.

---

## 8. Linux (toutes distributions)

| Fichier | Usage | Compatibilité |
|---|---|---|
| `HashPulse-linux-x86_64` | CPU uniquement | **100 % statique (musl), zéro dépendance** : Ubuntu, Debian, Arch, Fedora, CentOS/RHEL, Alpine, NixOS, vieux serveurs… testé et validé |
| `HashPulse-linux-x86_64-gpu` | CPU + GPU (OpenCL) | glibc ≥ 2.17 (CentOS 7+, Ubuntu 14.04+, Debian 8+, Arch, Fedora). Nécessite le runtime OpenCL du système |

Les deux binaires ont été **validés en conditions réelles** (minage sur pool, shares acceptées) sous Ubuntu 24.04 et Alpine 3.24, avec détection/pilotage OpenCL complet vérifié.

### Démarrage rapide

```bash
chmod +x HashPulse-linux-x86_64*
# CPU (recommandé — XelisHash V3 favorise le CPU) :
./HashPulse-linux-x86_64 --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --worker rig_01
# CPU + GPU :
./HashPulse-linux-x86_64-gpu --gpu --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --worker rig_01
```

Toutes les options de la [section 3](#3-toutes-les-options) sont identiques sous Linux.

### Prérequis GPU sous Linux

Le binaire `-gpu` utilise le loader OpenCL standard de la distribution (`libOpenCL.so.1`), normalement déjà présent avec le pilote :

- **NVIDIA** : pilote propriétaire (installe `libnvidia-opencl.so` + l'ICD)
- **AMD** : `rocm-opencl-runtime` (ou Mesa/RADV `mesa-opencl` pour GPU récents)
- **Intel** : `intel-opencl-icd` (ou `intel-compute-runtime`)

Si aucune plateforme OpenCL n'est trouvée, HashPulse **bascule automatiquement en CPU-only** avec un message d'avertissement (aucun crash).

> Sans interface graphique (serveur headless), préférez le binaire CPU.
> GPU non disponible sur Alpine (musl) et NixOS (FHS) — le binaire CPU y fonctionne parfaitement.

### Niveau de log (bash)

```bash
RUST_LOG=info ./HashPulse-linux-x86_64-gpu --gpu --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE
```

### Exécution en arrière-plan (serveur)

```bash
nohup ./HashPulse-linux-x86_64 --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE > hashpulse.log 2>&1 &
```

---

<div align="center">

**HashPulse v1.0.0** — Mineur Xelis avec support CPU & GPU OpenCL

XelisHash V3 · Dev Tax configurable · Solo & Pool Mining · Windows & Linux

</div>
