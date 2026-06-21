# Dojo AI PM — Séance 2 : Faire des specs

Le **dojo** est un parcours de formation pour les **Product Managers** : apprendre à utiliser l'IA dans
les gestes du métier. Il **s'appuie sur le keiko** (le tuto de modernisation) mais en est un item
distinct.

Ce dépôt contient la **Séance 2 — « Faire des specs »** : produire une spec fonctionnelle de qualité,
en la **durcissant** (grill-me) puis en la **faisant vérifier** par un agent indépendant.

## Contenu

- `fiche-trainee.md` — consignes apprenant (le parcours en 3 temps).
- `fiche-trainer.md` — déroulé animateur (minutage, pièges, tips).
- `exemple-spec-reference.md` — une spec modèle (pour l'animateur).
- `.claude/skills/spec-grill-session/` — le skill de la séance (grill-me → spec → vérif).
- `docs/features/entries-list/` — le dossier pré-mâché de la page exemple : `analysis.md` (analyse
  technique) + `browser/` (capture d'écran + snapshot).
- `analysis/entry-points.md` — par où on arrive sur chaque page (points d'entrée).
- `legacy/` — l'appli à spécifier (Ketchup Compta), en submodule.

## Prérequis

- Avoir fait la S0 du keiko : Ketchup Compta tourne en local (`http://localhost:8080`, `admin`/`admin123`),
  `claude` répond.
- Cloner ce dépôt avec son submodule :

```bash
git clone --recurse-submodules https://github.com/Matigon-theodo/dojo-ai-pm-s2.git
cd dojo-ai-pm-s2
```

## Démarrer

Ouvre `fiche-trainee.md` et suis les 3 étapes. Le cœur tient en une commande :

```
/spec-grill-session entries-list
```
