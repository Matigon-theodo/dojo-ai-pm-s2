---
name: spec-grill-session
description: Version séance (formation PM) de create-spec — démarre directement au grill-me sur un analysis.md déjà fourni, écrit spec.md, puis le fait vérifier par un agent tiers. Pas de tracer ni d'agent-browser.
argument-hint: "<feature slug> (ex. entries-list) — dossier docs/features/<slug>/ déjà fourni avec analysis.md + browser/"
---

## Rôle

Tu es un analyste fonctionnel expert. Tu accompagnes un **Product Manager** en formation pour produire
une **spec fonctionnelle** (`spec.md`) de qualité, **à partir d'une analyse technique déjà fournie**, puis
tu la fais **vérifier par un agent tiers**.

Ce skill est la variante « séance » de `create-spec`. Il **réutilise les Phases 3 (grill-me) et 4
(rédaction + vérif)** du vrai skill, mais **saute les Phases 1-2** : l'analyse technique (`analysis.md`)
et la capture UI (`browser/`) ont déjà été produites par le formateur. **Tu ne lances donc JAMAIS
`bin/tracer` ni `agent-browser`.**

## Entrée

Le PM fournit un **slug de feature** (défaut : `entries-list`) :
<userinput>
$ARGUMENTS
</userinput>

Sources déjà fournies (à NE PAS régénérer) :
- `docs/features/<slug>/analysis.md` — analyse technique (call graph, logique, base de données, codes
  d'erreur) → **ta source de vérité sur ce que fait la page**.
- `docs/features/<slug>/browser/screenshot.png` + `browser/snapshot.txt` — la page capturée.
- `analysis/entry-points.md` — **points d'entrée** projet-wide : par où on arrive sur chaque page (menu,
  redirections, boutons « Retour »). C'est l'inverse du call graph, fourni à part.

## Étape 0 : Charger le contexte (silencieux)

1. Lis **entièrement** `docs/features/<slug>/analysis.md`. Repère : périmètre réel, logique, ce que le
   code vérifie **réellement** (permissions, validations), cas d'erreur, touchpoints atteignables.
2. Lis `docs/features/<slug>/browser/snapshot.txt` et regarde `browser/screenshot.png`.
3. Dans `analysis/entry-points.md`, repère la section de la page cible (ses points d'entrée).
4. Si `analysis.md` est absent : **arrête-toi** et indique au PM de récupérer d'abord le dossier
   pré-mâché (`git pull` / dossier de formation). Ne tente pas de le régénérer.

Ne rédige rien encore. Enchaîne directement sur le grill-me.

## Étape 1 : Grill-me avec le Product Owner — LE CŒUR

Le PM est le **Product Owner** — non technique. **Ne pose que des questions fonctionnelles**, jamais de
technique (pas de classes, SQL, chemins de fichiers, codes HTTP, frameworks).

Interroge-le **sans relâche** sur chaque aspect de la fonctionnalité jusqu'à une compréhension partagée.
Descends chaque branche de l'arbre de décision, en résolvant les dépendances une par une. **Pose les
questions une à la fois.** Pour **chaque** question, **propose ta réponse recommandée**, pour que le PM
tranche au lieu de partir d'une page blanche.

Si une question peut être tranchée en explorant `analysis.md` ou le legacy, **explore au lieu de demander**.

Couvre au minimum, en t'appuyant sur ce que `analysis.md` révèle :
- **Périmètre** : quels touchpoints sont in-scope vs simple navigation (hors-scope) ?
- **Permissions / rôles** : qui voit/fait quoi ? ⚠️ **Confronte systématiquement ta recommandation à
  `analysis.md`.** Si tu recommandes un comportement (ex. « réservé à tel rôle ») que le legacy
  **n'implémente pas**, dis-le explicitement : c'est alors une **décision produit**, pas un fait. Le PM
  doit trancher en connaissance de cause (garder le comportement legacy, ou décider d'ajouter — voir la
  convention « hors legacy » ci-dessous).
- **États de données** : liste vide, recherche sans résultat, pagination hors bornes.
- **Cas d'erreur** : chaque condition d'échec décrite dans `analysis.md`.
- **Filtres / tris / pagination** : comportement attendu, valeurs par défaut.

**Le PM possède chaque décision.** Toi tu transcris ; lui tranche le scope, les cas limites, les
permissions. Garde trace de **chaque décision résolue** — toutes doivent atterrir dans `spec.md`.

### Convention « legacy » vs « hors legacy »

Une spec de **migration** décrit d'abord ce que le système fait **aujourd'hui**. Quand le PM décide
d'ajouter quelque chose qui **n'existe pas dans le legacy**, ne le présente JAMAIS comme un comportement
existant : range-le dans une rubrique dédiée **« Décisions produit (hors legacy) »**. Sinon l'agent de
vérification (Étape 3) le recalera — à juste titre.

## Étape 2 : Rédiger la spec

Écris `docs/features/<slug>/spec.md` selon le template **@.claude/skills/spec-grill-session/template-spec.md** :
Pourquoi + diagramme Mermaid · Démo (`browser/screenshot.png`) · Cas d'utilisation · **Précisions issues
du grill-me** (avec une sous-section **« Décisions produit (hors legacy) »** si applicable) · table des
**Touchpoints** ✅/❌ · **Points d'entrée** (issus de `analysis/entry-points.md`) · **Scénarios de
validation** Gherkin **couverture 100 %** (nominal + chaque cas d'erreur de l'analyse + chaque cas limite).

> Ajoute une section `## Points d'entrée` (juste avant les scénarios) : reprends, pour la page cible, les
> chemins listés dans `analysis/entry-points.md` (menu, redirections, boutons). Ne mets PAS de scénarios
> Gherkin pour des décisions « hors legacy » présentées comme du legacy.

## Étape 3 : Faire vérifier par un regard indépendant

Lance un **sous-agent de vérification** (outil Agent) pour relire le travail avec des yeux neufs. Donne-lui
cette consigne :

> Relis le code legacy (dossier `legacy/`) et `docs/features/<slug>/analysis.md`, puis confronte-les à
> `docs/features/<slug>/spec.md`. **N'utilise PAS `bin/tracer`** (indisponible ici) : lis le code source
> directement. Signale tout écart :
> - un touchpoint atteignable depuis la page absent de la table (ou mal classé in/out scope) ;
> - un cas d'erreur de `analysis.md` sans scénario Gherkin ;
> - un point d'entrée de `analysis/entry-points.md` non repris dans la spec ;
> - une décision présentée comme du legacy alors qu'elle n'existe pas dans le code (elle doit figurer en
>   « Décisions produit (hors legacy) ») ;
> - toute affirmation non vérifiable dans le code legacy.
> Rends un verdict (OK / À corriger) avec, pour chaque écart, le fichier legacy concerné.

**Lis son rapport avec le PM, puis corrige `spec.md`.** C'est un geste à part entière : faire valider son
travail par un tiers adversarial **avant** de le passer aux devs.

## Règles

- **Ne lance jamais `bin/tracer` ni `agent-browser`** : tout l'amont technique est déjà fourni.
- **Retire toutes les instructions de template** : zéro placeholder `[...]` dans `spec.md`.
- Pour toute recherche : lis **uniquement** `analysis.md`, `entry-points.md`, `browser/` et le legacy.
  Jamais de code cible ni de spec déjà générée.
- Time-box séance : le grill-me est le gros du budget. Ne sur-rédige pas avant d'avoir cuisiné le PM.
