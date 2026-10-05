# SAE5.02
Projet SAE5.02

Projet réalisé dans le cadre de la SAÉ 5.02 — Piloter un projet informatique

## 1. Présentation du projet

Le projet consiste à mettre en place une plateforme multimédia conteneurisée et automatisée permettant de gérer et consulter des films, séries, mangas, bandes dessinées et comics.

L'infrastructure repose sur Docker pour l'exécution des différents services et sur Ansible pour automatiser l'installation, la configuration et le déploiement de la plateforme.

L'objectif est de pouvoir déployer l'ensemble de la plateforme à partir des playbooks Ansible.

## 2. Services utilisés

La plateforme est composée de 7 services :

* Prowlarr : gestion des indexeurs et communication avec Sonarr et Radarr.
* Sonarr : gestion et organisation des séries.
* Radarr : gestion et organisation des films.
* qBittorrent : gestion des téléchargements.
* Jellyfin : consultation des films et séries.
* Komga : consultation des mangas, BD et comics.
* Bazarr : gestion automatique des sous-titres pour Sonarr et Radarr.

## 3. Architecture

Les services sont exécutés dans des conteneurs Docker et communiquent principalement à travers un réseau Docker dédié nommé `media_network`.

L'infrastructure est déployée sur une machine virtuelle servant d'environnement d'exécution.

### Arborescence des données

```text
/srv/mediastack/
├── data/
│   ├── dl/
│   ├── film/
│   ├── serie/
│   └── livre/
│
└── config/
    ├── prowlarr/
    ├── sonarr/
    ├── radarr/
    ├── qbittorrent/
    ├── jellyfin/
    ├── komga/
    └── bazarr/
```

La séparation entre `data` et `config` permet de distinguer les fichiers multimédias des fichiers de configuration des applications.

## 4. Communication entre les services

Les différents services utilisent le réseau Docker `media_network`.

Les principales communications sont les suivantes :

```text
Prowlarr
   ├──> Sonarr
   └──> Radarr

Sonarr ───> qBittorrent
Radarr ───> qBittorrent

Bazarr ───> Sonarr
Bazarr ───> Radarr

Jellyfin ───> data/film
Jellyfin ───> data/serie

Komga ───> data/livre
```

Prowlarr centralise la gestion des indexeurs et transmet les informations nécessaires à Sonarr et Radarr.

Sonarr et Radarr utilisent qBittorrent pour les téléchargements. Les fichiers téléchargés sont ensuite organisés dans les répertoires correspondants.

Jellyfin utilise les répertoires contenant les films et les séries afin de permettre leur consultation.

Komga utilise le répertoire `livre` pour les mangas, bandes dessinées et comics.

Bazarr communique avec Sonarr et Radarr afin de gérer les sous-titres.

## 5. Déploiement avec Ansible

Le déploiement est organisé autour de plusieurs rôles Ansible.

### Rôles prévus

```text
roles/
├── docker/
├── storage/
├── mediastack/
├── security/
└── verify/
```

### Description des rôles

`docker`

Installation et configuration de Docker et Docker Compose.

`storage`

Création de l'arborescence `/srv/mediastack`, des différents répertoires nécessaires et configuration des permissions.

`mediastack`

Création du réseau Docker `media_network` et déploiement des 7 services avec Docker Compose.

`security`

Configuration du pare-feu et mise en place des règles permettant de limiter les accès aux services nécessaires.

`verify`

Vérification automatique du bon fonctionnement de l'installation et du déploiement.

## 6. Playbooks

Les playbooks prévus sont :

```text
playbooks/
├── docker.yml
├── storage.yml
├── mediastack.yml
├── security.yml
├── verify.yml
└── site.yml
```

Le playbook `site.yml` permet d'orchestrer l'ensemble des étapes dans l'ordre :

```text
Docker
  ↓
Storage
  ↓
MediaStack
  ↓
Security
  ↓
Verify
```

L'objectif est de pouvoir lancer le déploiement complet depuis le playbook principal.

## 7. Réseau et ports

Les conteneurs communiquent entre eux grâce au réseau Docker privé `media_network`.

Les ports exposés sur la machine hôte sont volontairement limités.

Ports accessibles :

| Service  |  Port | Utilisation               |
| -------- | ----: | ------------------------- |
| Jellyfin |  8096 | Interface de consultation |
| Komga    | 25600 | Interface de consultation |

Les autres services ne sont pas exposés inutilement et sont accessibles uniquement lorsque cela est nécessaire à leur administration ou à leur fonctionnement.

## 8. Sécurité

Plusieurs mesures de sécurité sont prévues :

* limitation des ports exposés ;
* configuration d'un pare-feu ;
* utilisation d'un réseau Docker dédié ;
* gestion des permissions sur `/srv/mediastack` ;
* séparation des données et des configurations ;
* utilisation d'Ansible Vault pour les informations sensibles ;
* aucun mot de passe ou secret stocké en clair dans le dépôt Git ;
* utilisation d'un `.gitignore` pour éviter l'envoi accidentel de fichiers sensibles.

La plateforme est destinée à fonctionner sur le réseau local de la machine virtuelle.

Aucun accès depuis Internet n'est prévu dans le périmètre actuel du projet.

La mise en place du HTTPS et d'un reverse proxy ne fait pas partie du périmètre actuel. Elle pourra éventuellement être envisagée comme amélioration future.

## 9. Tests et vérifications

Un rôle `verify` est prévu afin de contrôler automatiquement le résultat du déploiement.

Les vérifications porteront notamment sur :

* l'état des conteneurs avec `docker ps` ;
* l'existence du réseau `media_network` ;
* l'accessibilité des services avec `curl` ;
* l'existence des répertoires avec `stat` ;
* les permissions des répertoires ;
* la présence des fichiers nécessaires ;
* le bon démarrage des services ;.

Les playbooks seront exécutés plusieurs fois afin de vérifier qu'une seconde exécution ne provoque pas de modifications inutiles.

## 10. Automatisation

L'objectif principal du projet est de rendre le déploiement entièrement automatisable.

Une nouvelle installation doit pouvoir être réalisée en exécutant le playbook principal :

```bash
ansible-playbook -i inventory site.yml
```


## 11. Gestion du projet

Le projet est versionné avec Github

Le dépôt contient notamment :

```text
.
├── inventory/
├── playbooks/
├── roles/
├── docker-compose.yml
├── README.md
└── .gitignore
```

Les commits seront réalisés régulièrement afin de suivre l'avancement du projet.

Aucun secret ne doit être présent dans le dépôt.

## 12. Estimation du temps

L'estimation globale du développement est d'environ 30 heures.

| Tâche                                       | Temps estimé |
| ------------------------------------------- | -----------: |
| Installation et configuration de Docker     |          3 h |
| Arborescence et permissions                 |          3 h |
| Création des rôles Ansible                  |         10 h |
| Déploiement et configuration des 7 services |          6 h |
| Sécurité et pare-feu                        |          2 h |
| Tests et vérifications                      |          3 h |
| Documentation                               |          3 h |
| **Total**                                   |     **30 h** |

## 13. Objectif final

L'objectif final est d'obtenir une plateforme multimédia fonctionnelle, conteneurisée et automatisée.

Le système devra pouvoir être installé et configuré à partir des playbooks Ansible, avec une séparation claire entre les données, les configurations et les services.

La démonstration devra permettre de montrer le déploiement, le fonctionnement des différents conteneurs, les communications entre les services, les mesures de sécurité ainsi que les tests de vérification.

## 14. Améliorations possibles

Les évolutions suivantes pourront être envisagées après la réalisation du périmètre principal :

* mise en place d'un reverse proxy ;
* ajout du HTTPS avec certificats SSL ;
* amélioration des tests automatisés ;
* ajout de sauvegardes automatisées ;
* supervision des conteneurs ;
* amélioration de la documentation technique.
