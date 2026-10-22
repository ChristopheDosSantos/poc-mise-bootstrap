# poc-mise-bootstrap

Ce dépôt est un bootstrap multi-profils géré par `mise`.

L'idée n'est plus de brancher un seul bootstrap “universel”, mais de regrouper plusieurs fichiers de configuration (profils) et de les activer avec `mise -E ...` selon les besoins du poste.

Exemple de lancement :

```bash
mise -E cds,virtualBox bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Cela charge le profil de base, le profil `cds`, le profil `virtualBox` et exécute la tâche `bootstrap` de chacun d'eux.

Principe
- `config.toml` : profil de base, commun à tous.
- `config.cert.toml` : profil réservé aux environnements virtualisés (VMs / images de travail guest). Il ne doit pas être installé sur une machine physique de travail.
- `config.cds.toml` : profil de poste de travail (shell, fonts, Flatpak, certificats utilisateur).
- `config.virtualBox.toml` : profil d'installation de VirtualBox et Vagrant sur le poste physique qui doit exécuter des machines virtuelles.
- `-E cds,virtualBox` signifie “active les profils `cds` et `virtualBox`”.

Le profil Docker n'est plus isolé dans un fichier distinct : il a été intégré dans le profil de base (`config.toml`), car c'est un composant de base de l'environnement de dev sur la plupart des machines.

Pourquoi un admin est nécessaire pour certains profils
- Les profils de base et `cert` contiennent des tâches qui utilisent `sudo` : installation de paquets système, modification du truststore système, configuration de Podman / subuid / subgid, etc.
- C'est normal et volontaire : certaines étapes nécessitent des droits système sur la machine.
- Les profils orientés utilisateur (`cds`, `virtualBox` selon le cas) restent majoritairement dans `$HOME`, `~/.local`, `~/.config`, ou Flatpak --user.

En pratique
- Si tu as un compte standard avec un admin à côté, tu peux lancer la commande avec ton compte habituel.
- Avant le bootstrap, l'admin doit donner temporairement les droits sudo au compte concerné :

```bash
sudo usermod -aG sudo <username>
```

- Ensuite le compte peut lancer la commande et le `sudo` demandé sera limité aux tâches qui le nécessitent.
- Une fois la configuration terminée, l'admin peut retirer les droits sudo du compte pour revenir à un usage standard :

```bash
sudo gpasswd -d <username> sudo
```

- Tu ne donnes pas de sudo à l'ensemble du bootstrap, seulement aux tâches qui en ont besoin.

Règle claire sur le profil `cert`
- `cert` est réservé aux VMs, machines virtuelles ou images de travail guest.
- Il ne doit pas être utilisé sur un poste physique de travail standard.
- L'objectif est d'éviter de modifier le truststore système d'un vrai poste de travail en production, alors qu'on veut juste préparer un environnement virtuel cohérent.
- En pratique : sur le host physique, on fait `cds` + éventuellement `virtualBox`; dans la VM, on peut ajouter `cert` si nécessaire.

Les profils disponibles

1. Profil de base (`config.toml`)
   - outils de base : Java, Node, Python, Yarn, jq, etc.
   - crée le truststore Java personnalisé avec les certificats internes.
   - définit les variables d'environnement du Java truststore.
   - installe Podman / podman-docker
   - configure les subuid / subgid et le linger du compte
   - prépare le `containers.conf` utilisateur

2. Profil certificat (`config.cert.toml`)
   - télécharge les certificats internes depuis la source publique de l'organisation.
   - installe les certificats dans le truststore système (`/usr/local/share/ca-certificates`)
   - exécute `update-ca-certificates` via `sudo`.
   - doit être utilisé uniquement dans des VMs ou des environnements virtualisés.

3. Profil CDS (`config.cds.toml`)
   - active `mise` et `starship` dans `.bashrc`
   - installe les polices Nerd Fonts
   - installe les applications Flatpak Firefox et Chromium
   - télécharge les certificats internes dans `~/.local/share/dinum-certs`
   - configure le trust pour Firefox et Chromium au niveau utilisateur

4. Profil VirtualBox (`config.virtualBox.toml`)
   - installe `virtualbox`, `virtualbox-ext-pack`, `vagrant` et `dkms` via APT
   - ajoute l'utilisateur au groupe `vboxusers`
   - vérifie que `VBoxManage` et `vagrant` sont disponibles
   - sert à utiliser des machines virtuelles sur le poste local

VirtualBox est donc bien installable via `mise`, mais il s'agit d'un package système intégralement géré via les profils APT, pas d'un outil de runtime géré directement par `mise install`.

Commandes courantes

Installer le poste standard de travail :

```bash
mise -E cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Installer le poste avec VirtualBox :

```bash
mise -E cds,virtualBox bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Installer un environnement virtuel / VM avec les certificats système :

```bash
mise -E cert,cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Installer seulement le profil VirtualBox :

```bash
mise -E virtualBox bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Mettre à jour les outils installés selon les profils actifs :

```bash
mise -E cds,virtualBox bootstrap --adopt --update 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

ou, pour une VM :

```bash
mise -E cert,cds bootstrap --adopt --update 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Cookbook

1. Poste de dev standard
   - L'admin ajoute temporairement le compte au groupe sudo :

```bash
sudo usermod -aG sudo <username>
```

   - Le user lance :

```bash
mise -E cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

   - Une fois le bootstrap terminé, l'admin retire les droits sudo :

```bash
sudo gpasswd -d <username> sudo
```

2. Poste de dev avec VirtualBox
   - L'admin donne temporairement le droit sudo au compte :

```bash
sudo usermod -aG sudo <username>
```

   - Le user lance :

```bash
mise -E cds,virtualBox bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

   - Une fois le poste prêt, l'admin retire les droits sudo :

```bash
sudo gpasswd -d <username> sudo
```

3. VM / environnement virtuel avec certificats système
   - L'admin attribue temporairement sudo au compte de la VM :

```bash
sudo usermod -aG sudo <username>
```

   - Le user lance :

```bash
mise -E cert,cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

   - Lorsque la VM est stabilisée, l'admin supprime le sudo du compte :

```bash
sudo gpasswd -d <username> sudo
```

Important : le fait de donner temporairement sudo n'est pas un mode de fonctionnement normal du bootstrap ; c'est seulement un mécanisme de préparation pour exécuter les étapes système demandées par certains profils.

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
mise -E cds,virtualBox bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

ou, pour une VM :

```bash
git pull
mise -E cert,cds bootstrap --adopt 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

ou, si tu veux aussi recalculer/mettre à jour les versions gérées par `mise` :

```bash
git pull
mise -E cds,virtualBox bootstrap --adopt --update 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git'
```

Autre option : si tu as déjà cloné le dépôt localement, tu peux simplement relancer `mise` depuis ce dépôt et le même profil actif pour appliquer les changements de configuration sans repartir de zéro.

Important
- `--adopt` sert à “adopter” le dépôt comme source de configuration pour la session courante.
- `-E` sélectionne les profils actifs.
- Les tâches `setup:*` sont conçues pour être relancées, ce qui les rend adaptées à la mise à jour incrémentale d'un poste.

Résumé
- Le profil de base intègre désormais Docker/Podman.
- `cert` est réservé aux VMs / environnements virtualisés, pas aux machines physiques de travail.
- Sur un poste physique, on utilise plutôt `cds` + éventuellement `virtualBox`.
- Pour appliquer une modification dans un profil du dépôt, il suffit de relancer la commande `mise -E ... bootstrap` avec les profils concernés.
