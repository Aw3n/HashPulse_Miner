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

---

## 1. Informations générales

### Fichier exécutable

| | |
|---|---|
| **Nom (Windows)** | `HashPulse.exe` |
| **Taille** | ~5,2 MB |
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

Le protocole n'est pas Stratum : ce sont les lignes JSON du mineur officiel `seine`
(`login` avec capacités, `job` avec `header_base`, `target` et une plage de nonces, `submit`
avec le `claimed_hash`, `refill` en fin de plage). Mesuré sur 700 s avec un seul thread :
**4 parts acceptées, 0 rejetée, première part 23 s après le login**, 11 jobs suivis sur les
blocs 72664→72666 et difficulté de part ajustée par le pool de 60 à 90, 112 puis 85 sans
reconnexion du client. Le cycle de dev tax passe aussi par le pool : resommission du `login`
côté frais, puis retour côté utilisateur.

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
| `--pool <URL>` | URL du pool pour le pool mining : Stratum pour Xelis/DragonX/Parano1d/Firo, lignes JSON du protocole `seine` pour Blocknet | **Requis (pool)** |
| `--address <ADDR>` | Adresse wallet du coin miné, pour recevoir les récompenses | **Obligatoire** |
| `--coin <NOM>` | Coin à miner : `xelis`, `dragonx`, `parano1d`, `blocknet`, `firo` | déduit de l'adresse |
| `--worker <NAME>` | Nom du worker visible sur le pool | `worker1` |
| `--threads <N>` | Nombre de threads CPU pour le minage | auto (nb cœurs, GPU : 0) |
| `--gpu` | Active le minage GPU OpenCL (désactive le CPU par défaut) | désactivé |
| `--gpu-ids <IDS>` | IDs des GPU à utiliser (séparés par virgule) | `0` |
| `--gpu-intensity <N>` | Work-items par GPU (~531 KB VRAM chacun). Plus = plus de hashrate | auto (2048) |
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

Réglages mesurés sur `pool.dragonx.is:3433` (Ryzen 7 5700X, 2026-11-06) :

- **46-60 H/s à 4 threads, 140-165 H/s à 16** — RandomX se compte en hachures par
  seconde, pas en kH/s : ~10 H/s par thread. La RAM n'est pas le facteur limitant :
  266 MiB residents à 1 thread, 281 MiB à 8, le cache d'époque de 256 MiB étant partagé.
- **16 parts acceptées, 0 rejetée en 671 s** à 4 threads (54 jobs suivis, 23 hauteurs,
  première part 115 s après la connexion). Le pool a monté la difficulté de part de
  1 072 à 2 145 puis 15 000 en cours de route, et à 3 750 lors du second essai
  (16 threads, 260 s, 1 part acceptée / 0 rejetée) : le client s'adapte à chaque
  `target` sans reconnexion, et c'est ce qui prouve que la cible lue est la bonne.
- **Une digest de référence ancré** : le header de 140 octets `(0..140)` haché sous la
  clé d'exemple donne `5607a326eb9337d519c400c577891ec43fab02997deda2e42baf8b41f74981c0`,
  identique par le JIT, par l'interpréteur (`flags = 0`) et par la voie pipelinée
  `hash_first`/`hash_next`/`hash_last`. Contre-preuve : passer `RANDOMX_JUMP_BITS`
  de 8 à 7 — une macro que le build ne vérifie pas — fait rougir ce test, donc le
  digest atteste bien des paramètres Hush et pas d'un chemin de repli.
- **Un défaut relevé par la preuve live elle-même** : au troisième essai (4 threads,
  301 s, 8 parts acceptées), le pool a répondu `Duplicate share` à une part déjà payée.
  Le compteur de nonces repartait de l'indice du worker à chaque job, donc un simple
  changement de `target` sur la même hauteur re-soumettait les nonces déjà comptés. Le
  curseur est maintenant unique, partagé par tous les threads, et ne revient jamais en
  arrière ; le test de non-régression rougit `[0, 1, 0, 1]` si on lui rend l'ancien
  comportement. Re-épreuve sur le binaire corrigé : **20 parts acceptées, 0 rejetée en
  641 s** à 4 threads (46 jobs, 18 hauteurs, 51-61 H/s, `target` passé de 1 000 à 7 500
  en cours de hauteur), et aucun nonce soumis deux fois.
- `--gpu` est ignoré (RandomX est CPU-only par construction), et rien ici ne vient du
  code GPL d'XMRig : les cinq paramètres Hush sont portés dans l'arbre RandomX
  BSD-3-Clause vendoré (`randomx-c/`), patché de 20 lignes dans un seul fichier.

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

Le moteur CPU pré-absorbe le préfixe du patron : les cinq blocs qui précèdent le nonce
atteignent le même état d'éponge pour tous les candidats, donc un candidat ne coûte plus
que **3 permutations au lieu de 8**. `cargo run --release --bin noid_bench` mesure le gain
sur le gabarit `mining.notify` réel (même machine, mêmes plages de nonces) :
**154 → 407 KH/s à 1 thread** et **1,11 → 2,90 MH/s à 16 threads**, soit ×2,6 à tous les
compteurs de threads, et le checksum des digests reste identique à celui du noyau
d'origine — la vitesse ne vient pas d'un hash différent. Contre-preuves : les quatre tests
de `src/noid/batch.rs` comparent octet à octet les 256 nonces d'un lot au chemin packé
d'origine et au chemin scalaire de consensus.

Preuve live sur `eu2.innovlab.cc:19601` (TLS, 16 threads, 704 s, 7 novembre 2026) :
**59 parts acceptées, aucune rejetée par le pool**, 23 jobs suivis de la hauteur 287 886
à 287 908, hashrate affiché 2,70-3,02 MH/s aux fenêtres de 10 s et cycles
« canonical-tip-unverified » repris sans reconnexion. Le compteur « Rejected » de la
session vaut 1 : une part résolue sur un job supersédé pendant les 200 ms de la pause a
été écartée par le client avant l'envoi, ce qui est le comportement attendu. Le binaire
Linux statique (`HashPulse-linux-x86_64`, sha256 `9632dc3f4e4d…`) rend la même chose :
2,99-3,14 MH/s à 16 threads, 8 parts acceptées et 0 rejetée en 160 s
(`proofs/noid-linux-live.log`).

Le GPU n'accélère pas ce coin, et ce n'est pas un choix de confort : le mineur de
référence `noid-gpu-amd` ne publie que des binaires, et l'arborescence Apache-2.0 du dépôt
Parano1d (1616 entrées, branche `main`, vérifiée le 2026-11-07 — `proofs/noid-tree.json`)
ne contient aucun chemin `.cl` / `gpu` / `opencl`. Le kernel dont ils sortent n'a donc ni
source ni licence, et il ne peut pas entrer dans HashPulse. Seule la technique est
reprise, ré-implémentée en Rust dans `src/noid/batch.rs` au-dessus de l'API publique des
crates vendorisés, qui restent byte-identiques à l'import.

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

Blocknet se mine en pool ou en solo. En pool, HashPulse parle le protocole JSON-lines du
mineur officiel `seine` : `--pool` est attendu ici, pas une adresse de daemon. En solo, le
minage passe par l'API HTTP du core, que le core n'ouvre que si on le lui demande : c'est
`-api <adresse>` (ex. `127.0.0.1:8332`) qui l'allume, et `--daemon` doit reprendre exactement
cette adresse.

```powershell
HashPulse.exe --coin blocknet --pool stratum+tcp://bntpool.com:3333 --address VOTRE_ADRESSE_BLOCKNET --worker rig_01
HashPulse.exe --coin blocknet --daemon http://127.0.0.1:8332 --address VOTRE_ADRESSE_BLOCKNET
```

Prérequis et réglages mesurés :

- **Un core Blocknet à jour, démarré avec son API** (solo) : `-api 127.0.0.1:8332`. Sans ce
  drapeau le core ne sert aucune route de minage et n'écrit pas de jeton. Le jeton est
  écrit sous `<répertoire du nœud>/api.cookie` (`./blocknet-data-mainnet` par défaut,
  0600, supprimé à l'arrêt) et HashPulse le lit tout seul ; `--api-token` ou
  `--api-cookie` le donnent sinon.
- **Le wallet Blocknet pour Windows se suffit à lui-même** : il démarre son propre core
  avec `--api 127.0.0.1:18432` et écrit son jeton dans
  `%APPDATA%\com.blocknet.wallet\data\api.cookie`, sans sous-répertoire de réseau.
  HashPulse sonde `BNT_CONFIG_DIR`, `~/.config/bnt` et ce répertoire, avec et sans
  sous-répertoire `mainnet` / `testnet` : `--daemon http://127.0.0.1:18432` suffit alors.
- **Ne pas lancer le minage interne du core en même temps** (`POST /api/mining/start`) :
  chaque thread Argon2id, où qu'il soit, réserve 2 GiB.
- **2 GiB de RAM par thread.** Argon2id est un PoW à mémoire : la dureté est l'arena,
  pas le temps CPU. Sans `--threads`, HashPulse choisit un thread et n'en ajoute que si
  la RAM libre physique le permet (il garde 1 GiB de marge pour le système).
- **Débit attendu : ~0,8 H/s par thread**, soit ~1,25 s par hash. À la difficulté
  actuelle du mainnet (≈ 23 M, réseau à ≈ 100 kH/s), un thread seul en solo part sur une
  dizaine de mois avant un bloc : ce mode ne se choisit que pour soutenir le réseau ou si
  l'on groupe plusieurs machines.
- **En pool, le même thread est payé en minutes.** `bntpool.com:3333` attribue une
  difficulté de part de 60 (et l'ajuste seul, observé jusqu'à 112) : sur 700 s avec un seul
  thread, **4 parts acceptées, 0 rejetée, la première 23 s après le login**, 11 jobs suivis
  sur les blocs 72664→72666. Le protocole est celui du mineur officiel `seine` (BSD-3) :
  une ligne JSON par message, `login` avec les capacités, `job` avec `header_base`, `target`
  et la plage de nonces, `submit` avec le `claimed_hash`, `refill` quand la plage est
  épuisée, keepalive `\n` après 10 s d'inactivité.
- **Au-delà de la hauteur 103 680**, le PoW de Blocknet devient du BLAKE2b-512 et le source
  amont marque ces paramètres non définitifs : HashPulse refuse l'ouvrage (solo comme pool)
  et ne déclare pas `pow_blake2b_v1` au login, donc un pool qui bascule lui demande une mise
  à jour au lieu de voir empiler des parts rejetées.
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

Le wallet payé dépend du coin miné : l'adresse réellement utilisée est affichée au
démarrage (`Dev Fee: 1.0% -> ...`). En solo Blocknet, le core reçoit un `blocktemplate`
dont le coinbase pointe vers ce wallet pendant les quelques secondes de la phase
développeur, puis revient à votre adresse. En pool Blocknet, la bascule renvoie un `login`
portant ce wallet, puis un `login` portant le vôtre au retour : mesuré sur bntpool.com,
`Switching to dev mining` à 02:23:53 et `Switching back to user mining` à 02:24:06, la
session et le compteur de parts survivant aux deux reconnexions (heures UTC). Un wallet mainnet est
vérifié à la construction : une adresse testnet ou une faute de frappe est refusée avant le
premier bloc perdu.

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

**HashPulse v2.0.0** — Mineur Xelis, DragonX, Parano1d, Blocknet et Firo avec support CPU & GPU OpenCL

XelisHash V3 · Dev Tax configurable · Solo & Pool Mining

</div>
