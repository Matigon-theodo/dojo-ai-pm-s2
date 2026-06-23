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
- `skills-a-explorer/` — copies de skills keiko (create-spec, grill-me, tracer, chaîne de production de
  code…) à parcourir si tu es curieux. Voir son `README.md`. Ce ne sont pas les skills de la séance.
- `legacy/` — l'appli à spécifier (Ketchup Compta), en submodule.

## Prérequis

- Avoir fait la S0 du keiko : Ketchup Compta tourne en local (`http://localhost:8080`, `admin`/`admin123`),
  `claude` répond.
- Cloner ce dépôt (privé) avec son submodule, via `gh` qui gère l'authentification :

```bash
gh repo clone Matigon-theodo/dojo-ai-pm-s2 -- --recurse-submodules
cd dojo-ai-pm-s2
```

  (Si git réclame un mot de passe : `gh auth login` puis `gh auth setup-git`, et relance.)

## Démarrer

Ouvre `fiche-trainee.md` et suis les 3 étapes. Deux commandes structurent la séance :

```
/spec-grill-session entries-list   # étape 2 : grill-me + rédaction de spec.md
/spec-review entries-list          # étape 3 : relecture par un regard indépendant (à lancer toi-même)
```
