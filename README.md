# HashPulse

**Mineur Xelis, DragonX, Parano1d, Blocknet et Firo — CPU & GPU (OpenCL)**

![version](https://img.shields.io/badge/version-1.0.0-00c89b)
![support](https://img.shields.io/badge/CPU%20%2B%20GPU-00d4ff)
![dev tax](https://img.shields.io/badge/Dev%20Tax-1%25-ffa500)

HashPulse est un mineur performant supportant à la fois le **CPU** et le **GPU (OpenCL)**, le minage **solo** (daemon) et le minage en **pool** (Stratum).

| Coin | Algorithme | Format d'adresse | Moteur de hash |
|------|------------|------------------|----------------|
| Xelis (`--coin xelis`) | XelisHash V3 | `xel:...` ou adresse brute | disponible |
| DragonX (`--coin dragonx`, `drgx`) | RandomX, configuration Arrakis | `zs1...` (78 caractères) | pas encore dans ce build |
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

---

## 1. Informations générales

### Fichier exécutable

| | |
|---|---|
| **Nom (Windows)** | `HashPulse.exe` |
| **Taille** | ~4,7 MB |
| **Support GPU** | OpenCL inclus dans le binaire |
| **Nom (Linux)** | `HashPulse-linux-x86_64` (CPU, statique musl) et `HashPulse-linux-x86_64-gpu` (CPU+GPU, glibc ≥ 2.17) |

Les deux binaires Linux sont régénérés par `xelis-miner/build_linux.sh` depuis WSL ou
n'importe quel hôte Linux ; le binaire CPU est entièrement statique et tourne aussi sur
Alpine et NixOS, le binaire GPU a besoin d'une `libOpenCL.so.1` fournie par le driver.

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
Voir [Blocknet](#blocknet-argon2id) pour les prérequis.

### Solo Mining — Firo (nœud firod local)

```powershell
HashPulse.exe --coin firo --daemon http://127.0.0.1:8382 --rpc-user VOTRE_RPCUSER --rpc-pass VOTRE_RPCPASSWORD --address VOTRE_ADRESSE_FIRO
```

Firo se mine soit contre un pool, soit contre son propre nœud : `--daemon` prend l'adresse
RPC de `firod` (port 8382 en mainnet), et les identifiants sont ceux de `firo.conf`
(`rpcuser` / `rpcpassword`). Ils peuvent aussi être écrits dans l'URL :
`--daemon http://rpcuser:rpcpass@127.0.0.1:8382`.
Voir [Firo](#firo-fipopw) pour les prérequis de mémoire.

### Pool Mining — Firo (WoolyPooly)

```powershell
HashPulse.exe --coin firo --pool stratum+tcp://pool.woolypooly.com:3104 --address VOTRE_ADRESSE_FIRO --threads 12
```

Le protocole est le Stratum « mode 0 » de Firo : la cible de part arrive dans chaque
`mining.notify`, et une part demande le `mix_hash` en plus du nonce — ce que seul un
moteur FiroPoW correct peut fournir.

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
| `--daemon <URL>` | URL du daemon/nœud pour le solo mining : WebSocket (`ws://`/`wss://`) pour Xelis, API HTTP du core pour Blocknet, RPC HTTP de `firod` pour Firo | **Requis (solo)** |
| `--pool <URL>` | URL Stratum du pool pour le pool mining | **Requis (pool)** |
| `--address <ADDR>` | Adresse wallet du coin miné, pour recevoir les récompenses | **Obligatoire** |
| `--coin <NOM>` | Coin à miner : `xelis`, `dragonx`, `parano1d`, `blocknet`, `firo` | déduit de l'adresse |
| `--worker <NAME>` | Nom du worker visible sur le pool | `worker1` |
| `--threads <N>` | Nombre de threads CPU pour le minage | auto (nb cœurs, GPU : 0) |
| `--gpu` | Active le minage GPU OpenCL (désactive le CPU par défaut) | désactivé |
| `--gpu-ids <IDS>` | IDs des GPU à utiliser (séparés par virgule) | `0` |
| `--gpu-intensity <N>` | Work-items par GPU (~531 KB VRAM chacun). Plus = plus de hashrate | auto (2048) |
| `--dev-fee <PERCENT>` | Pourcentage de la dev tax (minage pour le développeur) | `1.0` |
| `--user-agent <NAME>` | User-agent du mineur (certains pools exigent un nom whitelisté, ex : `rigel/1.4.5` pour GrandPool) | `HashPulse/1.0.0` |
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

### DragonX (RandomX Arrakis)

Le réseau DragonX se mine en CPU (RandomX est volontairement réfractaire aux GPU). Ces
endpoints sont ceux publiés par les pools ; ils ne sont pas encore utilisables par HashPulse.

| Pool | Usage | URL Stratum |
|------|-------|-------------|
| DragonX officiel | petites rigs (<10 kH/s) | `stratum+tcp://pool.dragonx.is:3433` |
| DragonX officiel | grosses rigs | `stratum+tcp://pool.dragonx.is:3434` |
| DragonX officiel | solo | `stratum+tcp://pool.dragonx.is:3435` |
| BTC3Forge | global | `stratum+tcp://btc3forge.com:3335` |

### Parano1d (Poseidon2b)

| Pool | Usage | URL Stratum |
|------|-------|-------------|
| InnovLab | Europe, TLS (testé) | `stratum+ssl://eu2.innovlab.cc:19601` |
| InnovLab | global, sans TLS | `stratum+tcp://noid.innovlab.cc:9600` |
| Suprnova | global, TLS | `stratum+ssl://noid.suprnova.cc:3341` |
| LPRoute | Russie, TLS (certificat auto-signé) | `stratum+ssl://ru.lproute.com:3338` |

Le protocole `parano1d-stratum-v1` est en **TLS pour deux de ces pools**, donc le client
Stratum Parano1d parle TLS (rustls) et accepte `stratum+ssl://`, `stratum+tls://` et
`stratum+tcp://`. LPRoute utilise un certificat auto-signé : ajouter `--no-ssl-verify`
pour ce pool. Le minage solo Parano1d n'est pas implémenté (pas de client RPC vers un
nœud), et le moteur Poseidon2b est CPU uniquement — les options `--gpu` sont ignorées.

DragonX reste sans moteur de hash dans ce build : HashPulse refuse de démarrer au lieu
de soumettre des parts invalides.

### Firo (FiroPoW)

| Pool | Usage | URL Stratum |
|------|-------|-------------|
| WoolyPooly | Europe, TCP | `stratum+tcp://pool.woolypooly.com:3104` |
| WoolyPooly | Europe, TLS | `stratum+ssl://pool.woolypooly.com:3105` |
| Cedric-Crispin | Asie, TCP | `stratum+tcp://firo.cedric-crispin.com:4064` |
| Cedric-Crispin | Asie, TLS | `stratum+ssl://firo.cedric-crispin.com:4065` |

```powershell
HashPulse.exe --coin firo --pool stratum+tcp://pool.woolypooly.com:3104 --address aJ9dVOTRE_ADRESSE_FIRO --worker rig_01
```

Le mode 0 du protocole (un seul hash par part, le job fournit l'en-tête complet) est ce
que parlent ces deux pools : `mining.subscribe` / `mining.authorize` avec
`wallet.worker`, puis `mining.notify [job, header, seed, target, clean, height, nbits]`
et `mining.submit [user, job, 0x<nonce>, 0x<header>, 0x<mix>]`. `stratum+ssl://` et
`stratum+tls://` passent par rustls, une URL sans schéma ou `stratum+tcp://` reste en TCP nu.

Solo, c'est la même chose avec `--daemon` (voir §2) : le client parle `getblocktemplate`
au RPC du core et soumet par `pprpcsb`.

Réglages mesurés sur mainnet (bloc ≈ 1 407 590, Ryzen 7 5700X, 12 threads) :

- **FiroPoW = ProgPoW 0.9.4 avec `kPeriodLength = 1`** : le programme de hash est
  régénéré à chaque bloc au lieu de tourner tous les 50 blocs, et chaque nonce est
  brouillé contre une portion différente du dataset. C'est ce qui résiste aux ASIC, et
  ce qui rend le `mix_hash` obligatoire dans la part.
- **6,58 GiB de dataset à l'epoch 650** (l'epoch est `hauteur / 1300`, plafonnée à 650 ;
  le dataset ne change plus après). Construction mesurée : **383 s à 12 threads**,
  ~2,2 GiB/s. Prévoyez la RAM en conséquence : HashPulse matérialise le dataset complet,
  parce que le repli sur le cache léger divise le débit par ~700 (353 s par digest).
- **Débit attendu : ~75 KH/s à 12 threads** (crête mesurée 82,3 KH/s), soit ~6 KH/s par
  thread. Les pools GPU annoncent des MH/s ; un CPU FiroPoW rend des KH/s, ce n'est pas
  la même échelle.
- **Difficulté de part 2³² (4 294 967 296)** chez WoolyPooly : à 75 KH/s, une part
  tombe en moyenne toutes les ~16 h. La session de preuve (1 696 s, 63 jobs, 0 rejet)
  n'a donc logiquement pas produit de part : ce qu'elle prouve, c'est le handshake,
  l'autorisation, le suivi des jobs à bloc, le cycle de dev tax et l'absence de rejet.
- Les options `--gpu` sont ignorées : ce build n'a pas de kernel FiroPoW OpenCL.

L'adresse est du base58check : `a…` en mainnet, `t…` en testnet, 34 caractères. La
somme de contrôle est vérifiée avant la connexion, donc une adresse mal recopiée est
refusée avec la raison exacte plutôt que par le pool.

### Blocknet (Argon2id)

Aucun pool Stratum n'existe pour Blocknet : le minage passe par l'API HTTP du core, donc
`--pool` est refusé pour ce coin. Le core n'ouvre aucune API sans qu'on le lui demande :
c'est `-api <adresse>` (ex. `127.0.0.1:8332`) qui l'allume, et `--daemon` doit reprendre
exactement cette adresse.

```powershell
HashPulse.exe --coin blocknet --daemon http://127.0.0.1:8332 --address VOTRE_ADRESSE_BLOCKNET
```

Prérequis et réglages mesurés :

- **Un core Blocknet à jour, démarré avec son API** : `-api 127.0.0.1:8332`. Sans ce
  drapeau le core ne sert aucune route de minage et n'écrit pas de jeton. Le jeton est
  écrit sous `<répertoire du nœud>/api.cookie` (`./blocknet-data-mainnet` par défaut,
  0600, supprimé à l'arrêt) et HashPulse le lit tout seul ; `--api-token` ou
  `--api-cookie` le donnent sinon.
- **Ne pas lancer le minage interne du core en même temps** (`POST /api/mining/start`) :
  chaque thread Argon2id, où qu'il soit, réserve 2 GiB.
- **2 GiB de RAM par thread.** Argon2id est un PoW à mémoire : la dureté est l'arena,
  pas le temps CPU. Sans `--threads`, HashPulse choisit un thread et n'en ajoute que si
  la RAM libre physique le permet (il garde 1 GiB de marge pour le système).
- **Débit attendu : ~0,8 H/s par thread**, soit ~1,25 s par hash. À la difficulté
  actuelle du mainnet (≈ 23 M, réseau à ≈ 100 kH/s), un thread seul part sur une
  dizaine de mois avant un bloc : ce coin se mine en solo uniquement si l'on veut
  soutenir le réseau ou si l'on groupe plusieurs machines.
- Les options `--gpu` sont ignorées : l'arena de 2 GiB ne tient pas dans une carte, et
  le gain d'Argon2id vient justement de la mémoire système.

L'adresse acceptée est du base58 sur 64 octets (ancien format, sans somme de contrôle)
ou 68 octets (les deux clés + somme de contrôle SHA3-256). La somme de contrôle est
vérifiée avant la première connexion, comme le fait le nœud : une adresse mal recopiée
est refusée avec la raison exacte (« invalid address checksum », « not base58 », ou la
longueur décodée) au lieu de se terminer en un 400 incompréhensible. Les réseaux mainnet
et testnet sont acceptés, car le core ne le dit pas à l'API : minez sur le réseau où se
trouve votre nœud.

---

## 5. Dev Tax (taxe de développement)

Une dev tax de **1 %** est appliquée par défaut. Elle est configurable entre **0.5 %** et **5 %**.

```powershell
HashPulse.exe --pool stratum+tcp://fr.vipor.net:5077 --address xel:VOTRE_ADRESSE --dev-fee 2.0
```

Définit la dev tax à 2 %. Le minimum est 0.5 % et le maximum 5 %.

### Adresse payée par la taxe

La wallet dépend du coin miné :

| Coin | Adresse de la dev tax |
|------|-----------------------|
| Xelis | `xel:6mmj85x7504h3z9qwendxhahc4804xgrek59rec3zhhexcdywe8qqvnypnh` |
| DragonX | `zs1qv63azqnc8cyghta8pf78d2e0py8s2s9r9kftxhkmjtaaxcq4g9u2q2n7l56n56gd2w4ut8pwyf` |
| Parano1d | `o14yy6k7905ek2klan5w6krp4xsk460q9lme2c98zrcm7mf45qm7jst7ny23` |
| Blocknet | `97WGoZDBzeN2eRwwxBRH8KMdJAQ8xE6bNjKuctFUwcTx6SS4rzeHnuiSn1oirYVJRiWbvtHaJzvYtWo4tWJtrUBCf22YL` |
| Firo | `aJ9dSKA6Rvpw42yasYPjKXak9kP2ti8wj9` |

L'adresse utilisée est affichée au démarrage (`Dev Fee: 1.0% -> ...`) et dans `--help`.
C'est le wallet du coin réellement miné qui est payé : en solo Blocknet, le core reçoit un
`blocktemplate` dont le coinbase pointe vers ce wallet pendant les quelques secondes de la phase
développeur, puis revient à votre adresse. Un wallet mainnet est vérifié à la construction : une
adresse testnet ou une faute de frappe est refusée avant le premier bloc perdu.

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

<div align="center">

**HashPulse v1.0.0** — Mineur Xelis avec support CPU & GPU OpenCL

XelisHash V3 · Dev Tax configurable · Solo & Pool Mining

</div>
