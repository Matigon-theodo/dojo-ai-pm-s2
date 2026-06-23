---
name: spec-review
description: Fait relire une spec par un regard indépendant (sans tracer) et liste les écarts vs le code legacy. À lancer après spec-grill-session, quand spec.md est prête.
argument-hint: "<feature slug> (ex. entries-list) — relit docs/features/<slug>/spec.md"
---

## Rôle

Tu organises la **relecture indépendante** d'une spec déjà rédigée. Le but pédagogique : le PM confie son
travail à un tiers (un relecteur qui n'a pas participé à la rédaction), comme on demanderait l'avis d'un
collègue avant de passer la spec aux devs. On ne se relit jamais aussi bien soi-même.

## Entrée

Le PM fournit un **slug de feature** (défaut : `entries-list`) :
<userinput>
$ARGUMENTS
</userinput>

Pré-requis : `docs/features/<slug>/spec.md` existe (produit par `spec-grill-session`). S'il est absent,
arrête-toi et indique au PM de produire d'abord la spec avec `/spec-grill-session <slug>`.

## Ce que tu fais

Lance un **sous-agent relecteur** (outil Agent) avec un regard neuf. Donne-lui exactement cette consigne :

> Tu es un relecteur indépendant. Relis le code legacy (dossier `legacy/`), `docs/features/<slug>/analysis.md`
> et `analysis/entry-points.md`, puis confronte-les à `docs/features/<slug>/spec.md`. **N'utilise PAS
> `bin/tracer`** (indisponible ici) : lis le code source directement. Signale tout écart :
> - un touchpoint atteignable depuis la page absent de la table (ou mal classé in/out scope) ;
> - un cas d'erreur de `analysis.md` sans scénario Gherkin ;
> - un point d'entrée de `analysis/entry-points.md` non repris dans la spec ;
> - une décision présentée comme du legacy alors qu'elle n'existe pas dans le code (elle doit figurer en
>   « Décisions produit (hors legacy) ») ;
> - toute affirmation non vérifiable dans le code legacy.
> Rends un verdict (OK / À corriger) avec, pour chaque écart, le fichier legacy concerné. Lecture seule :
> ne modifie aucun fichier.

Quand le sous-agent a rendu son rapport : **présente-le au PM**, explique chaque écart simplement, puis
**propose de corriger `spec.md`** point par point (le PM tranche ce qu'il garde / ajuste). Une boucle de
correction suffit pour la séance.

## Règles

- **Ne lance jamais `bin/tracer` ni `agent-browser`** : le relecteur lit le code legacy directement.
- Le relecteur doit avoir un **regard neuf** : ne lui souffle pas les réponses, laisse-le trouver les écarts.
- Reste fonctionnel face au PM : explique les écarts en langage métier, pas en jargon technique.
