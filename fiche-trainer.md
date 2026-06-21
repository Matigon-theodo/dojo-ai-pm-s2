# Séance 2 - Faire des specs · Fiche Trainer

## 👁️ En un coup d'œil

- **Objectif** : faire produire au PM une spec fonctionnelle **durcie puis vérifiée**, en lui faisant
  vivre les trois temps : v1 à la main → grill-me sur analyse fournie → vérification par agent tiers.
- **Format** : binôme, chacun sa machine. ~45 min, mains au clavier ~30 min.
- **Durée** : 45 min (time-box **strict** - voir Notes animateur).
- **Prérequis (S0)** : Ketchup Compta tourne sur `localhost:8080` (`admin`/`admin123`), `claude` répond,
  `gh auth status` OK. **Dossier pré-mâché récupérable** (branche à puller OU dossier fourni).

## 🎓 Ce qu'on veut faire apprendre

- **Compétence cible** : transformer un besoin flou en une spec qu'un dev - ou un agent - peut construire
  sans revenir voir le PM, et savoir la **faire fiabiliser**.
- **Les gestes** : (1) explorer une page ; (2) se faire **grill-er** une question à la fois et
  **trancher** ; (3) faire **vérifier** son livrable par un agent tiers et corriger.
- **Les pièces** (montée en exigence) : `spec-v1.md` (à la main) → `spec.md` durcie → `spec.md` **vérifiée**.
- **Le reframe à ancrer** :
  > Une spec ne se rédige pas, elle se **durcit** puis se **fait vérifier**. Sans analyse tu spécules ;
  > avec le grill-me tu **tranches** ; avec un regard tiers tu **fiabilises**. Le PM **possède chaque
  > décision**.

## ⏱️ Déroulé minuté

| Temps      | Séquence                                                                 |
| ---------- | ------------------------------------------------------------------------ |
| 0-3 min    | Intro + reframe (« durcir puis vérifier ») + objectif du jour            |
| 3-15 min   | **Étape 1** - spec v1 à la main (Claude seul)                            |
| 15-33 min  | **Étape 2** - grill-me avec `spec-grill-session` (le cœur)               |
| 33-44 min  | **Étape 3** - vérification par l'agent `spec-verifier`                    |
| 44-45 min  | Clôture + question de synthèse                                           |

## 🧵 Le fil rouge : l'objectif du trainee

Produire une `spec.md` durcie **et vérifiée** pour la page « Toutes les Écritures », et nommer ce que le
grill-me + l'agent ont attrapé. Page cible : `/modules/entries/list.php`.

---

## Étape 1 - Spec v1 à la main

- **La pièce produite** : `spec-v1.md` (rapide, incomplète par construction).
- **Pitch à lire au groupe** : « Avant tout outil : ouvrez la page, demandez à Claude de vous écrire la
  spec. En 10 minutes vous aurez quelque chose qui *paraît* bien. Gardez-le : on va le défoncer après. »
- **Consigne binôme** : chacun fait sa v1 ; comparez vite à l'oral ce que Claude a mis (et pas mis).
- **La bonne manière de faire** : Claude Code **seul**, sans skill, sans analyse. Explorer (2-3 questions),
  puis « Écris-moi la spec fonctionnelle de cette page » → `spec-v1.md`. Ne pas fignoler.
- **Ce que la pièce apporte** : le **point de comparaison** qui rend visibles les trous aux étapes 2 et 3.
- **Le geste exercé** : explorer une page + première formalisation (geste S1 réappliqué).

<details><summary>💡 Tips animateur</summary>

- Résiste à l'envie de « bien faire » la v1. Plus elle est naïve, plus le contraste est fort.
- Trous typiques laissés par Claude : accès/permissions, liste vide, cas d'erreur, in/out scope.
</details>

---

## Étape 2 - Le grill-me avec l'usine · LE CŒUR

- **La pièce produite** : `spec.md` durcie (Pourquoi + Mermaid, Démo, Cas d'utilisation, **Précisions
  issues du grill-me** dont une sous-section **« Décisions produit (hors legacy) »**, table Touchpoints
  ✅/❌, **Points d'entrée**, Gherkin 100 %).
- **Pitch à lire au groupe** : « On vous donne l'analyse technique déjà faite. Claude va vous cuisiner,
  une question à la fois, et vous recommander une réponse. Votre job : **trancher** - et ne jamais
  inventer ce que le code ne fait pas. »
- **Consigne binôme** : chacun mène son grill-me ; si l'un bloque, l'autre l'aide à formuler une
  **décision métier** (pas une réponse technique).
- **La bonne manière de faire** : cloner le dépôt `dojo-ai-pm-s2`, lancer
  **`/spec-grill-session entries-list`**, répondre **une question à la fois**, laisser Claude écrire
  `spec.md`. **Ni tracer ni agent-browser** - tout l'amont est fourni.
- **Ce que la pièce apporte** : une spec **actionnable** où chaque cas limite et chaque accès a été
  tranché par le PM, et où les ajouts « hors legacy » sont isolés.
- **Le geste exercé** : se faire grill-er, **vérifier ses hypothèses contre la vérité terrain**, et
  posséder ses décisions.

<details><summary>💡 Tips animateur - LE moment pédagogique (permissions)</summary>

- ⚠️ **Le piège volontaire** : Claude va probablement recommander une règle « à base de rôles » (cacher
  « Nouvelle écriture » à un lecteur). **Or l'app n'a AUCUN rôle** : la table `users` n'a pas de colonne
  `role`, aucun `require_role` (le `CLAUDE.md` du legacy le prétend à tort - l'`analysis.md` dit vrai).
- C'est **le** moment à faire vivre : le PM doit **confronter la reco à l'`analysis.md`**, constater
  qu'il n'y a pas de rôles, et **trancher** : garder le comportement legacy (ouvert à tous) et noter
  « introduire des rôles » comme **décision produit hors legacy**, clairement séparée. S'il l'inscrit
  comme du legacy, l'agent de l'étape 3 le recalera - et c'est tant mieux.
- Autres branches : **liste vide**, **in/out scope** des liens vers `edit.php` (navigation = hors-scope),
  recherche (libellé + n° pièce), pagination, `journal_id` invalide → liste vide.
- Si le clone coince, débloquez-vous à deux (le binôme est le filet). Ne pas perdre 5 min sur git.
</details>

---

## Étape 3 - Vérification par un agent tiers

- **La pièce produite** : `spec.md` **vérifiée** (corrigée d'après le rapport de l'agent).
- **Pitch à lire au groupe** : « Vous pensez avoir une bonne spec ? On la passe à un agent qui relit le
  vrai code et cherche vos trous. C'est exactement ce qu'on veut **avant** de la donner aux devs. »
- **Consigne binôme** : lire le rapport ensemble, repérer la remarque la plus surprenante, corriger.
- **La bonne manière de faire** : le skill lance un sous-agent de vérification qui relit le vrai code
  legacy (sans tracer) ; lire son verdict et ses recommandations (chacune cite un fichier legacy), puis
  corriger `spec.md` avec Claude.
- **Ce que la pièce apporte** : la **fiabilité**. Un regard indépendant attrape ce que l'auteur ne voit
  plus (touchpoint manqué, cas non couvert, point d'entrée mal qualifié, décision mal étiquetée).
- **Le geste exercé** : **faire valider son travail par un tiers adversarial** et itérer.

<details><summary>💡 Tips animateur</summary>

- Dédramatise le FAIL : l'agent est **fait pour trouver des trous**. Un rapport rouge = de la valeur, pas
  un échec personnel.
- Insiste sur le fait qu'il **relit le code réel** (pas la spec) : c'est ça qui rend le regard crédible.
- Time-box : vise **une** boucle de correction, pas la perfection. L'objectif est le **geste**.
- La vérif relit le **vrai code legacy** (présent via le submodule du clone) ; elle n'utilise pas le tracer.
</details>

---

> ## ✅ Livrable et réussite - C'est bon si…
> - le PM a une **`spec.md` durcie** avec une section « Précisions issues du grill-me » contenant au moins
>   une **décision « hors legacy »** proprement isolée (typiquement les rôles) ;
> - le PM a **lancé la vérification** (le relecteur indépendant) et traité (ou explicitement noté) ses remarques ;
> - le PM cite **≥ 3 trous** comblés vs sa v1.

---

## 🗒️ Notes animateur

- **Time-box strict.** La S1 a glissé « big time » : annonce les durées, tiens l'étape 1 à ~12 min
  (chrono visible). Le grill-me reste le gros du budget ; l'étape 3 vise **une** boucle de correction.
- **L'arc = durcir PUIS vérifier.** Les deux « aha » : (2) le grill-me te fait trancher ce que tu
  ignorais ; (3) l'agent tiers attrape ce qu'il te reste. Ne sacrifie aucun des deux ; coupe plutôt dans
  le fignolage de rédaction.
- **Récupération du dojo** : les PM clonent le dépôt `dojo-ai-pm-s2` (`git clone --recurse-submodules`)
  au début de l'étape 2 - pas avant (l'étape 1 doit rester mains nues).
- **Pré-requis formateur (fait en amont)** : `docs/features/entries-list/` rempli (`analysis.md` avec
  call graph réel + `browser/`), et `analysis/entry-points.md` à jour. Les PM ne touchent jamais à
  `bin/tracer` ni à `agent-browser`.
- **Le piège des rôles est intentionnel** et c'est le cœur pédagogique de l'étape 2 - ne le « corrige »
  pas à l'avance dans l'`analysis.md` : il y est documenté honnêtement (aucun rôle), c'est au PM de le
  découvrir et de trancher.
- **Tease S3** : « Avec ce `spec.md` vérifié, on pourrait générer un proto cliquable - prochaine séance. »
