# Les métiers de la communication et des réseaux sociaux

[Voir le site](https://williamfournialuca.github.io/metierwilliam/)

Veille France Travail pour préparer le projet métier et les séances de cours.
Le site présente les offres actives, leur localisation, les contrats, les salaires,
les compétences, les recruteurs et l’évolution des volumes.

## Métiers suivis

| Métier | Code ROME |
|---|---|
| Social media manager | E1124 |
| Community manager | E1101 |
| Chargé(e) des relations avec les influenceurs | M1719 |
| Influenceur(se) web | E1406 |
| Chargé(e) de communication | E1112 |
| Chargé(e) des relations publiques | E1103 |
| Chef(fe) de projet événementiel | E1107 |
| Assistant(e) en publicité | E1404 |

Les huit métiers sont cochés par défaut et peuvent être comparés ou filtrés.
La sélection est conservée entre les cinq pages. Les anciens filtres marketing
sont réinitialisés lors du passage à ce périmètre.

## Données et actualisation

`scripts/extraire.py` définit les métiers dans `METIERS` et interroge l’API
France Travail pour chaque code. Le workflow `.github/workflows/veille.yml`
lance la collecte chaque jour à 5 h UTC (7 h à Paris en été, 6 h en hiver).

- `data/brut/` conserve chaque version d’annonce.
- `data/actives/` conserve les identifiants actifs à chaque extraction.
- `data/serie.csv` conserve les volumes quotidiens par métier.
- `scripts/resumer.py` produit `data/resume.json` pour les cinq pages du site.

Les archives des anciens métiers sont conservées ; le résumé, ses compteurs
et ses séries historiques ne prennent en compte que les huit codes ci-dessus.
Le classement dépend des codes ROME de France Travail : une annonce peut être
mal classée ou apparaître plusieurs fois. Ces chiffres décrivent ce canal,
pas l’ensemble du marché. Les limites figurent aussi sur la page « Le marché bouge ».

## Exécution locale

```sh
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# Renseigner FT_CLIENT_ID et FT_CLIENT_SECRET dans .env (voir .env.example).
python scripts/extraire.py
python scripts/resumer.py
python -m http.server 8125
```

Pour recalculer le site à partir des archives existantes, seule la commande
`python scripts/resumer.py` est nécessaire. Les identifiants restent dans `.env`
ou les secrets GitHub Actions et ne doivent jamais être versionnés.
