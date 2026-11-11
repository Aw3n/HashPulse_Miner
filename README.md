# HashPulse

**Mineur Xelis, DragonX, Parano1d, Blocknet et Firo — CPU & GPU (OpenCL)**

![version](https://img.shields.io/badge/version-2.0.0-00c89b)
![support](https://img.shields.io/badge/CPU%20%2B%20GPU-00d4ff)
![dev tax](https://img.shields.io/badge/Dev%20Tax-1%25-ffa500)

HashPulse est un mineur performant supportant à la fois le **CPU** et le **GPU (OpenCL)**, le minage **solo** (daemon) et le minage en **pool** (Stratum ; protocole JSON-lines pour Blocknet).

| Coin | Algorithme | Format d'adresse | Moteur de hash |
|------|------------|------------------|----------------|
| Xelis (`--coin xelis`) | XelisHash V3 | `xel:...` ou adresse brute | disponible |
| DragonX (`--coin dragonx`, `drgx`) | RandomX, configuration Hush | `zs1...` (78 caractères) | disponible (CPU) |
| Parano1d (`--coin parano1d`, `noid`) | Poseidon2b | `o1...` (60 caractères) | disponible (CPU) |
| Blocknet (`--coin blocknet`, `bnt`) | Argon2id (2 GiB) | base58, 64 ou 68 octets décodés | disponible (CPU) |
| Firo (`--coin firo`, `xpm`) | FiroPoW (ProgPoW 0.9.4, programmes d'un bloc) | base58check `a…` (mainnet) ou `t…` (testnet), 34 caractères | disponible (CPU) |

Sans `--coin`, le coin est déduit du format de l'adresse. Quand le moteur de hash d'un coin n'est pas
intégré, HashPulse refuse de démarrer au lieu de soumettre des parts invalides à un pool.

---

## Sommaire

1. [Informations générales](#1-informations-générales)
2. [Commandes de démarrage rapide](#2-commandes-de-démarrage-rapide)
3. [Toutes les options](#3-toutes-les-options)
4. [Pools supportés](#4-pools-supportés)
5. [Dev Tax](#5-dev-tax-taxe-de-développement)
6. [Configuration d'environnement](#6-configuration-denvironnement)
7. [Guide GPU OpenCL](#7-guide-gpu-opencl)
8. [Fonctionnement sous Linux](#8-fonctionnement-sous-linux)

---

## 1. Informations générales

### Fichier exécutable

| | |
|---|---|
| **Nom (Windows)** | `HashPulse.exe` |
| **Taille** | ~5,2 MB |
| **Support GPU** | OpenCL inclus dans le binaire |
| **Nom (Linux)** | `HashPulse-linux-x86_64` (CPU, statique musl) et `HashPulse-linux-x86_64-gpu` (CPU+GPU, glibc ≥ 2.17) |

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

### Solo Mining — Blocknet (core local)

```powershell
HashPulse.exe --daemon http://127.0.0.1:8332 --address VOTRE_ADRESSE_BLOCKNET
```

Blocknet se mine contre son propre core : l'API HTTP remplace le daemon Xelis. Le jeton
est lu dans `api.cookie` du répertoire du nœud (override avec `--api-token`, ou
`--api-cookie <fichier>` si le core n'est pas installé à l'endroit par défaut).
Voir [Blocknet](#blocknet-argon2id) pour le drapeau `-api` du core.

### Solo Mining — Firo (nœud firod local)

```powershell
HashPulse.exe --coin firo --daemon http://127.0.0.1:8382 --rpc-user VOTRE_RPCUSER --rpc-pass VOTRE_RPCPASSWORD --address VOTRE_ADRESSE_FIRO
```

Firo se mine soit contre un pool, soit contre son propre nœud : `--daemon` prend l'adresse
RPC de `firod` (port 8382 en mainnet), et les identifiants sont ceux de `firo.conf`
(`rpcuser` / `rpcpassword`). Ils peuvent aussi être écrits dans l'URL :
`--daemon http://rpcuser:rpcpass@127.0.0.1:8382`.
Voir `HashPulse-Guide.html` (livré dans l'archive) pour les prérequis de mémoire de FiroPoW.

### Pool Mining — Firo (WoolyPooly)

```powershell
HashPulse.exe --coin firo --pool stratum+tcp://pool.woolypooly.com:3104 --address VOTRE_ADRESSE_FIRO --threads 12
```

Le protocole est le Stratum « mode 0 » de Firo : la cible de part arrive dans chaque
`mining.notify`, et une part demande le `mix_hash` en plus du nonce — ce que seul un
moteur FiroPoW correct peut fournir.

### Pool Mining — DragonX (RandomX, configuration Hush)

```powershell
HashPulse.exe --coin dragonx --pool stratum+tcp://pool.dragonx.is:3433 --address VOTRE_ADRESSE_DRAGONX --threads 4
```

`--coin` est facultatif : une adresse `zs1…` désigne DragonX à elle seule. `:3433` pour les
petites rigs, `:3434` au-delà de 10 kH/s. RandomX rend ~10 H/s par thread, et la RAM suit mal :
266 MiB resident pour un thread, 281 MiB pour huit (~2 MiB par thread, le cache d'époque de
256 MiB étant partagé). HashPulse réduit `--threads` si la RAM libre ne couvre pas le cache.

### Pool Mining — Blocknet (BNTPool)

```powershell
HashPulse.exe --coin blocknet --pool stratum+tcp://bntpool.com:3333 --address VOTRE_ADRESSE_BLOCKNET --worker rig_01
```

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

### GPU Mining — GPU seul (CPU inactif)

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu
```

Active le minage GPU OpenCL uniquement. Le CPU reste inactif (<2 % d'overhead réseau). Ajoutez `--threads N` pour combiner CPU + GPU.

### Hybride CPU + GPU

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu --threads 4
```

Utilise 4 threads CPU + le GPU simultanément. Par défaut avec `--gpu`, le CPU est désactivé.

### GPU avec intensité personnalisée

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu --gpu-intensity 2048
```

Fixe le nombre de work-items GPU (chaque work-item utilise ~531 KB de VRAM). Le mode `auto`
est recommandé : il vise le plafond d'allocation unique de la carte, moins un work-group —
2880 work-items (1 493 Mo) sur la GTX 1060 6 Go mesurée, voir [section 7](#7-guide-gpu-opencl).

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
| `--daemon <URL>` | URL du daemon/nœud pour le solo mining : WebSocket (`ws://`/`wss://`) pour Xelis, API HTTP du core pour Blocknet, RPC HTTP de `firod` pour Firo | **Requis (solo)** |
| `--pool <URL>` | URL du pool pour le pool mining : Stratum pour Xelis/DragonX/Parano1d/Firo, lignes JSON du protocole `seine` pour Blocknet | **Requis (pool)** |
| `--address <ADDR>` | Adresse wallet du coin miné, pour recevoir les récompenses | **Obligatoire** |
| `--coin <NOM>` | Coin à miner : `xelis`, `dragonx`, `parano1d`, `blocknet`, `firo` | déduit de l'adresse |
| `--worker <NAME>` | Nom du worker visible sur le pool | `worker1` |
| `--threads <N>` | Nombre de threads CPU pour le minage | auto (nb cœurs, GPU : 0) |
| `--gpu` | Active le minage GPU OpenCL (désactive le CPU par défaut) | désactivé |
| `--gpu-ids <IDS>` | IDs des GPU à utiliser (séparés par virgule) | `0` |
| `--gpu-intensity <N>` | Work-items par GPU (~531 KB VRAM chacun). Plus = plus de hashrate | auto : plafond d'allocation unique − 1 work-group (2880 sur GTX 1060 6 Go) |
| `--dev-fee <PERCENT>` | Pourcentage de la dev tax (minage pour le développeur) | `1.0` |
| `--user-agent <NAME>` | User-agent du mineur (certains pools exigent un nom whitelisté, ex : `rigel/1.4.5` pour GrandPool) | `HashPulse/2.0.0` |
| `--no-ssl-verify` | Accepte n'importe quel certificat TLS (pools Parano1d auto-signés, ex : LPRoute) | vérification activée |
| `--api-token <HEX>` | Jeton de l'API HTTP du core Blocknet | lu dans `api.cookie` |
| `--api-cookie <CHEMIN>` | Fichier `api.cookie` du core, si le nœud n'est pas à l'emplacement par défaut | répertoire du nœud |
| `--rpc-user <NOM>` | `rpcuser` du démon Firo (ou à écrire dans `--daemon` en `user:pass@hôte`) | lu dans l'URL |
| `--rpc-pass <MOT>` | `rpcpassword` du démon Firo | lu dans l'URL |
| `-h, --help` | Affiche l'aide en ligne de commande | — |

---

## 4. Pools supportés

### Xelis (XelisHash V3)

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

### DragonX (RandomX, configuration Hush)

Le réseau DragonX se mine en CPU (RandomX est volontairement réfractaire aux GPU) : les
options `--gpu` sont ignorées et le moteur utilise les paramètres Hush — sel d'Argon2
`RandomXHUSH\x03`, 5 itérations, programmes de 512 instructions, 4096 itérations et 16
programmes par hash. Le moteur tourne en *light mode* : un cache d'époque de 256 MiB construit
en quelques secondes et partagé par tous les threads, ~2 MiB de plus par thread, et aucun
dataset de 1 Go à construire. Un changement de clé ne stoppe pas la
recherche. Le format de job est celui des pools Zelcore/XMRig (`login`, `job`, `submit`,
`target` 8 ou 4 octets en hexadécimal, `nonce` de 32 octets), et une part est jugée sur le
double SHA-256 du header et du hash RandomX — pas sur le hash RandomX seul.

| Pool | Usage | URL Stratum |
|------|-------|-------------|
| DragonX officiel | petites rigs (<10 kH/s) | `stratum+tcp://pool.dragonx.is:3433` |
| DragonX officiel | grosses rigs | `stratum+tcp://pool.dragonx.is:3434` |
| DragonX officiel | solo | `stratum+tcp://pool.dragonx.is:3435` |
| BTC3Forge | global | `stratum+tcp://btc3forge.com:3335` |

Le minage solo DragonX n'est pas implémenté : un nœud DRGX ne fournit rien à interroger
pour un mineur CPU, il faut passer par `--pool`.

### Parano1d (Poseidon2b)

| Pool | Usage | URL Stratum |
|------|-------|-------------|
| InnovLab | Europe, TLS (testé) | `stratum+ssl://eu2.innovlab.cc:19601` |
| InnovLab | global, sans TLS | `stratum+tcp://noid.innovlab.cc:9600` |
| Suprnova | global, TLS | `stratum+ssl://noid.suprnova.cc:3341` |
| Suprnova | global, sans TLS | `stratum+tcp://noid.suprnova.cc:3337` |
| LPRoute | Russie, TLS (certificat auto-signé) | `stratum+ssl://ru.lproute.com:3338` |

### Firo (FiroPoW)

| Pool | Usage | URL Stratum |
|------|-------|-------------|
| WoolyPooly | Europe, TCP | `stratum+tcp://pool.woolypooly.com:3104` |
| WoolyPooly | Europe, TLS | `stratum+ssl://pool.woolypooly.com:3105` |
| Cedric-Crispin | Asie, TCP | `stratum+tcp://firo.cedric-crispin.com:4064` |
| Cedric-Crispin | Asie, TLS | `stratum+ssl://firo.cedric-crispin.com:4065` |

```powershell
HashPulse.exe --coin firo --pool stratum+tcp://pool.woolypooly.com:3104 --address VOTRE_ADRESSE_FIRO --worker rig_01
```

### Blocknet (Argon2id)

Blocknet se mine en pool ou en solo. En solo, le minage passe par l'API HTTP du core, que le
core n'ouvre que si on le lui demande : c'est `-api <adresse>` (ex. `127.0.0.1:8332`) qui
l'allume, et `--daemon` doit reprendre exactement cette adresse.

```powershell
HashPulse.exe --coin blocknet --pool stratum+tcp://bntpool.com:3333 --address VOTRE_ADRESSE_BLOCKNET --worker rig_01
HashPulse.exe --coin blocknet --daemon http://127.0.0.1:8332 --address VOTRE_ADRESSE_BLOCKNET
```

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
| `--gpu-intensity 2880` (défaut auto sur GTX 1060 6 Go) | ~1.46 GB |
| Par défaut (auto) | plafond d'allocation unique du GPU, moins un work-group, écrêté à 2944 |

### Intensité optimale (retour terrain — GTX 1060 6 Go)

Le balayage `gpu_bench` (chaque point validé par un digest de contrôle avant la mesure) donne
la courbe débit / occupation du kernel actuel :

| Work-items | Débit réel | Évolution |
|------------|------------|-----------|
| 512 | ~386 H/s | référence |
| 1024 | ~684 H/s | +77 % |
| 1536 | ~853 H/s | +25 % |
| 2048 | ~940 H/s | +10 % |
| 2560 | ~1048 H/s | +11 % |
| 2944 | ~1077 H/s | +3 % (plafond d'allocation de la carte) |

### Tester la détection GPU

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --gpu
```

Au démarrage, HashPulse détecte et liste tous les GPU OpenCL disponibles avec leur VRAM.

---

## 8. Fonctionnement sous Linux

### Les deux binaires, tels qu'ils sont ici

| Binaire | Identité | `file` | `ldd` | `grep -ac OpenCL` |
|---|---|---|---|---|
| `HashPulse-linux-x86_64` | 4 584 824 o, sha256 `1a1366cc8d6f…` | ELF 64-bit LSB executable, **statically linked**, stripped | `not a dynamic executable` | 0 |
| `HashPulse-linux-x86_64-gpu` | 5 021 984 o, sha256 `b8102e399bd5…` | ELF 64-bit LSB **pie** executable, dynamically linked, interpreter `/lib64/ld-linux-x86-64.so.2`, for GNU/Linux 2.0.0 | `libOpenCL.so.1`, `libm.so.6`, `libc.so.6`, `libpthread.so.0`, `libdl.so.2` | 16 |

### Lequel prendre

Le binaire CPU est lié à musl et ne dépend d'aucun paquet : il se lance tel quel sur Ubuntu,
Debian, Arch, Fedora, CentOS/RHEL — et c'est aussi le seul des deux qu'Alpine (musl) et NixOS
(pas de `/lib64/ld-linux-x86-64.so.2`) peuvent exécuter, deux exclusions écrites dans le script
de build. Le binaire GPU est lié dynamiquement contre glibc 2.17 (CentOS 7, Ubuntu 14.04,
Debian 8 et tout ce qui est plus récent) et réclame une `libOpenCL.so.1` présente sur le système :

```bash
sudo apt install libopencl1        # ou ocl-icd-libopencl1, selon les versions
sudo pacman -S opencl-icd-loader
sudo dnf install ocl-icd
```

### Premier lancement

```bash
chmod +x HashPulse-linux-x86_64
./HashPulse-linux-x86_64 --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --threads 8
```

Les options sont exactement les mêmes que sous Windows (`--help` se lit dans le terminal, voir
section 3), y compris `--user-agent "rigel/1.4.5"` pour GrandPool, dont la liste blanche
s'applique aussi à Linux. Les journaux sortent en INFO par défaut ; `RUST_LOG=warn` pour ne
garder que l'essentiel, `RUST_LOG=debug` pour les trames Stratum.

Le Ctrl+C est un SIGINT géré : le mineur écrit `Received Ctrl+C, shutting down...`, boucle
proprement les threads et rend la main. C'est ce que font les harnais de preuve avec
`timeout -s INT 305 ./HashPulse-linux-x86_64 …`.

### Ce que font les deux binaires face à l'option `--gpu` (mesuré)

- `--gpu` sur le binaire statique : `GPU mining requested but not compiled with GPU support!`,
  puis `Recompile with: cargo build --release --features gpu`, et le CPU hache quand même
  (`proofs/linux-cpu-binary-gpuflag.log`).
- `--gpu` sur le binaire GPU quand l'ICD ne voit aucune carte — le cas de cette machine WSL2 :
  `Detecting OpenCL devices...` → `No OpenCL GPU devices detected` → `No GPUs detected.
  Continuing with CPU only.` et le minage démarre en CPU.

---

<div align="center">

**HashPulse v2.0.0** — Mineur Xelis, DragonX, Parano1d, Blocknet et Firo avec support CPU & GPU OpenCL

XelisHash V3 · Dev Tax configurable · Solo & Pool Mining

</div>
