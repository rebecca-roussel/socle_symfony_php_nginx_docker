# socle_symfony_php_nginx_docker

Socle de configuration reproductible pour démarrer des projets Symfony avec **PHP-FPM**, **Nginx** et **Docker**.

---

## Objectif

Ce dépôt a pour but de fournir une base technique exploitable pour démarrer un projet Symfony dans un environnement conteneurisé.

Il sert à démarrer avec :

- **Symfony 7.4** : framework PHP de l'application
- **PHP 8.4-FPM** : moteur d'exécution du code PHP
- **Nginx 1.27** : serveur web en façade
- **Docker** : conteneurisation des services
- **Docker Compose** : orchestration de l'environnement local
- **phpDocumentor** : génération d'une documentation technique à partir du code source et des DocBlocks

> Pour Symfony 7.4, la version minimale requise de PHP est **8.2**.

---

## Avant-propos

Ce dépôt fournit un socle technique réutilisable pour démarrer un projet Symfony à partir d'une configuration valide, d'un environnement prêt à l'emploi et d'une arborescence de travail structurée.

L'objectif est de poser dès le départ des fondations saines pour le développement, de limiter les erreurs de configuration et de disposer d'une base homogène et reproductible d'un projet à l'autre.

### Utilités de ce dépôt

- lancer un environnement Symfony conteneurisé
- séparer correctement Nginx et PHP-FPM
- disposer d'une base de configuration lisible
- réutiliser facilement ce socle comme point de départ pour un nouveau projet
- possibilité d'y ajouter des services (ex : SGBD, NoSQL, API, etc)
- générer une documentation technique du code avec phpDocumentor
- conserver un fichier de configuration de documentation versionné avec le projet

---

## Prérequis

### Système d'exploitation cible

Ce socle est prévu en priorité pour l'environnement suivant :

- **Windows 11**
- **WSL 2** avec une distribution Linux
- **Ubuntu** dans WSL 2
- **Docker Desktop** avec intégration WSL activée

> Cette base permet de travailler dans un environnement Linux tout en conservant Docker Desktop comme moteur Docker sur Windows.

### Dépendances techniques obligatoires

Avant d'utiliser ce dépôt, les éléments suivants doivent être installés :

#### À installer sur Windows

- **Docker Desktop** avec intégration **WSL 2** activée

#### À installer dans Ubuntu (WSL)

- **PHP**
- extensions PHP minimales requises : **Ctype**, **iconv**, **PCRE**, **Session**, **SimpleXML** et **Tokenizer**
- **Composer**
- **Git**
- **curl**

### Dépendance recommandée

- **Symfony CLI** : recommandée pour vérifier les prérequis avec `symfony check:requirements` et pour initialiser un nouveau projet Symfony. Elle n'est pas indispensable si le projet est créé via Composer uniquement.

> La Symfony CLI n'est pas nécessaire pour cloner ce dépôt, installer les dépendances ou démarrer l'environnement Docker.

---

## Installations à effectuer avant le clone du dépôt

### 1. Installer WSL

À exécuter dans **PowerShell en tant qu'administrateur** :

```powershell
wsl --install
```

Commande utile pour vérifier les distributions disponibles :

```powershell
wsl --list --online
```

Si nécessaire, installer explicitement Ubuntu :

```powershell
wsl --install -d Ubuntu
```

> Après l'installation, redémarrer la machine si Windows le demande.

### 2. Installer Docker Desktop

À exécuter dans PowerShell :

```powershell
winget install Docker.DockerDesktop
```

Après l'installation :

- démarrer Docker Desktop
- ouvrir **Settings > General**
- vérifier que l'option **Use WSL 2 based engine** est activée
- ouvrir **Settings > Resources > WSL Integration**
- activer l'intégration pour la distribution Ubuntu utilisée

> Ce socle repose sur Docker Desktop avec backend WSL 2.
> Il est préférable d'éviter une installation parallèle de Docker Engine directement dans la distribution Ubuntu WSL, afin de limiter les conflits.

### 3. Ouvrir Ubuntu dans WSL 2

Depuis PowerShell :

```powershell
wsl
```

### 4. Mettre à jour Ubuntu

À exécuter dans le terminal Ubuntu (WSL 2) :

```bash
sudo apt update && sudo apt upgrade -y
```

### 5. Installer les outils de base dans Ubuntu

À exécuter dans le terminal Ubuntu (WSL 2) :

```bash
sudo apt install -y curl git composer php8.4-cli php8.4-common php8.4-curl php8.4-intl php8.4-mbstring php8.4-xml php8.4-zip
```

### 6. Installer Symfony CLI dans Ubuntu

À exécuter dans le terminal Ubuntu (WSL 2) :

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
sudo apt install -y symfony-cli
symfony -v
```

---

## Vérification minimale avant le clonage de ce dépôt

À exécuter dans le terminal Ubuntu (WSL 2) :

```bash
git --version
php -v
composer --version
docker version
docker compose version
```

Si la Symfony CLI est installée, vérifier également les prérequis Symfony :

```bash
symfony -v
symfony check:requirements
```

> Si ces commandes répondent correctement, l'environnement minimal requis est prêt et le dépôt peut être cloné, sinon voir la section **Débogage** ci-dessous.

### Note sur l'avertissement PDO

Lors de l'exécution de `symfony check:requirements`, un avertissement peut apparaître si aucun pilote PDO n'est installé. Cet avertissement est normal dans le cadre de ce socle, car il ne présuppose pas de base de données relationnelle par défaut.

> - **PDO** fournit une interface commune d'accès aux bases de données relationnelles.
> - Un **pilote PDO** est nécessaire pour se connecter à un moteur précis.
> - Sans pilote PDO, Symfony peut démarrer, mais Doctrine ne pourra pas communiquer avec une base relationnelle.

Le pilote à installer dépend du moteur de base de données choisi pour le projet. Adapter la version du paquet PHP à la version installée sur la machine.

#### PostgreSQL

```bash
sudo apt install -y php8.4-pgsql
```

#### MySQL / MariaDB

```bash
sudo apt install -y php8.4-mysql
```

#### SQLite

```bash
sudo apt install -y php8.4-sqlite3
```

Après l'installation du pilote adapté, relancer `symfony check:requirements` pour confirmer la résolution de l'avertissement.

#### Contraintes à prendre en compte avec ce socle

Le choix du SGBD doit rester cohérent avec l'ensemble de la stack technique. Il faut notamment vérifier :

- la disponibilité du pilote PDO correspondant
- la compatibilité avec Doctrine si le projet utilise un accès aux données via Doctrine DBAL ou ORM
- la cohérence de la variable de connexion, par exemple `DATABASE_URL`
- la présence éventuelle d'un service de base de données dans `compose.yaml` si la base doit être conteneurisée
- les différences de fonctionnement entre moteurs, notamment sur les types SQL, les contraintes, les index ou certaines requêtes spécifiques

---

## Cloner le dépôt

À exécuter dans le terminal Ubuntu (WSL 2) :

```bash
git clone <url_du_depot>
cd socle_symfony_php_nginx_docker
composer install
```

> Cette étape permet de récupérer le socle technique du projet et d'installer les dépendances PHP définies dans `composer.json`.

### Pourquoi lancer `composer install` localement ?

Dans cette architecture, la commande `composer install` est exécutée sur la machine hôte, c'est-à-dire dans Ubuntu sous WSL 2.

Le dossier `vendor/` est donc généré sur l'hôte, puis partagé avec le conteneur PHP via le montage de fichiers défini dans `compose.yaml`.

> Cette organisation présente deux avantages :
>
> - l'éditeur de code local peut lire directement le dossier `vendor/` pour l'autocomplétion et l'analyse du code ;
> - le conteneur PHP continue d'exécuter l'application avec ces mêmes dépendances, car il voit les fichiers partagés depuis l'hôte.

---

## Documenter le code avec phpDocumentor

Le socle intègre un fichier `phpdoc.dist.xml` afin de préparer la génération d'une documentation technique du projet.

### Pourquoi utiliser phpDocumentor ?

phpDocumentor permet d'analyser le code source PHP et les DocBlocks pour produire une documentation d'API exploitable.

Son intérêt dans ce socle est multiple :

- mieux comprendre la structure réelle du projet
- rendre les classes, méthodes et responsabilités plus lisibles
- documenter le code au fil du développement
- conserver une configuration de documentation versionnée avec le dépôt
- faciliter la reprise du projet ou sa transmission

> L'objectif n'est pas seulement de documenter pour produire des pages HTML, mais aussi d'encourager une écriture du code plus claire, plus rigoureuse et plus explicable.

### Pourquoi le fichier `phpdoc.dist.xml` est-il présent à la racine ?

Le fichier `phpdoc.dist.xml` correspond au fichier de configuration recommandé par phpDocumentor.

Placée à la racine du projet, cette configuration peut être versionnée avec le dépôt afin de conserver des réglages partagés d'un environnement à l'autre.

### Générer la documentation avec Docker

Comme ce socle utilise déjà Docker, il est possible d'utiliser directement l'image officielle de phpDocumentor sans installation locale supplémentaire.

À exécuter à la racine du projet :

```bash
docker run --rm -v "$(pwd):/data" phpdoc/phpdoc:3
```

> phpDocumentor reste l'outil qui génère la documentation.
> L'image Docker officielle fournit simplement un environnement prêt à l'emploi qui contient déjà phpDocumentor et ses dépendances.

La documentation générée est placée dans le dossier `docs/api/` à la racine du projet.

---

## Comment le socle a été initialisé ?

Cette section explique comment le squelette Symfony de base a été initialisé lors de la création du socle.  
Elle sert de référence technique pour comprendre l'origine de l'arborescence du projet, mais ne constitue pas une étape obligatoire lors de l'utilisation courante du dépôt cloné.

Avant de créer le projet, il faut disposer d'un outil de génération.  
Deux approches sont possibles :

- utiliser la **CLI Symfony**
- utiliser **Composer**

> Dans le cadre de ce socle, la CLI Symfony a été utilisée pour créer le premier squelette du projet.

### 1. Vérifier les prérequis du poste

À exécuter dans le terminal Ubuntu :

```bash
symfony check:requirements
```

### 2. Générer le squelette Symfony

Pour une application web classique, avec les paquets utiles à un projet web :

```bash
symfony new nom_du_projet --version="7.4.*" --webapp
cd nom_du_projet
```

> Cette commande crée une base Symfony adaptée à une application web.
> Pour un socle plus léger, destiné par exemple à une API, un microservice ou une application console, il est possible d'utiliser la commande sans l'option `--webapp`.

### Alternative sans la CLI Symfony

Si la CLI Symfony n'est pas utilisée, le projet peut aussi être créé avec Composer :

```bash
composer create-project symfony/skeleton:"7.4.*" nom_du_projet
cd nom_du_projet
composer require webapp
```

### Arborescence de base attendue

La structure exacte peut varier selon les paquets installés, mais la structure par défaut d'un projet Symfony comprend notamment les éléments suivants :

```text
.
├── assets
│   ├── app.js
│   ├── controllers
│   │   ├── csrf_protection_controller.js
│   │   └── hello_controller.js
│   ├── controllers.json
│   ├── stimulus_bootstrap.js
│   └── styles
│       └── app.css
├── bin
│   ├── console
│   └── phpunit
├── composer.json
├── composer.lock
├── compose.yaml
├── config
│   ├── bundles.php
│   ├── packages
│   │   ├── asset_mapper.yaml
│   │   ├── cache.yaml
│   │   ├── csrf.yaml
│   │   ├── debug.yaml
│   │   ├── doctrine_migrations.yaml
│   │   ├── doctrine.yaml
│   │   ├── framework.yaml
│   │   ├── mailer.yaml
│   │   ├── messenger.yaml
│   │   ├── monolog.yaml
│   │   ├── notifier.yaml
│   │   ├── property_info.yaml
│   │   ├── routing.yaml
│   │   ├── security.yaml
│   │   ├── translation.yaml
│   │   ├── twig.yaml
│   │   ├── ux_turbo.yaml
│   │   ├── validator.yaml
│   │   └── web_profiler.yaml
│   ├── preload.php
│   ├── reference.php
│   ├── routes
│   │   ├── framework.yaml
│   │   ├── security.yaml
│   │   └── web_profiler.yaml
│   ├── routes.yaml
│   └── services.yaml
├── docker
│   ├── nginx
│   │   └── default.conf
│   └── php
│       └── Dockerfile
├── docs
│   └── api
├── .dockerignore
├── .editorconfig
├── .env
├── .env.test
├── .gitignore
├── importmap.php
├── migrations
│   └── .gitignore
├── phpdoc.dist.xml
├── phpunit.dist.xml
├── public
│   └── index.php
├── README.md
├── src
│   ├── Controller
│   │   ├── AccueilController.php
│   │   └── .gitignore
│   ├── Entity
│   │   └── .gitignore
│   ├── Kernel.php
│   └── Repository
│       └── .gitignore
├── symfony.lock
├── templates
│   ├── accueil
│   │   └── index.html.twig
│   └── base.html.twig
├── tests
│   └── bootstrap.php
└── translations
    └── .gitignore
```

Le répertoire `public/` correspond au point d'entrée web de l'application.
Il contient le fichier `index.php`, qui sert de front controller.

---

## Définition des dossiers et fichiers de configuration

Pour lire correctement ce socle, il est utile de distinguer deux niveaux :

- **l'applicatif**, c'est-à-dire le projet Symfony lui-même
- **l'infrastructure**, c'est-à-dire les éléments qui permettent d'exécuter ce projet dans un environnement conteneurisé

### Ce qui relève de l'applicatif

`bin/`  
Contient les exécutables du projet Symfony, notamment `console`, qui permet d'utiliser les commandes du framework en ligne de commande.

`config/`  
Regroupe les fichiers de configuration internes de Symfony. Ce dossier contient les réglages du framework, des services, des routes et des différents environnements.

`public/`  
Correspond au répertoire exposé par le serveur web. Il contient le point d'entrée de l'application Symfony, situé dans `public/index.php`, ainsi que les ressources publiques du projet.

`src/`  
Contient le code source PHP de l'application. C'est dans ce dossier que se trouvent les contrôleurs, services, classes métier et autres composants applicatifs.

`templates/`  
Contient les vues Twig utilisées pour générer le rendu HTML des pages.

`var/`  
Contient les fichiers temporaires générés par Symfony, comme le cache et certains journaux d'exécution.

`vendor/`  
Contient les dépendances PHP installées par Composer. Ce dossier est généré lors de l'exécution de `composer install`.

`composer.json`  
Décrit le projet PHP et ses dépendances. Ce fichier définit également certaines commandes et réglages utilisés par Composer.

`composer.lock`  
Fige les versions exactes des dépendances installées. Il garantit une installation cohérente d'un environnement à l'autre.

`.env`  
Contient les variables d'environnement de base utilisées par Symfony. Ce fichier sert de référence commune pour le fonctionnement général du projet. Il est commité et ne doit jamais contenir de secrets réels.

`.env.dev.local`  
Fichier non commité, destiné aux surcharges locales spécifiques à l'environnement de développement. Il n'est pas fourni dans ce dépôt et doit être créé manuellement sur chaque poste si des valeurs locales sont nécessaires pour l'environnement `dev`. Il est ignoré par Git.

### Ce qui relève de l'infrastructure

`docker/`  
Centralise les éléments liés à la conteneurisation du projet. Ce dossier permet de séparer clairement la configuration technique des services du code applicatif.

`docker/php/`  
Contient les éléments nécessaires à la construction du conteneur PHP, en particulier le fichier `Dockerfile` utilisé pour préparer l'environnement d'exécution de l'application.

`docker/nginx/`  
Contient la configuration du serveur Nginx. Ce dossier héberge notamment le fichier qui définit le répertoire servi par le serveur web et la transmission des requêtes PHP vers PHP-FPM.

`docker/php/Dockerfile`  
Définit l'image PHP du projet. Ce fichier permet de préparer un environnement cohérent pour exécuter Symfony dans un conteneur, avec les extensions et outils nécessaires.

`docker/nginx/default.conf`  
Contient la configuration du serveur Nginx. Ce fichier précise notamment le répertoire web servi, les règles de routage et la communication avec le service PHP-FPM.

`compose.yaml`  
Décrit les services du projet, leur orchestration, leurs volumes, leurs ports et leurs relations. Ce fichier permet de démarrer l'environnement local complet avec une seule commande.

### Documents de référence et sortie de documentation

`docs/api/`  
Contient la documentation technique générée par phpDocumentor à partir du code source et des DocBlocks. Ce dossier est produit lors de l'exécution de la commande de génération.

`phpdoc.dist.xml`  
Contient la configuration de phpDocumentor. Ce fichier permet de définir comment la documentation technique du projet doit être générée et versionnée avec le socle.

`README.md`  
Documente le rôle du dépôt, sa structure, les prérequis et les étapes de mise en place. Il sert de point d'entrée pour comprendre rapidement le fonctionnement du socle.

---

## Configurer les variables d'environnement

Avant de démarrer l'environnement, il faut vérifier quels fichiers de variables sont utilisés par le socle.

### 1. Vérifier les fichiers présents à la racine du dépôt

À exécuter dans le terminal Ubuntu (WSL 2) :

```bash
ls -la
```

> Vérifier notamment la présence des fichiers suivants :
>
> `.env`  
> `compose.yaml`

### 2. Comprendre le rôle des fichiers

Symfony charge les fichiers d'environnement dans un ordre précis, chaque fichier suivant pouvant surcharger le précédent :

- `.env` : fichier de base commité, contenant les valeurs par défaut des variables d'environnement du projet. Il ne doit jamais contenir de secrets réels.
- `.env.local` : fichier non commité, contenant des surcharges locales propres à la machine. C'est dans ce fichier que doit être définie `APP_SECRET` avec une valeur réelle, unique par projet.
- `.env.dev.local` : fichier non commité, contenant des surcharges locales propres à l'environnement `dev`. Il peut être créé manuellement sur chaque poste si des valeurs locales sont nécessaires pour cet environnement.

> Les variables d'environnement réelles définies dans le système ont toujours priorité sur les fichiers `.env`.

### 3. Adapter les variables au projet cible

Avant de lancer un nouveau projet à partir de ce socle, il faut relire et adapter les variables utiles, par exemple :

- le nom du projet
- le port exposé localement
- les paramètres de base de données
- les variables liées à l'environnement Symfony
- les variables spécifiques aux futurs services ajoutés au socle

---

## Démarrer l'environnement

Une fois le dépôt cloné, les dépendances PHP installées et les variables d'environnement vérifiées, l'environnement conteneurisé peut être démarré.

À exécuter dans le terminal Ubuntu (WSL 2) :

```bash
docker compose up --build -d
```

> Cette commande construit les images si nécessaire, crée ou recrée les conteneurs, puis démarre les services en arrière-plan.

---

## Vérifier que l'application répond

Après le démarrage des services, vérifier que les conteneurs sont actifs et que les ports sont exposés :

```bash
docker compose ps
```

> Vérifier que les services attendus sont indiqués comme démarrés et repérer le port exposé par le service Nginx.

Ouvrir ensuite le navigateur et accéder à l'URL correspondant à ce port, par exemple :

[http://localhost:8082](http://localhost:8082)

> L'URL exacte dépend du port déclaré dans `compose.yaml`.

---

## Débogage

Cette section regroupe les vérifications à effectuer lorsque l'environnement ne démarre pas correctement ou lorsqu'un service ne répond pas comme prévu.

### 1. Vérifier que Symfony sert bien le point d'entrée prévu

Le serveur web doit servir le répertoire `public/`, dont le point d'entrée est le fichier :

`public/index.php`

Si Nginx pointe vers un autre répertoire, l'application ne répondra pas correctement.

### 2. Vérifier l'état des services

```bash
docker compose ps
```

> Cette commande permet de vérifier rapidement :
>
> - si les conteneurs sont démarrés
> - si un service s'est arrêté immédiatement
> - quels ports sont exposés

### 3. Lire les logs

Afficher les logs de l'ensemble des services :

```bash
docker compose logs
```

Suivre les logs en temps réel :

```bash
docker compose logs -f
```

Afficher les logs d'un seul service :

```bash
docker compose logs nginx
docker compose logs php
```

> Les logs permettent d'identifier :
>
> - une erreur de configuration Nginx
> - un problème de démarrage du service PHP
> - une erreur de chemin ou de montage de volume
> - un problème lié au port exposé

### 4. Entrer dans un conteneur

Ouvrir un terminal dans le service PHP :

```bash
docker compose exec php sh
```

Ouvrir un terminal dans le service Nginx :

```bash
docker compose exec nginx sh
```

> Cette vérification permet par exemple de :
>
> - confirmer la présence des fichiers attendus dans le conteneur
> - vérifier les chemins réellement utilisés
> - exécuter des commandes de contrôle directement dans le service concerné

### 5. Vérifier les points de contrôle essentiels

En cas de dysfonctionnement, vérifier en priorité que :

- Docker Desktop est bien démarré
- l'intégration WSL est activée si l'environnement est utilisé sous Windows avec WSL 2
- les services définis dans `compose.yaml` sont bien lancés
- les noms de services utilisés dans les fichiers de configuration correspondent bien aux noms déclarés dans Docker Compose
- les volumes montés pointent vers les bons dossiers
- le port local choisi n'est pas déjà utilisé par un autre service
- le chemin du point d'entrée web est cohérent avec l'arborescence du projet

### 6. Rebuild propre de l'environnement

Si une modification de configuration n'est pas prise en compte, arrêter puis relancer proprement l'environnement :

```bash
docker compose down
docker compose up --build
```

> Cette relance permet de repartir d'un état assaini après une modification du `Dockerfile`, du `compose.yaml` ou des fichiers de configuration liés aux services.

### 7. Vérifications minimales utiles

Vérifier que Docker répond correctement :

```bash
docker version
docker compose version
```

Vérifier l'environnement WSL si nécessaire :

```bash
wsl
```

### 8. Erreurs fréquentes à contrôler

Erreurs à vérifier en priorité :

- chemin incorrect dans une instruction `COPY` du fichier `Dockerfile`
- fichier absent du contexte de build
- port déjà utilisé sur la machine hôte
- nom de service incorrect dans la configuration Nginx
- volume monté sur un mauvais répertoire
- service démarré, mais point d'entrée web placé dans un dossier non servi par le serveur

Cette version s'appuie sur les commandes Compose officielles :

- `docker compose up` sert à construire, recréer et démarrer les services
- `docker compose ps` affiche l'état des conteneurs et les ports
- `docker compose logs` affiche les journaux et `-f` les suit en temps réel
- `docker compose exec` permet d'exécuter une commande dans un service
- `docker compose down` arrête les services et supprime les conteneurs et réseaux créés par `up`

---

## Auteur

**Rebecca Roussel**

## Sources officielles

### Symfony

- [Symfony CLI](https://symfony.com/doc/current/setup/symfony_cli.html)  
  Documentation officielle de la Symfony CLI : installation, création de projet, vérification des prérequis et usage général.

- [Download Symfony](https://symfony.com/download)  
  Page officielle de téléchargement de Symfony et de la Symfony CLI, avec les méthodes d'installation selon le système.

- [Installing & Setting up the Symfony Framework](https://symfony.com/doc/current/setup.html)  
  Documentation officielle sur les prérequis techniques, la création d'un projet Symfony et la mise en place d'un projet existant.

- [Symfony 7.4 Release](https://symfony.com/releases/7.4)  
  Page officielle de la version Symfony 7.4, utilisée pour vérifier la compatibilité avec PHP 8.2 ou supérieur.

- [Configuring a Web Server](https://symfony.com/doc/current/setup/web_server_configuration.html)  
  Documentation officielle sur la configuration du serveur web pour Symfony, notamment le rôle du répertoire `public/` et du fichier `index.php`.

- [Configuration](https://symfony.com/doc/current/configuration.html)  
  Documentation officielle sur la configuration Symfony, utile pour expliquer le rôle du dossier `config/`.

### Docker

- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/)  
  Documentation officielle d'installation de Docker Desktop sur Windows.

- [Docker Desktop WSL 2 backend on Windows](https://docs.docker.com/desktop/features/wsl/)  
  Documentation officielle sur l'utilisation de Docker Desktop avec WSL 2.

- [Use WSL](https://docs.docker.com/desktop/features/wsl/use-wsl/)  
  Documentation officielle sur le travail avec Docker Desktop dans un environnement WSL 2.

- [How Compose works](https://docs.docker.com/compose/intro/compose-application-model/)  
  Documentation officielle expliquant le fonctionnement de Docker Compose et le rôle du fichier `compose.yaml`.

- [Compose file reference](https://docs.docker.com/reference/compose-file/)  
  Référence officielle de la structure d'un fichier Compose.

- [docker compose](https://docs.docker.com/reference/cli/docker/compose/)  
  Référence officielle des commandes `docker compose`.

- [Compose Build Specification](https://docs.docker.com/reference/compose-file/build/)  
  Documentation officielle sur la section `build` dans un fichier Compose et le rôle du `Dockerfile`.

### phpDocumentor

- [Installation](https://docs.phpdoc.org/guide/getting-started/installing.html)  
  Documentation officielle sur les méthodes d'installation de phpDocumentor, notamment l'image Docker officielle et l'installation via Composer.

- [Configuration](https://docs.phpdoc.org/guide/references/configuration.html)  
  Documentation officielle sur le fichier `phpdoc.dist.xml`, son emplacement recommandé à la racine du projet et son usage comme configuration versionnée.
