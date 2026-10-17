<div align="center">

<!-- ▸ Row de badges cliquables (https://shields.io) : un badge = un lien. -->
<p align="center">
  <a href="https://isocpp.org/"><img src="https://img.shields.io/badge/Language-C%2B%2B98-00599C?style=for-the-badge&logo=cplusplus&logoColor=white" alt="C++98"></a>
  <a href="https://datatracker.ietf.org/doc/html/rfc1459"><img src="https://img.shields.io/badge/Protocol-RFC_1459-000000?style=for-the-badge&logo=internetexplorer&logoColor=white" alt="RFC 1459"></a>
  <a href="https://github.com/42School/norminette"><img src="https://img.shields.io/badge/Code_Style-Norminette-brightgreen?style=for-the-badge&logo=42&logoColor=white" alt="Norminette"></a>
  <a href="https://hexchat.github.io/"><img src="https://img.shields.io/badge/Compatibility-HexChat-02A8F3?style=for-the-badge&logo=chat&logoColor=white" alt="HexChat"></a>
  <a href="https://github.com/Lamizana/FT_IRC"><img src="https://img.shields.io/badge/Projet-42-000000?style=for-the-badge&logo=42&logoColor=white" alt="Projet 42"></a>
</p>

<h1>FT_IRC</h1>

<p><strong>Serveur IRC en C++98 conforme à la RFC 1459, où plusieurs clients dialoguent en temps réel via TCP/IP.</strong></p>

<a href="https://lamizana.github.io/ZehdBox/projets/ft_irc/">
  <img src="https://img.shields.io/badge/Portfolio-ZehdBox-8A2BE2?style=for-the-badge&logo=githubpages&logoColor=white" alt="Portfolio ZehdBox">
</a>

<p>Créé le 08/12/2023 · Dernière mise à jour : 17/10/2026</p>

</div>

---

## Sommaire

<details>
<summary>Afficher le sommaire</summary>

- [Sommaire](#sommaire)
- [À propos](#à-propos)
- [Stack technique](#stack-technique)
- [Fonctionnalités](#fonctionnalités)
- [Comment ça marche](#comment-ça-marche)
- [Installation](#installation)
  - [Prérequis](#prérequis)
  - [Étapes](#étapes)
  - [Compilation](#compilation)
  - [Lancement](#lancement)
- [Bibliothèques](#bibliothèques)
- [Utilisation](#utilisation)
- [HexChat](#hexchat)
- [Structure du projet](#structure-du-projet)
- [Tests](#tests)
- [Limitations connues](#limitations-connues)
- [Contribuer](#contribuer)
- [Auteurs](#auteurs)
- [Remerciements](#remerciements)

</details>

---

## À propos

**FT_IRC** est un serveur IRC écrit en C++98 d'après la RFC 1459. Il accepte plusieurs clients simultanément sur une seule socket d'écoute et les fait dialoguer par canaux publics ou locaux, messages privés et commandes d'opérateur.

Le serveur ne dépend d'aucune bibliothèque externe, comme l'impose le sujet : l'écoute, l'acceptation des connexions et la lecture des messages reposent uniquement sur les appels système POSIX, orchestrés par `select()` pour qu'aucun client ne bloque les autres.

Le tout tient dans un seul binaire, `ircserv`, lancé avec un port et un mot de passe.

| Ressource | Lien |
| --- | --- |
| *Fiche du projet (portfolio)* | [ZehdBox](https://lamizana.github.io/ZehdBox/projets/ft_irc/) |
| *Dépôt* | [github.com/Lamizana/FT_IRC](https://github.com/Lamizana/FT_IRC) |

---

## Stack technique

| Catégorie | Technologie | Badge |
| --- | --- | --- |
| **Langage** | C++98, norme 42 | [![C++98](https://img.shields.io/badge/C%2B%2B-98-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)](https://isocpp.org/) |
| **Réseau** | Sockets TCP/IPv4, multiplexage par `select()` | [![Sockets](https://img.shields.io/badge/POSIX-sockets-4E5EE4?style=for-the-badge&logo=linux&logoColor=white)](https://pubs.opengroup.org/onlinepubs/9699919799/functions/select.html) |
| **Protocole** | IRC, RFC 1459 | [![RFC 1459](https://img.shields.io/badge/RFC-1459-000000?style=for-the-badge&logo=internetexplorer&logoColor=white)](https://datatracker.ietf.org/doc/html/rfc1459) |
| **Compilation** | `c++`, Makefile, sans relink | [![Makefile](https://img.shields.io/badge/Build-Makefile-427819?style=for-the-badge&logo=gnu&logoColor=white)](https://www.gnu.org/software/make/) |
| **Client de test** | HexChat, `nc` | [![HexChat](https://img.shields.io/badge/Client-HexChat-02A8F3?style=for-the-badge&logo=chat&logoColor=white)](https://hexchat.github.io/) |

---

## Fonctionnalités

- Authentification par mot de passe (`PASS`)
- Pseudonyme et nom d'utilisateur (`NICK`, `USER`)
- Canaux publics `#` et locaux `&` (`JOIN`, `PART`)
- Messages privés et messages de canal (`PRIVMSG`)
- Opérateurs de canaux : `KICK`, `INVITE`, `TOPIC`, `MODE`
- Modes de canaux : `+i`, `+t`, `+k`, `+o`, `+l`
- Gestion multi-clients non bloquante via `select()`
- Bonus : bot Pierre-Papier-Ciseaux (`BOT`) et compatibilité HexChat (`CAP`)

Le détail de chaque commande se trouve dans la section [Utilisation](#utilisation).

---

## Comment ça marche

Le serveur ne crée jamais de thread ni de processus : il repose entièrement sur `select()`, qui lui rend la main dès qu'un descripteur de fichier est prêt, et sur un tampon par client qui recompose les messages arrivés en plusieurs morceaux.

1. **Vérification des arguments** (`checkArgs`) : le port et le mot de passe sont validés avant tout appel réseau.
2. **Création du serveur** : la socket est créée, attachée au port, mise en écoute, puis inscrite dans un `fd_set` par `FD_SET`.
3. **Boucle principale** : à chaque tour, `select()` surveille l'ensemble des descripteurs. Le programme ne bloque jamais sur un client en particulier.
4. **Nouvelle connexion** : `accept()` crée la socket du client, qui rejoint la liste et le `fd_set`.
5. **Données reçues** : `recv()` remplit le tampon. Si la ligne n'est pas terminée par `\n`, le fragment est accumulé pour ce client. Sinon, la commande complète est découpée, puis aiguillée vers son gestionnaire par `Server::command`.
6. **Déconnexion** : `recv()` renvoie 0, ou le client envoie `QUIT`, alors la socket est fermée, retirée du `fd_set` par `FD_CLR` et le client est supprimé de la liste.
7. **Arrêt** : `SIGINT` n'est pas ignoré. Il déclenche une exception qui déroule la pile, libère les ressources et ferme la socket d'écoute.

```console
        socket() ──► bind() ──► listen()
            │
            └──► boucle select()
                    ├── descripteur prêt        ──► accept()  ──► FD_SET
                    ├── données reçues          ──► recv()
                    │        ├── ligne incomplète ──► bufTmp(PUSH)   (tampon par client)
                    │        └── ligne complète   ──► découpage ──► Server::command() ──► réponse
                    └── connexion fermée        ──► close() ──► FD_CLR ──► removeClient()
```

> [!NOTE]
> Le tampon par client est ce qui permet de recevoir une commande découpée en plusieurs paquets réseau. C'est ce que vérifie le test « réception de données partielles » plus bas, où `com`, puis `man`, puis `d` finissent par former `command`.

---

## Installation

### Prérequis

- Système Unix/Linux (macOS pris en charge)
- Compilateur `c++` compatible norme C++98
- `make`
- `nc` ou un client IRC (HexChat, irssi, WeeChat...) pour se connecter

### Étapes

```bash
git clone https://github.com/Lamizana/FT_IRC.git
cd FT_IRC
make
```

> [!NOTE]
> Variante SSH : `git clone git@github.com:Lamizana/FT_IRC.git`

### Compilation

```bash
make        # règle all
```

- Le Makefile fournit les règles `all`, `clean`, `fclean` et `re`.
- Compilation avec `c++` et les drapeaux `-Wall -Wextra -Werror -std=c++98`.
- Le Makefile ne relink pas.

```bash
make clean      # supprime les fichiers objets (.o)
make fclean     # supprime les objets + l'exécutable
make re         # fclean + all
```

### Lancement

```bash
./ircserv <port> <password>
```

- **port** : numéro de port sur lequel le serveur IRC écoutera les connexions entrantes.
- **password** : mot de passe de connexion requis pour tout client.

```bash
./ircserv 6667 toto
```

---

## Bibliothèques

Aucune bibliothèque externe : le sujet les interdit, Boost compris. Le projet n'utilise que la bibliothèque standard C++98 et les en-têtes système POSIX.

| En-tête | Rôle |
| --- | --- |
| [`<sys/socket.h>`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/sys_socket.h.html) | `socket`, `bind`, `listen`, `accept`, `recv`, `send` |
| [`<sys/select.h>`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/sys_select.h.html) | `select` et `fd_set` : le multiplexage des descripteurs |
| [`<netinet/in.h>`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/netinet_in.h.html) | Structures d'adresses IPv4 (`sockaddr_in`) |
| [`<arpa/inet.h>`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/arpa_inet.h.html) | Conversions d'adresses |
| [`<unistd.h>`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/unistd.h.html) | `read`, `write`, `close` |
| [`<signal.h>`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/signal.h.html) | Interception de `SIGINT` pour l'arrêt propre |

---

## Utilisation

### Connexion avec nc

```bash
nc 127.0.0.1 6667
PASS toto
NICK toto
USER toto
JOIN #general
```

### Connexion avec un client IRC (HexChat, irssi, WeeChat...)

- Adresse : `127.0.0.1`
- Port    : `6667`
- Mot de passe : `toto`

### Commandes implémentées

| Commande | Description |
| --- | --- |
| CAP | Négociation de capacités (compatibilité client) |
| PASS | Authentification (mot de passe du serveur) |
| NICK | Définir / changer de pseudonyme |
| USER | Définir le nom d'utilisateur (inchangeable) |
| PRIVMSG | Message privé à un utilisateur ou à un canal |
| JOIN | Rejoindre ou créer un canal (#canal) |
| KICK | Éjecter un client d'un canal (opérateur) |
| INVITE | Inviter un client à un canal |
| TOPIC | Modifier ou afficher le thème d'un canal |
| MODE | Changer les modes du canal |
| PART | Quitter un canal |
| BOT | Bonus : jouer à Pierre-Papier-Ciseaux |

### Modes de canal (MODE)

| Mode | Description |
| --- | --- |
| +i / -i | Canal sur invitation uniquement |
| +t / -t | Restreindre TOPIC aux opérateurs |
| +k / -k | Définir / supprimer le mot de passe du canal |
| +o / -o | Donner / retirer le statut d'opérateur |
| +l / -l | Définir / supprimer la limite d'utilisateurs |

### Exécution des commandes

> [!IMPORTANT]
> - Chaque commande se tape sur **UNE SEULE ligne**, directement dans la connexion (nc ou client IRC), **sans slash** (le `/` est un raccourci propre aux clients graphiques, pas au protocole brut).
> - Les commandes sont **sensibles à la casse** : utilisez uniquement des **MAJUSCULES** (`PASS`, `NICK`, `JOIN`...).
> - Le mot de passe s'envoie sur la **même ligne** que `PASS` : `PASS toto` (jamais `PASS` sur une ligne puis `toto` sur la suivante).
> - Ordre obligatoire : `PASS` → `NICK` → `USER`. Le serveur répond alors `: 001 <pseudo> :Welcome to the IRC Network...`.
> - Les autres commandes (`JOIN`, `PRIVMSG`, `MODE`...) ne fonctionnent qu'**après** ce message de bienvenue.

| Ce que vous tapez | Effet |
| --- | --- |
| `PASS toto` | Authentification (doit correspondre au mot de passe de lancement) |
| `NICK alice` | Définit le pseudonyme (unique, max 9 caractères) |
| `USER alice` | Définit le nom d'utilisateur (inchangeable ensuite) |
| `JOIN #general` | Crée ou rejoint le canal `#general` |
| `JOIN #securise mdp` | Rejoint un canal protégé par mot de passe |
| `PRIVMSG #general :salut !` | Diffuse un message à tout le canal |
| `PRIVMSG bob :coucou` | Envoie un message privé à `bob` |
| `TOPIC #general :Bienvenue` | Modifie le sujet du canal |
| `TOPIC #general` | Affiche le sujet du canal |
| `INVITE bob #general` | Invite `bob` dans le canal |
| `KICK #general bob :troll` | Éjecte `bob` (opérateur requis) |
| `MODE #general +t` | Active un mode du canal (`i`, `t`, `k`, `o`, `l`) |
| `PART #general :a+` | Quitte le canal |
| `BOT PAPER` | Bonus : joue à Pierre-Papier-Ciseaux avec le BOT |

### Exemple de session

```bash
./ircserv 6667 toto     # Terminal 1 : lancement du serveur

# Terminal 2 : connexion au serveur
nc 127.0.0.1 6667
PASS toto
NICK alice
USER alice
JOIN #general
TOPIC #general :Bienvenue
INVITE bob #general
PRIVMSG #general :salut tout le monde !
PART #general
```

---

## HexChat

### Installation (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install hexchat
```

### Lancement

```bash
hexchat
```

- Fenêtre « Réseaux » : bouton **+ Add** pour créer un réseau.
- Onglet **Connexion** : cocher « Utiliser un mot de passe » et saisir `toto`.
- Onglet **Utilisateur** : définir le pseudo et le nom d'utilisateur.
- Dans « Serveurs », ajouter `127.0.0.1/6667` puis cliquer sur **Connecter**.

### Ou en commandes internes à HexChat

```bash
/server 127.0.0.1 6667 toto
/join #general
```

### Exemple de session

```bash
/server 127.0.0.1 6667 toto
/join #general
/topic #general Bienvenue
/msg #general salut tout le monde !
/msg alice coucou en privé
/invite alice #general
/kick #general alice troll    (opérateur requis)
/part #general
```

---

## Structure du projet

```console
FT_IRC/
├── Makefile              # règles all, clean, fclean, re
├── main.cpp              # boucle serveur (select) et gestion des connexions
├── define.hpp            # numerics et messages IRC
├── library.hpp           # bibliothèques et includes communs
├── utils.cpp / utils.hpp # helpers (trim, validation des arguments)
├── Channel/
│   └── Channel.cpp / Channel.hpp    # canal : utilisateurs, opérateurs, modes
├── Client/
│   ├── Client.cpp / Client.hpp      # client : socket, buffer, enregistrement
│   ├── bot.cpp, invite.cpp, join.cpp, kick.cpp   # handlers des commandes
│   ├── mode.cpp, nick.cpp, part.cpp, pass.cpp
│   └── prvmsg.cpp, topic.cpp, user.cpp
└── Server/
    ├── Server.cpp / Server.hpp      # logique serveur et dispatch
    ├── ServerDebug.cpp              # affichage de débogage (<<)
    ├── ServerMode.cpp               # implémentation des modes
    ├── ServerSwitch.cpp             # aiguillage des commandes
    └── allClient.cpp                # réponses NAMES/ENDOFNAMES
```

---

## Tests

### Connexion multiple

Ouvrir plusieurs terminaux et se connecter simultanément (`nc 127.0.0.1 <port>`) pour vérifier la gestion multi-clients.

### Réception de données partielles

Envoyer une commande en plusieurs fragments avec nc et Ctrl+D :

```bash
$> nc 127.0.0.1 6667
com^Dman^Dd
$>
```

`com`, puis `man`, puis `d` + Entrée reconstruisent la commande transmise au serveur.

### Commandes opérateurs

Créer un canal, donner `+o` à un utilisateur, puis tester `KICK`, `TOPIC` et `MODE`.

### Valgrind

```bash
valgrind --track-origins=yes --leak-check=full --track-fds=yes ./ircserv <port> <password>
```

---

## Limitations connues

- **PING/PONG non implémenté** : les clients IRC se déconnectent après ~60 s d'inactivité (« Ping timeout » dans HexChat).

---

## Contribuer

Projet pédagogique 42 : les corrections et les remarques sont bienvenues via une **issue** ou une **pull request**.

Les messages de commit suivent [Conventional Commits](https://www.conventionalcommits.org/fr/) :

```text
<type>(<scope>): <description>
```

| Type | Usage |
| --- | --- |
| `feat` | Nouvelle fonctionnalité |
| `fix` | Correction de bug |
| `docs` | Documentation seule |
| `refactor` | Réécriture sans changement de comportement |
| `chore` | Maintenance (configuration, fichiers annexes) |

**Scopes** : `server`, `client`, `channel`, `parsing`, `readme`.

---

## Auteurs

**Alex Lamizana**, étudiant à 42 Angoulême, spécialisation Data & IA

- [Site](https://lamizana.github.io/ZehdBox/)
- [GitHub](https://github.com/Lamizana)
- [Contact](https://lamizana.github.io/ZehdBox/contact/)

**mvitiell**, **nminotte** : coéquipiers sur le projet.

---

## Remerciements

- **42 Angoulême** : cadre pédagogique du projet.
- **mvitiell** et **nminotte** : conception et développement du serveur.

---

<div align="center">

*Ce projet a été réalisé conformément à la norme de 42.*

</div>
