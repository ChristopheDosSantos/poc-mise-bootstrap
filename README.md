# poc-mise-bootstrap

Bootstrap personnel sans sudo pour la branche `dev/cds`.

Ce dépôt installe les outils de développement dans le profil utilisateur (`$HOME`) pour éviter les droits administrateur et les dépendances au groupe `docker`.

Commandes pour lancer le bootstrap :

```bash
mise bootstrap --from 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git?ref=dev/cds'
```

Pour relancer et mettre à jour le bootstrap :

```bash
mise bootstrap --update --from 'https://github.com/ChristopheDosSantos/poc-mise-bootstrap.git?ref=dev/cds'
```

Ce bootstrap installe automatiquement :
- Rust via `mise`
- Go via `mise`
- Typst via `mise`
- Starship
- JetBrains Mono Nerd Font
- JetBrains Toolbox
- le fichier de configuration Starship depuis `config/starship.toml`

Le fichier Starship est versionné dans le dépôt et copié vers `~/.config/starship.toml` au moment du bootstrap.

