# Séance 2 - Faire des specs · Fiche Trainer

> Cette fiche est auto-suffisante. Tu n'as pas besoin de savoir coder ni de connaître le projet.
> Si le sujet t'est étranger, lis d'abord la section « Ce que tu dois comprendre avant d'animer » :
> elle te donne le vocabulaire, le déroulé exact, et surtout le corrigé (les bonnes réponses attendues)
> pour que tu saches à tout moment si ta cohorte est sur les rails.

---

## 👁️ En un coup d'œil

- Objectif : faire produire à chaque PM une spec fonctionnelle de qualité, en l'améliorant par un
  questionnement (le grill-me) puis en la faisant vérifier.
- Public : Product Managers (non techniques).
- Format : binôme, chacun sa machine. ~45 min, mains au clavier ~30 min.
- Durée : 45 min, time-box strict.
- Page support : « Toutes les Écritures » de l'app Ketchup Compta (`/modules/entries/list.php`).
- Livrables du PM : `spec-v1.md` (brouillon), puis `spec.md` (de qualité et vérifiée).

---

## 🎯 L'intention de la séance

Apprendre aux PM à s'outiller avec Claude pour rédiger des specs, et à prendre les bonnes pratiques dès le
départ. On ne vise pas un document qu'on retouchera dix fois pendant le développement, mais une
spec proche du « right first time » (bonne du premier coup) : assez complète et tranchée pour qu'un dev,
ou un agent, la construise sans revenir poser des questions.

Plus largement, c'est un pas vers la vision du programme : passer d'un PM qui spécifie seul dans son coin
à un PM qui s'appuie sur une « usine IA » qu'il bâtit avec ses devs.

---

## 🧠 Ce que tu dois comprendre avant d'animer

### En une phrase
Une spec (spécification) décrit, en langage métier, ce que fait une fonctionnalité : ce que l'utilisateur
voit, fait, et ce qui doit se passer dans tous les cas (y compris les cas d'erreur). C'est le document
qu'un développeur lit pour construire la fonctionnalité sans avoir à revenir poser des questions.

### L'idée clé de la séance (le « reframe »)
> Une bonne spec ne s'écrit pas d'un coup. On la monte en qualité (vers le « right first time ») en deux
> mouvements : on se fait d'abord interroger pour combler les trous (le grill-me), puis on la fait
> vérifier par un tiers.

Les PM vont le vivre en 3 temps : (1) ils écrivent une spec vite fait tout seuls, (2) un outil les cuisine
question par question pour la solidifier, (3) un agent indépendant la relit et pointe ce qui manque.

### Le grill-me, d'où ça vient
Le grill-me est une technique popularisée par Matt Pocock. L'idée : se faire poser toutes les questions
difficiles le plus tôt possible - on dit « shift left » les questions. Plutôt que de
découvrir les zones grises tard (pendant le dev, ou en recette, quand corriger coûte cher), on les fait
remonter dès l'écriture de la spec, là où trancher ne coûte presque rien.

### Le petit lexique (suffisant pour animer)
- Legacy : l'ancienne application (ici en PHP) qu'on est en train de moderniser.
- Touchpoint : un « point d'action » de la page (une URL ou une action déclenchée), par ex. ouvrir la
  page, soumettre un formulaire, cliquer un lien.
- Périmètre (scope) in / out : est-ce que ce touchpoint fait partie de cette page (in-scope ✅) ou est-ce
  juste un lien vers une autre page (out-of-scope ❌, navigation) ?
- Point d'entrée : par où on arrive sur la page (le menu, un bouton « Retour », une redirection). C'est
  l'inverse d'un touchpoint (qui, lui, part de la page).
- `analysis.md` : l'analyse technique déjà préparée par le tech lead. Elle décrit factuellement ce que la
  page fait (sa logique, ses données, ses cas d'erreur). C'est la vérité terrain. Le PM s'appuie dessus,
  il n'a pas à la produire.
- grill-me : le fait de se faire interroger sans relâche, une question à la fois, pour trancher chaque
  détail (voir « d'où ça vient » ci-dessus).
- Gherkin : une façon d'écrire des scénarios de test en langage simple (« Étant donné… Quand… Alors… »).
  Pas besoin d'être technique pour les lire.
- « hors legacy » : une décision que le PM veut ajouter mais qui n'existe pas dans l'app actuelle. On la
  note à part pour ne pas la confondre avec l'existant.
- PO (Product Owner) : le rôle que joue le PM pendant le grill-me (celui qui tranche le besoin).

### ⭐ LE point à ne pas rater : le piège des rôles
Pendant le grill-me, l'outil va probablement suggérer une règle « selon le rôle de l'utilisateur »
(par ex. « cacher le bouton *Nouvelle écriture* aux profils en lecture seule »). C'est un piège volontaire
et c'est le cœur pédagogique de la séance.

La vérité : l'application n'a aucune notion de rôle. Tous les utilisateurs connectés ont exactement les
mêmes droits. Le PM doit le découvrir en vérifiant l'`analysis.md`, et trancher :
- soit on reproduit l'existant (tout le monde voit tout) ;
- soit on décide d'ajouter des rôles, mais alors c'est une décision produit « hors legacy », à noter à
  part (pas comme si ça existait déjà).

👉 Si tu ne retiens qu'une chose pour animer : « ne jamais inventer ce que le code ne fait pas ; vérifier,
puis décider en connaissance de cause. »

---

## ✅ Checklist avant la séance (5-10 min avant)

Fais-la pour toi, puis vérifie-la avec chaque binôme au démarrage.

- [ ] L'app tourne : ouvrir <http://localhost:8080> → la page de connexion s'affiche (login `admin` /
      `admin123`). Sinon : relancer le Docker du keiko (`docker compose up -d` dans le dossier keiko).
- [ ] Chaque PM a `claude` qui répond et `gh auth status` OK (acquis en S0). ⚠️ Fais-leur lancer
      `gh auth setup-git` une fois : sinon `git clone` du repo privé réclamera un mot de passe et
      échouera (panne n°1 de la séance). Avec `gh repo clone`, c'est géré automatiquement.
- [ ] Chaque PM a accès en lecture au repo `dojo-ai-pm-s2` (sinon le clone échouera - voir Pannes).
- [ ] Toi, tu as relu le corrigé du grill-me plus bas (le tableau des décisions attendues).

---

## ⏱️ Déroulé minuté

| Temps      | Séquence                                                                 |
| ---------- | ------------------------------------------------------------------------ |
| 0-5 min    | Intro + reframe (« monter en qualité puis vérifier ») + objectif du jour |
| 5-15 min   | Étape 1 - spec v1 à la main (Claude seul)                                |
| 15-35 min  | Étape 2 - grill-me avec `spec-grill-session` (le cœur)                   |
| 35-45 min  | Étape 3 - relecture (`/spec-review`) + clôture                           |

---

## Étape 1 - La spec « à la main » (~10 min)

Objectif : produire vite une spec naïve (`spec-v1.md`) pour, plus tard, mesurer tout ce qui lui manquait.
Elle doit être imparfaite : c'est voulu.

Ce que tu dis au groupe :
> « Vous venez de recevoir un ticket : spécifier la page *Toutes les Écritures*. Réflexe normal : on ouvre
> la page, on demande à Claude de l'écrire. En 10 minutes vous aurez quelque chose qui *paraît* bien.
> Gardez-le précieusement : on va le malmener juste après. »

Ce que le PM fait (exactement) :
1. Ouvre <http://localhost:8080>, se connecte (`admin` / `admin123`), va dans Écritures → Toutes les
   écritures.
2. Dans Claude Code (sans aucune commande spéciale) : « *Explique-moi ce que fait la page
   /modules/entries/list.php.* »
3. Puis : « *Écris-moi la spec fonctionnelle de cette page.* »
4. Colle la réponse dans un fichier `spec-v1.md` et le garde ouvert.

Ce que le PM doit voir : une spec courte (objectif + quelques cas d'usage). Normal qu'elle oublie des
choses : les permissions, la liste vide, les cas d'erreur, le périmètre précis, la pagination.

Réussite de l'étape : un fichier `spec-v1.md` existe. Ne pas chercher la qualité ici.

Si ça coince : voir le tableau Pannes. Tiens la barre à ~10 min - coupe court si besoin, l'important est
d'avoir un brouillon à comparer.

> 💡 Résiste à l'envie d'aider les PM à « bien faire » leur v1. Plus elle est naïve, plus l'étape 2 frappe.

---

## Étape 2 - Le grill-me (~20 min) · LE CŒUR DE LA SÉANCE

Objectif : monter la spec en qualité en se faisant interroger, sur la base de l'analyse technique fournie.
Le PM tranche chaque décision (et n'invente rien).

Ce que tu dis au groupe :
> « Maintenant on vous donne l'analyse technique déjà faite par le tech lead. Vous lancez un outil qui va
> vous cuisiner, une question à la fois, en vous proposant une réponse à chaque fois. C'est le *grill-me*
> de Matt Pocock : on fait remonter toutes les questions maintenant, pas pendant le dev. Votre boulot :
> trancher - et surtout, ne jamais affirmer ce que le code ne fait pas. »

Ce que le PM fait (exactement) :
1. Récupère le dossier de la séance. Le repo est privé → cloner via `gh` (il authentifie tout seul, pas
   de mot de passe à taper) :
   ```bash
   gh repo clone Matigon-theodo/dojo-ai-pm-s2 -- --recurse-submodules
   cd dojo-ai-pm-s2
   ```
2. Vérifie qu'il a bien le dossier fourni : `ls docs/features/entries-list/` (doit montrer `analysis.md`
   et `browser/`).
3. Lance Claude Code dans ce dossier, puis tape : `/spec-grill-session entries-list`
4. Répond aux questions une par une. À chaque fois l'outil propose une réponse : valider, ajuster, ou
   trancher autrement.
5. À la fin, l'outil écrit `docs/features/entries-list/spec.md`. Le PM le relit.

Ce que le PM doit voir : l'outil pose des questions une à une (pas un mur de texte), puis produit un
`spec.md` structuré : Pourquoi (+ schéma), Démo (capture), Cas d'utilisation, Précisions issues du
grill-me, table des Touchpoints (✅/❌), Points d'entrée, et des scénarios de validation.

### 🔑 Le corrigé du grill-me (garde-le sous les yeux)
Voici les décisions attendues pour cette page. Si un PM s'en écarte sans raison, recadre-le vers
l'`analysis.md`.

| Question posée (en substance) | Bonne décision attendue |
| --- | --- |
| Les liens « Nouvelle écriture », « Voir », n° de pièce sont-ils dans le périmètre ? | Non : ce sont des liens de navigation vers la page d'édition → hors-scope (❌). La page elle-même est le seul touchpoint in-scope (✅). |
| ⭐ Faut-il masquer des actions selon le rôle de l'utilisateur ? | Piège. L'app n'a aucun rôle. On reproduit l'existant (ouvert à tout utilisateur connecté) ; toute idée de rôle est notée en « Décision produit (hors legacy) ». |
| Que se passe-t-il si la liste est vide / la recherche ne donne rien ? | Message « Aucune écriture trouvée. » et « Total : 0 écritures ». |
| Sur quoi porte la recherche ? | Sur le libellé OU le numéro de pièce (correspondance partielle). |
| Que propose le filtre journal ? | Seulement les journaux actifs, valeur par défaut « Tous ». |
| Comment marche la pagination ? | 30 écritures par page ; une page hors limites est ramenée à une page valide. |
| Le tri est-il configurable ? | Non : de la plus récente à la plus ancienne. |
| Et si l'utilisateur n'est pas connecté ? | Redirigé vers la connexion, message « Veuillez vous connecter. ». |
| Quels sont les points d'entrée (par où on arrive) ? | Le menu « Toutes les écritures », et depuis l'écran d'édition : « Retour à la liste », « Annuler », et le cas « écriture introuvable ». |

Réussite de l'étape : un `spec.md` existe, avec une section « Décisions produit (hors legacy) » contenant
la décision sur les rôles, une table de touchpoints, et des scénarios.

> 💡 Si le PM répond « technique » (parle de code, de SQL…), recadre : « réponds en métier : *qui* voit
> *quoi*, et *qu'est-ce qui s'affiche* ? ». S'il bloque, son binôme l'aide à formuler une décision, pas une
> solution technique.

---

## Étape 3 - Faire vérifier par un agent tiers (~10 min)

Objectif : découvrir la valeur d'un regard indépendant, en le déclenchant soi-même. Le PM lance un agent
qui relit le vrai code et pointe ce que lui (et le grill-me) ont laissé passer.

Ce que tu dis au groupe :
> « Vous pensez avoir une bonne spec ? Maintenant vous la confiez vous-même à un relecteur indépendant -
> un agent qui n'a pas participé à votre rédaction - dont le seul job est de relire le vrai code et de
> trouver vos trous. C'est exactement ce qu'on veut faire avant de donner la spec aux devs. »

Ce que le PM fait : il lance lui-même la relecture avec `/spec-review entries-list` (c'est un geste à
part, distinct de la création). Il laisse le relecteur tourner, lit le rapport, puis corrige `spec.md`
avec Claude. Une seule boucle de correction suffit.

Ce que le PM doit voir : un rapport qui rend un verdict (OK / À corriger) et, pour chaque écart, cite le
fichier de code concerné. Exemples d'écarts typiques et fréquents :
- un cas d'erreur de l'analyse sans scénario de test correspondant ;
- un point d'entrée oublié ;
- une décision « hors legacy » écrite comme si elle existait dans l'app (l'agent la recalera - et c'est
  exactement le but) ;
- une affirmation invérifiable dans le code.

Comment guider la correction : demande au PM de reprendre chaque remarque et de dire à Claude « corrige ce
point dans spec.md ». Pas besoin d'un verdict parfait : l'important est le geste (faire vérifier,
comprendre, corriger).

Réussite de l'étape : le PM a lancé lui-même `/spec-review`, lu le rapport, et traité (ou noté
explicitement) les remarques.

> 💡 Dédramatise un rapport sévère : l'agent est *fait* pour trouver des trous. Un rapport rouge = de la
> valeur trouvée, pas un échec. Insiste : il relit le code réel, pas la spec - c'est ça qui rend son regard
> crédible.

---

> ## ✅ Livrable et réussite - C'est bon si…
> - le PM a une `spec.md` de qualité avec une section « Précisions issues du grill-me » contenant au moins
>   une décision « hors legacy » proprement isolée (typiquement les rôles) ;
> - le PM a lancé lui-même la relecture (`/spec-review`) et traité (ou explicitement noté) ses remarques ;
> - le PM sait citer au moins 3 trous comblés par rapport à sa `spec-v1.md`.

---

## 🆘 Pannes & déblocage (le plus important pour un trainer)

| Symptôme | Cause probable | Déblocage |
| --- | --- | --- |
| `/spec-grill-session` n'apparaît pas dans Claude (taper `/`) | Claude n'est pas lancé dans le dossier `dojo-ai-pm-s2` | Quitter Claude, faire `cd dojo-ai-pm-s2`, relancer `claude`. Le skill vit dans `.claude/skills/` du dossier. |
| Le clone demande un mot de passe / « authentication failed » | git n'est pas branché sur le compte gh (le mot de passe GitHub ne marche plus pour git depuis 2021) | `gh auth login` puis `gh auth setup-git`, et relancer. Plus simple : `gh repo clone …`. Filet : le `.zip`. C'EST LA PANNE N°1, anticipe-la. |
| `git clone` / `gh repo clone` échoue (« repository not found ») | Le PM n'a pas encore accepté l'invitation au repo privé | Faire accepter l'invitation (mail GitHub ou page `/invitations`). En attendant : binôme sur une machine qui a accès, ou le `.zip`. |
| Le dossier `legacy/` est vide après le clone | Submodule non récupéré | `git submodule update --init` dans le dossier cloné. |
| `localhost:8080` ne répond pas | L'app n'est pas démarrée | Relancer le Docker du keiko. À défaut, l'étape 1 peut se faire en demandant à Claude de lire le code de la page. |
| L'outil écrit la spec sans poser de questions | Il a « foncé » | Lui dire : « *Pose-moi les questions une par une avant d'écrire la spec.* » |
| Le PM répond en termes techniques | Confusion de rôle | Recadrer : « réponds en métier : qui voit quoi, qu'est-ce qui s'affiche ? » |
| Le PM invente des rôles/permissions | Le piège a fonctionné… trop bien | Faire ouvrir `analysis.md` : « où est-il écrit qu'il y a des rôles ? ». Le ranger en « hors legacy ». |
| La vérification parle d'une erreur `bin/tracer` | Quelqu'un a tenté de lancer le tracer | Normal : le dojo n'utilise pas le tracer. Rassurer, ignorer, la vérif lit le code directement. |
| On déborde sur le temps | Étape 1 ou rédaction trop léchée | Couper le fignolage. Le grill-me (étape 2) est prioritaire ; l'étape 3 = une seule boucle. |

---

## ❓ Questions que ta cohorte va poser (et tes réponses)

- « C'est quoi un *touchpoint* ? » → Un point d'action de la page (une URL ou une action déclenchée). Ici :
  afficher la liste, filtrer. Les liens vers la page d'édition n'en sont pas (c'est de la navigation).
- « Pourquoi on n'invente pas les rôles, ce serait mieux ? » → Une spec de migration décrit d'abord
  l'existant. Les améliorations sont légitimes mais se notent à part (« hors legacy »), pour ne pas tromper
  le dev sur ce que fait l'app aujourd'hui.
- « Pourquoi ne pas demander direct la spec à Claude (étape 1) et s'arrêter là ? » → Justement, on le fait
  à l'étape 1 - et on voit que ça laisse des trous. Le grill-me et la vérif servent à les combler.
- « L'analyse technique, qui l'a faite ? » → Le tech lead, en amont, avec l'outillage keiko. Le PM part de
  ce résultat ; il n'a pas à le produire (ce sera l'objet d'autres séances).
- « À quoi servent les scénarios *Gherkin* ? » → Ce sont les cas concrets qu'un testeur (ou un agent)
  suivra pour valider. Couverture visée : tous les cas, y compris erreurs et cas limites.

---

## 🗒️ Notes animateur

- Time-box strict. Annonce les durées, garde un chrono visible. L'étape 1 ne doit pas dépasser ~10 min. Le
  grill-me est le gros du budget ; l'étape 3 vise une boucle de correction.
- L'arc : on monte en qualité (grill-me) puis on fiabilise (vérification). Deux moments « aha » : (2) le
  grill-me te fait trancher ce que tu ignorais ; (3) l'agent tiers attrape ce qu'il te reste. Ne sacrifie
  aucun des deux.
- Le piège des rôles est intentionnel. Ne le « corrige » pas à l'avance : il est documenté honnêtement
  dans l'`analysis.md` (aucun rôle), c'est au PM de le découvrir et de trancher.
- Récupération du dojo : au début de l'étape 2 (pas avant - l'étape 1 reste mains nues). Repo privé →
  `gh repo clone Matigon-theodo/dojo-ai-pm-s2 -- --recurse-submodules`. Si ça réclame un mot de passe,
  c'est l'auth git/gh (`gh auth login` + `gh auth setup-git`). Filet ultime : le `.zip` distribué (aucun git).
- Pour les curieux : le dossier `skills-a-explorer/` contient des copies de skills keiko (create-spec,
  grill-me, tracer, chaîne de production de code, agent de vérification) à parcourir. Ce ne sont pas les
  skills de la séance ; à proposer en bonus à ceux qui finissent en avance.
- Tease S3 : « Avec ce `spec.md` vérifié, on pourra générer un prototype cliquable - ce sera la prochaine
  séance. »
