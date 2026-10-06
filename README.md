# Le marché du social media manager

### 👉 **[Voir le site](https://williamfournialuca.github.io/metierwilliam/)**

Le site est mis à jour chaque matin par une Action GitHub : elle interroge
l'API France Travail, enregistre les offres du jour et publie les chiffres du
métier de social media manager.

| | |
|---|---|
| [Accueil](https://williamfournialuca.github.io/metierwilliam/) | les chiffres-clés, la carte de France et les filtres |
| [Ce que ça paie](https://williamfournialuca.github.io/metierwilliam/salaires.html) | salaires par niveau, contrat et territoire |
| [Ce qu'on vous demande](https://williamfournialuca.github.io/metierwilliam/exigences.html) | expérience, diplôme, outils et compétences |
| [Qui recrute](https://williamfournialuca.github.io/metierwilliam/recruteurs.html) | entreprises, secteurs et offres récentes |
| [Le marché bouge](https://williamfournialuca.github.io/metierwilliam/mouvement.html) | évolution du nombre d'offres et fraîcheur des annonces |

Dossier de travail pour la séance « Écouter le marché de votre métier »
(M2 MOD, IAE Clermont Auvergne) : API France Travail → données → Action
planifiée → site GitHub Pages.

## Le métier suivi

- **Métier** : social media manager
- **Code ROME suivi** : **E1124**
- L'extraction, les filtres et les graphiques portent uniquement sur ce code.

## Les questions étudiées

1. Combien d'offres sont publiées, et où se situent-elles ?
2. Quels contrats et quels salaires sont proposés ?
3. Quelles expériences, formations, compétences et quels outils sont demandés ?
4. Quelles entreprises et quels secteurs recrutent ?

## La chaîne

```
API France Travail → scripts/extraire.py → data/brut/<mois>/E1124.jsonl
                                        → data/actives/<date>.csv
                                        → data/serie.csv
                    scripts/resumer.py → data/resume.json → les cinq pages HTML
                    .github/workflows/veille.yml → collecte puis publication GitHub Pages
```

- `scripts/extraire.py` — une requête `codeROME=E1124` (token OAuth,
  pagination 150 / 1 150, total lu dans `Content-Range`). Le **brut est
  conservé intégralement** : une offre est écrite la première fois qu'on la
  voit, et de nouveau si son contenu change (empreinte SHA-1 du JSON, hors
  `dateActualisation`) — l'évolution d'une annonce est donc gardée, version
  par version. Relancer le même jour n'écrit rien deux fois.
- `scripts/resumer.py` — retravaille le brut des offres actives : salaires
  (libellé texte → min/max annuels bruts), outils cités dans les descriptions
  (grille à adapter), position (lat/lon de l'API, sinon centre de la commune
  via geo.api.gouv.fr, sinon ville principale du département).
- Cinq pages HTML statiques, un chantier par page, toutes servies telles quelles.
  Chacune charge `data/resume.json` et recalcule ses graphiques Chart.js dans le
  navigateur selon les filtres ; net mensuel estimé = brut × 0,78 / 12.
  - `index.html` — les filtres, les chiffres-clés, la carte Leaflet (survol =
    l'offre, clic = l'annonce sur France Travail), les départements, les
    contrats, et les liens vers les quatre autres pages.
  - `salaires.html` — ce que ça paie. `exigences.html` — ce qu'on vous demande.
    `recruteurs.html` — qui recrute. `mouvement.html` — le marché bouge, et les
    limites de ces chiffres (ancre `#limites`, liée depuis chaque pied de page).
- `assets/commun.js` et `assets/commun.css` — ce que les cinq pages partagent :
  chargement des données, panneau de filtres sur le contrat et le niveau de
  poste, barre de navigation, utilitaires et fabriques de graphiques.
  Une page ne contient que son HTML et son petit script `rendre(offres, D)`.

Les extractions historiques d'autres codes ROME restent archivées dans le dépôt,
mais ne sont plus utilisées dans le résumé publié ni affichées sur le site.

## Volume et limites GitHub

Le brut du métier suivi est conservé version par version. GitHub gratuit : dépôt
1 Go recommandé, fichier ≤ 100 Mo, Pages 1 Go publié et 100 Go/mois de bande
passante, Actions illimitées sur un dépôt public. Quand le brut dépassera
quelques centaines de Mo, l'Action archivera chaque mois écoulé (compressé)
dans les Releases du dépôt ou sur Hugging Face Datasets, et le dépôt ne
gardera que les derniers mois.

## Faire tourner chez soi

```
py -3.12 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
copy .env.example .env        (puis remplir avec ses identifiants francetravail.io)
.venv\Scripts\python.exe scripts\extraire.py --verifier
.venv\Scripts\python.exe scripts\extraire.py
.venv\Scripts\python.exe scripts\resumer.py
.venv\Scripts\python.exe -m http.server 8125      (puis http://localhost:8125)
```

## Faire tourner sans soi (GitHub)

1. Dépôt **public** (GitHub Pages gratuit ne fonctionne que sur un dépôt public).
2. Settings → Secrets and variables → Actions : `FT_CLIENT_ID` et `FT_CLIENT_SECRET`.
3. Actions → veille → Run workflow : l'Action collecte les offres, met à jour
   `data/resume.json` et publie les cinq pages sur GitHub Pages.
4. Le site est ensuite redéployé automatiquement à chaque exécution planifiée
   ou manuelle du workflow. Seules les pages, leurs fichiers partagés et le
   résumé du social media manager sont publiés.

## Règles

- Les identifiants sont dans `.env` (local) ou dans les secrets du dépôt
  (GitHub) : jamais dans un fichier versionné.
- Un canal, une requête, une date : chaque chiffre du site les affiche.
- Pas de scraping de LinkedIn, APEC ou Indeed (interdit par leurs CGU).
