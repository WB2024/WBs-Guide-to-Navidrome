# 🎵 Le Guide Ultime de Configuration Navidrome

Un guide complet et accessible aux débutants pour tirer le meilleur parti de votre serveur de musique auto-hébergé Navidrome — de l'installation à la gestion parfaite des métadonnées, en passant par la lecture et bien plus encore.

_Basé sur les recommandations de la communauté et les meilleures pratiques._

## 📋 Table des Matières

-   [Vue d'ensemble — Que construisons-nous ?](https://www.google.com/search?q=%23-vue-densemble--que-construisons-nous-)
-   [Étape 1 : Installer Navidrome (Docker)](https://www.google.com/search?q=%23-%C3%A9tape-1--installer-navidrome-docker)
-   [Étape 2 : Taguer votre musique avec MusicBrainz Picard](https://www.google.com/search?q=%23-%C3%A9tape-2--taguer-votre-musique-avec-musicbrainz-picard)
-   [Étape 3 : Corriger les tags de genre](https://www.google.com/search?q=%23-%C3%A9tape-3--corriger-les-tags-de-genre)
-   [Étape 4 : Configurer les applications de lecture](https://www.google.com/search?q=%23-%C3%A9tape-4--configurer-les-applications-de-lecture)
-   [Étape 5 : Accès à distance avec Tailscale](https://www.google.com/search?q=%23-%C3%A9tape-5--acc%C3%A8s-%C3%A0-distance-avec-tailscale)
-   [Étape 6 : Ajouter des stations de radio](https://www.google.com/search?q=%23-%C3%A9tape-6--ajouter-des-stations-de-radio)
-   [Conseils sur la qualité audio](https://www.google.com/search?q=%23-conseils-sur-la-qualit%C3%A9-audio)
-   [Alternatives envisagées](https://www.google.com/search?q=%23-alternatives-envisag%C3%A9es)
-   [Liens de référence rapide](https://www.google.com/search?q=%23-liens-de-r%C3%A9f%C3%A9rence-rapide)
    

----------

## 🗺️ Vue d'ensemble — Que construisons-nous ?

Voici un aperçu de la pile logicielle complète :
| **Composant** | **Rôle** | **Lien** |
|--|--|--|
| **Navidrome** | Serveur de musique auto-hébergé | [navidrome.org](https://www.google.com/search?q=https://www.navidrome.org) |
| **MusicBrainz Picard** | Auto-étiquetage des métadonnées (artiste, album, piste, etc.) | [picard.musicbrainz.org](https://www.google.com/search?q=https://picard.musicbrainz.org) |
| **Picard Filenaming Script Generator** | Générer des scripts de renommage Picard sans coder | [GitHub](https://www.google.com/search?q=https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator) |
| **Essentia Music Tagger** | Étiquetage des genres/ambiances par IA (remplace les genres de Picard) | [GitHub](https://www.google.com/search?q=https://github.com/WB2024/Essentia-to-Metadata) |
| **Artist Genre Metadata Enforcer** | Définissez votre propre genre par artiste, imposez-le partout | [GitHub](https://www.google.com/search?q=https://github.com/WB2024/Artist-Genre-Metadata-Enforcer) | 
| **Navidrome Radio Station Manager** | Ajouter des stations de radio internet à Navidrome |[GitHub](https://www.google.com/search?q=https://github.com/WB2024/Add-Navidrome-Radios) | 
| **Feishin** | Lecteur de bureau (interface type Spotify) |[GitHub](https://www.google.com/search?q=https://github.com/jeffvli/feishin) | 
| **Symfonium** | Lecteur de musique mobile (Android) | [symfonium.app](https://www.google.com/search?q=https://symfonium.app) | 
| **Tailscale** | Accès distant sécurisé (sans ouverture de ports) |[tailscale.com](https://www.google.com/search?q=https://tailscale.com) | 

```Plaintext
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Votre Musique│────▶│   Picard     │────▶│ Correction   │
│ (fichiers)   │     │ (étiquetage) │     │ des Genres   │
│              │     │              │     │ (Essentia ou │
│              │     │              │     │   Manuel)    │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                 │
                                                 ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Feishin     │◀────│  Navidrome   │◀────│ Bibliothèque │
│  (bureau)    │     │  (serveur)   │     │ Musicale     │
├──────────────┤     │              │     │ Propre       │
│  Symfonium   │◀────│              │     └──────────────┘
│  (mobile)    │     └──────┬───────┘
└──────────────┘            │
                    ┌──────┴───────┐
                    │  Tailscale   │
                    │  (distant)   │
                    └──────────────┘

```

----------

## 🐳 Étape 1 : Installer Navidrome (Docker)

### Prérequis

-   Docker et Docker Compose installés sur votre machine
-   Un dossier de bibliothèque musicale sur votre système
-   (Optionnel) Clés API Spotify et Last.fm pour les images d'artistes et le scrobbling
    

### Configuration Docker Compose

Créez un fichier `docker-compose.yml` :

```YAML
services:
  navidrome:
    image: ghcr.io/navidrome/navidrome:latest
    container_name: navidrome
    environment:
      - ND_MUSICFOLDER=/music
      - ND_PORT=4533
      - ND_SPOTIFY_ID=votre_spotify_client_id        # Pour les images d'artistes
      - ND_SPOTIFY_SECRET=votre_spotify_client_secret  # Pour les images d'artistes
      - ND_LASTFM_APIKEY=votre_lastfm_api_key         # Pour le scrobbling
      - ND_LASTFM_SECRET=votre_lastfm_secret           # Pour le scrobbling
      - ND_LOGLEVEL=info
    volumes:
      - /chemin/vers/data/navidrome:/data       # Base de données et config Navidrome
      - /chemin/vers/votre/musique:/music:ro    # Votre musique (lecture seule)
    ports:
      - "4533:4533"
    restart: unless-stopped

```

### Obtenir vos clés API

**Spotify (pour les images d'artistes et les pochettes)**

1.  Allez sur [developer.spotify.com/dashboard](https://www.google.com/search?q=https://developer.spotify.com/dashboard)
2.  Connectez-vous et créez une nouvelle application
3.  Copiez le "Client ID" et le "Client Secret"
4.  Collez-les dans `ND_SPOTIFY_ID` et `ND_SPOTIFY_SECRET`    

**Last.fm (pour le scrobbling)**

1.  Allez sur [last.fm/api](https://www.google.com/search?q=https://www.last.fm/api)
2.  Créez un nouveau compte/application API
3.  Copiez la "API Key" et le "Shared Secret"    
4.  Collez-les dans `ND_LASTFM_APIKEY` et `ND_LASTFM_SECRET`
    

### Lancer Navidrome

```Bash
docker compose up -d
```

Visitez `http://votre-ip-serveur:4533` dans votre navigateur. Au premier lancement, vous créerez un compte administrateur.

>💡 **Astuce :** Montez votre dossier de musique avec `:ro` (read-only / lecture seule) afin que Navidrome ne puisse pas modifier accidentellement vos fichiers.

----------

## 🏷️ Étape 2 : Taguer votre musique avec MusicBrainz Picard

MusicBrainz Picard est l'outil indispensable pour étiqueter automatiquement votre musique avec des métadonnées précises (artiste, album, titre, numéro de piste, pochette, etc.). Il compare votre musique à l'immense base de données MusicBrainz — qui offre également une excellente couverture pour la musique japonaise et coréenne.

### Option A : Exécuter Picard via Docker (Recommandé pour les serveurs)

Si votre musique est sur un serveur sans écran (headless), exécutez Picard en tant que conteneur Docker accessible via le web :



```YAML
services:
  picard-web:
    image: aandree5/picard-web:latest
    container_name: picard-web
    restart: unless-stopped
    ports:
      - "8086:5000"
    volumes:
      - /chemin/vers/config/picard:/picard-web:rw    # Config/état de Picard
      - /chemin/vers/votre/musique:/music:rw         # Votre musique (lecture-écriture !)
```
```Bash
docker compose up -d
```

Accédez à Picard dans votre navigateur à l'adresse `http://votre-ip-serveur:8086`.

> ⚠️ **Important :** Le volume de musique doit être en `:rw` (read-write / lecture-écriture) ici car Picard doit écrire les tags dans vos fichiers.

### Option B : Exécuter Picard sur le Bureau

Téléchargez-le sur [picard.musicbrainz.org](https://www.google.com/search?q=https://picard.musicbrainz.org) et installez-le normalement sur Windows, macOS ou Linux.

### Comment utiliser Picard (Flux de travail rapide)

1.  **Ajouter des fichiers** — Glissez-déposez votre dossier de musique dans Picard.
    
2.  **Grouper (Cluster)** — Cliquez sur "Grouper" pour rassembler les fichiers par album.
    
3.  **Rechercher (Lookup)** — Cliquez sur "Rechercher" pour trouver les correspondances dans MusicBrainz.
    
4.  **Réviser** — Vérifiez que les correspondances sont correctes (vert = bonne correspondance).
    
5.  **Enregistrer** — Cliquez sur "Enregistrer" pour écrire les tags dans vos fichiers.
    

### Générer un script de renommage de fichiers

Picard peut également renommer et réorganiser vos fichiers en fonction des métadonnées. Écrire ces scripts manuellement est complexe, utilisez donc le **Picard Filenaming Script Generator** :

📦 [WB's Picard Filenaming Script Generator](https://www.google.com/search?q=https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator)

Cet outil interactif en ligne de commande vous permet de :

-   Choisir parmi 6 préréglages prêts à l'emploi (ex: _Artiste/[Année] Album/01 - Titre.flac_)
-   Ou utiliser l'assistant de construction personnalisé pour configurer chaque détail    
-   Prévisualiser un exemple de sortie avant d'enregistrer    
-   Exporter directement dans Picard — aucune connaissance en script n'est requise
    
```Bash
# Installer et exécuter
git clone https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator.git

cd WBs-Picard-Filenaming-Script-Generator
python3 picard_script_generator.py
```

💡 **Pour la musique japonaise/coréenne :** MusicBrainz a une couverture solide pour la J-Pop, la K-Pop, les OST d'animes, et plus encore. Picard récupérera les métadonnées correctes, y compris les titres dans la langue d'origine lorsqu'ils sont disponibles.

----------

## 🎸 Étape 3 : Corriger les tags de genre

Picard est excellent pour la plupart des métadonnées, mais ses tags de genre sont souvent génériques ou incohérents (par exemple, étiqueter tout comme "Rock" ou "Pop"). Il existe deux approches pour corriger cela :

### Option A : Étiquetage des genres par IA avec Essentia (Automatique)

📦 [Essentia Music Tagger](https://www.google.com/search?q=https://github.com/WB2024/Essentia-to-Metadata)

Cet outil analyse la forme d'onde audio réelle à l'aide de l'apprentissage automatique (Machine Learning) pour prédire des genres et des ambiances précis. Aucune connexion internet n'est requise après l'installation — tout s'exécute localement.

**Ce qu'il fait :**
-   Classifie parmi 400 catégories de genres (taxonomie Discogs)
-   Détecte les ambiances (énergique, sombre, joyeux, agressif, etc.)
-   Écrit les tags `GENRE` et `MOOD` directement dans vos fichiers
-   Supporte le FLAC et le MP3
    

**Démarrage rapide :**

```Bash
# Cloner et configurer
git clone https://github.com/WB2024/Essentia-to-Metadata.git
cd Essentia-to-Metadata
# Créer un environnement virtuel
python3 -m venv venv
source venv/bin/activate
# Installer les dépendances
pip install essentia-tensorflow mutagen numpy
# Télécharger les modèles ML (~87 Mo)
bash download_models.sh
# Exécuter (faites toujours un test à blanc - dry run - d'abord !)
python tag_music.py
```

**Ou via Docker :**

```Bash
# Mode CPU
docker compose build essentia-tagger
MUSIC_DIR=/chemin/vers/musique docker compose run --rm essentia-tagger
# Test à blanc (dry run) d'abord !
MUSIC_DIR=/chemin/vers/musique docker compose run --rm essentia-tagger /music --auto --dry-run
```

**Exemple de sortie :**

```platintext
Entrée : 2Pac - Temptations.flac
Sortie : GENRE = Hip-Hop - Gangsta; Hip-Hop - East Coast Hip Hop
         MOOD  = Energetic; Dark
```

🧪 **Exécutez toujours en mode "dry-run" en premier pour prévisualiser les changements !**

### Option B : Imposition manuelle des genres (Vos règles)

📦 [Artist Genre Metadata Enforcer](https://www.google.com/search?q=https://github.com/WB2024/Artist-Genre-Metadata-Enforcer)

Si vous préférez vos propres définitions de genres plutôt que ce qu'un algorithme ou une base de données décide, cet outil vous permet de définir un genre une seule fois par artiste et de l'appliquer à toute sa discographie.

**Pourquoi utiliser ceci ?**

-   Vous décidez que tout 2Pac est du "Hip-Hop", un point c'est tout.
-   Plus d'incohérences entre "Hip Hop", "Hip-Hop" et "Rap".    
-   Définissez une fois, imposez partout — même sur des milliers de pistes.
    

**Démarrage rapide :**

```Bash
# Installer la dépendance
sudo apt install python3-mutagen   # Debian/Ubuntu
# ou : pip install mutagen

# Télécharger et installer
sudo curl -o /usr/local/bin/music-genre-enforcer \
  https://raw.githubusercontent.com/WB2024/Artist-Genre-Metadata-Enforcer/main/music-genre-enforcer.py
sudo chmod +x /usr/local/bin/music-genre-enforcer

# Exécuter
music-genre-enforcer
```

**Flux de travail interactif :**

```Plaintext
Genre pour [Aphex Twin] (ou l/p/?/s): p
Genres existants :
  [1] Electronic - Ambient
  [2] Hip-Hop
  [3] Post-Punk
  [4] Rock - Art
Choisissez le numéro (vide pour annuler) : 1
  -> Défini [Aphex Twin] = Electronic - Ambient
```
**Structure de dossiers attendue :**
```plaintext
/votre/bibliotheque/musicale/ 
├── A/ │
├── Aphex Twin/
│   │ ├── [1992] Selected Ambient Works 85-92/
│   │ │ ├── 01 - Xtal.flac 
├── B/ 
│   ├── Bowie, David/ 
├── #/ 
│   └── 2Pac/ 
└── Artistes Divers/
```

|**Fonctionnalité**  | **Essentia (Option A)**  | **Genre Enforcer (Option B)** |
|--|--|--|
| **Automatisation** | Entièrement automatique (IA) | Semi-automatique (vous définissez, il applique) |
| **Précision**| Basée sur l'analyse audio | Basée sur votre préférence personnelle |
| **Idéal pour**| Grandes bibliothèques, découverte | Bibliothèques curatées, cohérence |
| **Multi-genre** | Oui (configurable) | Un seul genre par artiste |

>💡 **Conseil d'expert :** Vous pouvez utiliser les deux — lancez Essentia en premier pour une base, puis utilisez le Genre Enforcer pour écraser des artistes spécifiques avec vos labels préférés.

----------

## 🎧 Étape 4 : Configurer les applications de lecture

### Bureau : Feishin

📦 [Feishin](https://www.google.com/search?q=https://github.com/jeffvli/feishin) — Un lecteur de musique de bureau moderne, de type Spotify.

**Pourquoi Feishin ?**

-   Interface magnifique qui ressemble à Spotify.
-   Se connecte directement à Navidrome (et Jellyfin).
-   Deux moteurs de lecture : MPV (natif de haute qualité) ou lecteur Web.
-   Playlists intelligentes, paroles, scrobbling, thèmes.
-   Disponible sur Windows, macOS et Linux.
    

**Installation :**

1.  Téléchargez-le depuis [github.com/jeffvli/feishin/releases](https://www.google.com/search?q=https://github.com/jeffvli/feishin/releases).
2.  Ouvrez Feishin → Add Server → Entrez l'URL de votre Navidrome et vos identifiants.
    

### 🔊 Configuration Audio Optimale (Avancé)

Pour la meilleure qualité audio sur bureau :

-   Utilisez **MPV** comme moteur de lecture dans les paramètres de Feishin.
-   Connectez un **DAC USB** (Convertisseur Numérique-Analogique) à votre ordinateur.
-   Réglez la sortie audio sur **WASAPI (Windows)** dans les paramètres de Feishin/MPV.
-   Activez le **mode audio exclusif** — cela empêche Windows de rééchantillonner votre audio.
    
   > _Cela contourne entièrement le mixeur audio de Windows, envoyant un audio "bit-perfect" à votre DAC._
    

### Mobile : Symfonium (Android)

📦 [Symfonium](https://www.google.com/search?q=https://symfonium.app) — Le meilleur lecteur de musique Android pour les serveurs auto-hébergés.

**Pourquoi Symfonium ?**

-   Se connecte à Navidrome (ainsi qu'à Plex, Jellyfin, Emby, stockage cloud, etc.).
-   Mise en cache hors-ligne — téléchargez de la musique pour quand vous n'avez pas internet.
-   Lecture sans blanc (gapless), gain de lecture (replay gain) et support audio haute résolution.
-   Diffusion vers Chromecast, DLNA, Sonos, etc.
-   Support Android Auto & Wear OS.
-   Achat unique, pas d'abonnement, pas de publicités.
    

----------

## 🌐 Étape 5 : Accès à distance avec Tailscale

Vous voulez écouter votre musique quand vous n'êtes pas chez vous ? N'exposez pas votre serveur sur internet avec une redirection de port — utilisez Tailscale à la place.

📦 [Tailscale](https://www.google.com/search?q=https://tailscale.com) — Crée une connexion réseau privée sécurisée (VPN) entre vos appareils sans configuration complexe.

**Comment ça marche**

1.  Installez Tailscale sur votre serveur (où Navidrome s'exécute).
2.  Installez Tailscale sur votre téléphone/ordinateur portable (où vous écoutez).
3.  Les deux appareils reçoivent une adresse IP privée Tailscale.
4.  Accédez à Navidrome en utilisant l'IP Tailscale — crypté, sécurisé, sans ouverture de port.
    

**Configuration**

```bash
# Sur votre serveur (Debian/Ubuntu)
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# Notez l'IP Tailscale fournie (ex: 100.x.y.z)

```

Installez l'application Tailscale sur votre téléphone/portable depuis [tailscale.com/download](https://www.google.com/search?q=https://tailscale.com/download).

Ensuite, dans Feishin ou Symfonium, réglez l'URL de votre serveur sur :

`http://100.x.y.z:4533`

🔒 **Pourquoi Tailscale ?** Aucun port à ouvrir, pas de DNS dynamique à configurer, tunnel WireGuard crypté, et une offre gratuite généreuse pour un usage personnel.

----------

## 📻 Étape 6 : Ajouter des stations de radio

📦 [Navidrome Radio Station Manager](https://www.google.com/search?q=https://github.com/WB2024/Add-Navidrome-Radios)

Un outil CLI pour rechercher, parcourir et ajouter des stations de radio internet à votre serveur Navidrome à partir de la base de données communautaire Radio-Browser.

**Installation**

```bash
# Installer la dépendance
pip3 install requests

# Télécharger et installer
curl -o /usr/local/bin/navidrome-radio \
  https://raw.githubusercontent.com/WB2024/Add-Navidrome-Radios/main/navidrome-radio.py
sudo chmod +x /usr/local/bin/navidrome-radio

# Exécuter
navidrome-radio

```

**Utilisation**

Au premier lancement, entrez le chemin de votre base de données Navidrome (ex: `/chemin/vers/navidrome/data/navidrome.db`).

Ensuite, recherchez et ajoutez des stations :

1.  Recherchez par nom, genre, pays, ou parcourez les stations les mieux notées.
2.  Sélectionnez les stations → tapez "add" → elles apparaissent immédiatement dans Navidrome !
    

>💡 **Utilisateurs Docker :** Le chemin de votre base de données est sur l'hôte, pas à l'intérieur du conteneur. Vérifiez les volumes de votre `docker-compose.yml` pour le trouver.

----------

## 🎶 Conseils sur la qualité audio

Ces conseils vous aideront à obtenir la meilleure expérience d'écoute :

### Construire une bibliothèque propre

**Recommandation**

**Détails**

**Format Audio**

Le **FLAC** (lossless) est idéal. Visez au minimum 16-bit / 44.1kHz.

**Format Unique**

Gardez tout dans un seul format pour la cohérence (ex: tout en FLAC).

**Stockage**

Si le stockage est limité et la qualité n'est pas prioritaire, le MP3 convient.

**Organisation**

Utilisez Picard pour renommer/organiser les fichiers dans une structure cohérente.

### Recommandations Matérielles

-   **DAC :** Procurez-vous un DAC USB dédié au lieu d'utiliser l'audio de votre carte mère.
    
-   **Enceintes/Casque :** Investissez dans la qualité — c'est là que vous entendrez la plus grande différence.
    
-   **Amplificateur :** Adaptez votre ampli à vos enceintes/casque.
    

### Chaîne Audio Logicielle (Bureau)

`Feishin → Moteur MPV → WASAPI (Exclusif) → DAC USB → Ampli → Enceintes`

Cette chaîne garantit :

✅ Sortie audio Bit-perfect
✅ Pas de rééchantillonnage par le mixeur audio Windows
✅ Accès matériel direct
✅ Qualité maximale de vos fichiers

----------

## 🔄 Alternatives envisagées

-   **Lidarr :** Gestionnaire automatique de bibliothèque musicale (type Sonarr/Radarr). Avis partagé — excellent pour les autres applications *arr, mais peut être frustrant pour la musique. Non recommandé pour les débutants.
    
-   **beets :** Outil CLI d'étiquetage et de gestion de bibliothèque. Excellent et puissant, mais uniquement en ligne de commande. Si vous êtes à l'aise avec un terminal, c'est un excellent choix. Des interfaces graphiques existent mais ne sont pas très stables.
    
-   **MusicBrainz Picard :** Étiqueteur avec interface graphique. **Recommandé** — interface conviviale, excellente couverture de base de données, fonctionne bien pour toutes les langues, y compris le japonais et le coréen.
    

----------

## 🔗 Liens de référence rapide

**Infrastructure de base**

-   [Navidrome](https://www.google.com/search?q=https://www.navidrome.org)
-   [MusicBrainz Picard](https://www.google.com/search?q=https://picard.musicbrainz.org)
-   [Feishin (Lecteur Bureau)](https://www.google.com/search?q=https://github.com/jeffvli/feishin)
-   [Symfonium (Lecteur Mobile)](https://www.google.com/search?q=https://symfonium.app)
-   [Tailscale (Accès Distant)](https://www.google.com/search?q=https://tailscale.com)
    

**Outils d'amélioration**

-   [Picard Filenaming Script Generator](https://www.google.com/search?q=https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator)
 -   [Essentia Music Tagger (Genres IA)](https://www.google.com/search?q=https://github.com/WB2024/Essentia-to-Metadata)
 -   [Artist Genre Metadata Enforcer](https://www.google.com/search?q=https://github.com/WB2024/Artist-Genre-Metadata-Enforcer)
 -   [Navidrome Radio Station Manager](https://www.google.com/search?q=https://github.com/WB2024/Add-Navidrome-Radios)
    

----------

## ✅ Ordre d'Opérations Suggéré

Voici la séquence recommandée pour tout configurer à partir de zéro :
```plaintext
1.  🐳 Déployer Navidrome via Docker.
2.  🏷️ Taguer votre musique avec Picard (réglez d'abord les métadonnées).
3.  📁 Générer un script de renommage pour organiser votre bibliothèque.
4.  🎸 Corriger les genres avec Essentia et/ou Genre Enforcer.
5.  🎧 Installer Feishin (bureau) et/ou Symfonium (mobile).
6.  🌐 Configurer Tailscale pour l'accès à distance.
7.  📻 Ajouter des stations de radio.
8.  🔊 Optimiser votre chaîne matérielle audio.
9.  🎶 Profitez de votre musique !
```    

Bonne écoute ! 🎶
