# poc-mise-bootstrap

Ce dépôt est un bootstrap multi-profils géré par `mise`.

L'idée n'est plus de brancher un seul bootstrap “universel”, mais de regrouper plusieurs fichiers de configuration (profils) et de les activer avec `mise -E ...` selon les besoins du poste.

Exemple de lancement :

```bash
mise -E cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Cela charge le profil par défaut et le profil `cds` et exécute la tâche `bootstrap` de chacun d'eux.

Principe
- `config.toml` : profil de base, commun à tous.
- `config.cert.toml` : profil de certificats internes / truststore (celui là n'est normalement pas utilisé, cela devrait être géré par l'image fournie).
- `config.cds.toml` : profil de poste de travail (shell, fonts, Flatpak, certificats utilisateur).
- `-E cds` signifie “active l'environnement nommé cds, on pourrait appeler aussi cert ainsi `-E cert,cds`”.

L
Pourquoi un admin est nécessaire pour certains profils
- Les profils par défaut et `cert` contiennent des tâches qui utilisent `sudo` : installation de paquets système, modification du truststore système, configuration de Podman / subuid / subgid, etc.
- C'est normal et volontaire : certaines étapes nécessitent des droits système sur la machine.
- Les autres profils restent orientés utilisateur (`$HOME`, `~/.local`, `~/.config`, Flatpak --user, etc.).

En pratique
- Si tu as un compte standard avec un admin à côté, tu peux lancer la commande avec ton compte habituel.
- Le script demandera un mot de passe sudo uniquement pour les étapes qui le nécessitent.
- Tu ne donnes pas de sudo à l'ensemble du bootstrap, seulement aux tâches qui en ont besoin.

Les profils disponibles

1. Profil de base (`config.toml`)
   - outils de base : Java, Node, Python, Yarn, jq, etc.
   - crée le truststore Java personnalisé avec les certificats internes.
   - définit les variables d'environnement du Java truststore.

2. Profil certificat (`config.cert.toml`)
   - télécharge les certificats internes depuis la source publique de l'organisation.
   - installe les certificats dans le truststore système (`/usr/local/share/ca-certificates`)
   - exécute `update-ca-certificates` via `sudo`.

3. Profil CDS (`config.cds.toml`)
   - active `mise` et `starship` dans `.bashrc`
   - installe les polices Nerd Fonts
   - installe les applications Flatpak Firefox et Chromium
   - télécharge les certificats internes dans `~/.local/share/dinum-certs`
   - configure le trust pour Firefox et Chromium au niveau utilisateur

Commandes courantes

Installer tous les profils utiles :

```bash
mise -E cert,cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Installer seulement le profil CDS :

```bash
mise -E cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Installer seulement le profil Docker :

```bash
mise -E docker bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Mettre à jour les outils installés selon les profils actifs :

```bash
mise -E docker,cert,cds bootstrap --adopt --update 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Que faire quand un fichier de profil est modifié dans le dépôt

Le mécanisme est le suivant :
- les fichiers de profil sont dans le dépôt Git
- `mise` lit ces fichiers au moment où tu relances la commande
- les tâches de bootstrap sont idempotentes : elles peuvent être relancées sans casser l'installation

Donc, dès qu'un profil est modifié dans le dépôt :

1. Récupère les dernières modifications du dépôt.
2. Relance la commande avec le même ensemble de profils.
3. Si tu veux recharger et reconstruire les composants liés aux outils, ajoute `--update`.

Exemples :

```bash
git pull
mise -E docker,cert,cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

ou, si tu veux aussi recalculer/mettre à jour les versions gérées par `mise` :

```bash
git pull
mise -E docker,cert,cds bootstrap --adopt --update 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Autre option : si tu as déjà cloné le dépôt localement, tu peux simplement relancer `mise` depuis ce dépôt et le même profil actif pour appliquer les changements de configuration sans repartir de zéro.

Important
- `--adopt` sert à “adopter” le dépôt comme source de configuration pour la session courante.
- `-E` sélectionne les profils actifs.
- Les tâches `setup:*` sont conçues pour être relancées, ce qui les rend adaptées à la mise à jour incrémentale d'un poste.

Résumé
- Tu n'as pas besoin d'un sudo global pour tout le bootstrap.
- Tu as besoin de sudo seulement sur les tâches expressément système (docker, certificats système).
- Le bon mode de travail est “un dépôt, plusieurs fichiers de profils, selection par `-E`”.
- Pour appliquer une modification dans un profil du dépôt, il suffit de relancer la commande `mise -E ... bootstrap` avec les profils concernés.
