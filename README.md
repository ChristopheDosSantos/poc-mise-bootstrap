# poc-mise-bootstrap
test de préparation de machine avec mise bootstrap

Commande pour lancer le bootstrap :

```bash
mise bootstrap --prompt-vars --from 'https://ChristopheDosSantos/poc-mise-bootstrap.git?ref=developpeur'
```

Si on relance la commande, ne pas oublier de faire ceci :


```bash
mise bootstrap --update --prompt-vars --from 'https://ChristopheDosSantos/poc-mise-bootstrap.git?ref=developpeur'
```

Comme les variables sont enregistrées dans le fichier `~/.mise/vars`, il est possible de ne pas utiliser l'option `--prompt-vars` si on veut relancer le bootstrap avec les mêmes valeurs.
Mais si on veut changer une valeur, il faut supprimer la variable dans le fichier `~/.mise/vars` avant de relancer le bootstrap.
On peut également effacer une variable avec la commande `mise vars unset dev_user`.

