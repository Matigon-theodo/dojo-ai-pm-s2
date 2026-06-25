# Session 2 - Trainee - Faire des specs avec Claude

## La situation

- Tu es PM sur la migration de Ketchup Compta, une appli de comptabilité qu'on fait passer du vieux PHP
  vers du neuf. Le tech lead a besoin de la spec de la page « Toutes les Écritures », assez précise pour
  qu'un dev puisse la prendre en dev directement.
- Tu vas y aller en trois temps : d'abord une spec écrite au feeling, puis la même passée à l'outil de
  l'usine, et enfin une relecture qui traque tes oublis.
- En binôme, chacun sa machine : si un setup coince, débloquez-vous à deux.

---

## Étape 1 - Spec écrite avec Claude seulement (~10 min)

> 📁 **Où tu travailles :** dans **ton projet `keiko-ai-mod`**, celui des séances 0 et 1. Lance Claude
> Code dedans, exactement comme en S1. Rien de nouveau à télécharger pour cette étape.

1. Ouvre l'appli : <http://localhost:8080>, connecte-toi avec `admin` / `admin123`.
2. Va dans le menu Écritures → Toutes les écritures.
3. Le code de la page vit dans `legacy/www/modules/entries/list.php`.

Dans cette première version, écris directement la spec avec Claude en te basant sur le code,
l'interface de l'app et ta compréhension. Vas-y, sans filet.

→ Colle ta réponse dans un fichier `spec-v1.md` (à la racine de `keiko-ai-mod`) et garde-le ouvert.

Ne dépasse pas 10 minutes : à ce stade on ne cherche pas la perfection, on cherche ton point de départ.

<details><summary>💡 Tips</summary>

- Reste sur le fonctionnel : ce que voit et fait l'utilisateur, pas le code.
</details>

Garde ta `spec-v1.md` sous la main : tu la compareras à la version « outillée » à la fin de l'étape 2.

---

## Étape 2 - L'usine entre en jeu (~15 min)

Maintenant on va se servir non seulement de Claude mais aussi d'un SKILL issu d'une usine agentique
éprouvée pour réaliser la spec.

> 📁 **Deux dossiers, ne les confonds pas.** À partir d'ici, tu changes d'endroit où tu travailles :
> - **`keiko-ai-mod`** = *ton projet* (séances 0 et 1). C'est là que tourne l'app et que tu viens
>   d'écrire ta `spec-v1.md`. Tu t'en es servi à l'étape 1.
> - **`dojo-ai-pm-s2`** = *le kit de la séance*, que tu télécharges maintenant. Il contient l'analyse
>   technique déjà faite par ton tech lead (`analysis.md`) et l'outil qui va t'aider à écrire la spec.
>   **C'est ici que tu travailles à partir de l'étape 2.**
>
> Pourquoi un dossier à part plutôt que ton projet ? Pour que tout le monde démarre du **même
> environnement contrôlé** : la même analyse, le même outil, la même version du code. Ton projet
> `keiko-ai-mod` reste intact de son côté.

Cette fois, le tech lead a déjà fait une analyse technique préalable qu'il a résumée dans le fichier
`analysis.md`.

### 2.1 Télécharge le kit de la séance

Le kit vit dans un dépôt dédié, `dojo-ai-pm-s2`. Clone-le **à côté de ton projet** (par exemple dans le
même dossier parent que `keiko-ai-mod`) et entre dedans :

```bash
git clone --recurse-submodules https://github.com/Matigon-theodo/dojo-ai-pm-s2.git
cd dojo-ai-pm-s2
```

Le dépôt est public : le clone doit passer sans rien te demander. Si jamais Git réclame quand même un
identifiant ou un mot de passe (selon la config de ta machine), ne le remplis pas et déplie le premier
toggle ci-dessous.

<details><summary>🔑 Git me demande un identifiant ou un mot de passe ?</summary>

Le dépôt est public, donc en principe ça ne devrait pas arriver. Si ça arrive quand même, c'est que ta
machine est configurée pour passer par une authentification (par ex. réécriture des URLs en SSH). GitHub
n'accepte plus le mot de passe de compte, donc on passe par le GitHub CLI (`gh`), qui gère ça sans
manipuler de mot de passe.

**1. Installe `gh`, sans Homebrew**

- Sur Mac : télécharge l'installeur `.pkg` macOS depuis cli.github.com (ou la page Releases
  `github.com/cli/cli/releases`), puis double-clique.
- Sur Windows : dans PowerShell, lance `winget install --id GitHub.cli --source winget`.

**2. Authentifie-toi** (quel que soit l'OS)

- `gh auth login` puis choisis GitHub.com, HTTPS, et Login with a web browser.
- `gh auth setup-git` pour brancher Git sur ces identifiants.

**3. Relance le clone.** Il doit passer cette fois.

Tu veux quand même installer `gh` via Homebrew ? Déplie le toggle suivant. Toujours bloqué après
authentification ? Ce n'est plus l'authentification mais l'accès : demande à l'admin de l'orga
`Matigon-theodo` de t'ajouter au dépôt.
</details>

<details><summary>🍺 Installer Homebrew (dernier recours)</summary>

À ne faire que si tu veux installer `gh` via `brew` et que tu n'as pas Homebrew. C'est l'option la plus
lourde, et l'installeur réclame ton mot de passe Mac en cours de route (Claude Code ne peut pas le saisir
à ta place).

**1. Installe Homebrew** dans le Terminal :
`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

Appuie sur Entrée pour confirmer, puis tape ton mot de passe Mac (il reste invisible à l'écran, c'est
normal).

**2. Ajoute `brew` au PATH** (Mac Apple Silicon, M1 à M4), trois commandes :

- `echo >> ~/.zprofile`
- `echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile`
- `eval "$(/opt/homebrew/bin/brew shellenv)"`

Sur Mac Intel, recopie plutôt la ligne exacte affichée par l'installeur à la fin (le chemin est
`/usr/local` au lieu de `/opt/homebrew`).

**3. Installe `gh`** avec `brew install gh`, puis reviens au toggle précédent pour l'authentification.
</details>

Ça y est ! Tu viens de faire une copie d'une version allégée de l'usine de modernisation IA de Theodo.
Deux réflexes maintenant que tu as changé de dossier :

1. **Lance Claude Code depuis `dojo-ai-pm-s2`** (et non plus depuis `keiko-ai-mod`). C'est le seul dossier
   où tu travailles jusqu'à la fin de la séance.
2. **Récupère ta `spec-v1.md`** ici, pour pouvoir la comparer à la version outillée en 2.5 :

   ```bash
   cp ../keiko-ai-mod/spec-v1.md .   # adapte le chemin si tu l'as rangée ailleurs
   ```

### 2.2 Explorer l'usine que tu viens de cloner !

- Ouvre le dossier dans VSCode pour l'explorer.
- Dans `skills-a-explorer/` tu as accès à des skills qu'on n'utilisera pas dans cette session mais qui
  sont utilisés tous les jours dans nos usines agentiques Theodo par nos PMs et nos techs.
- Le fichier `docs/features/entries-list/analysis.md` est le résultat de l'analyse technique de ton tech
  lead sur la page entries-list.

Vérifie que tu as bien le dossier pré-mâché (tu dois voir `analysis.md` et le dossier `browser/`) :

```bash
ls docs/features/entries-list/
```

### 2.3 Comprendre le skill create-spec et le grill-me

**Concrètement, le skill `spec-grill-session`, il fait quoi ?**

Vois-le comme un analyste fonctionnel qui a déjà fait ses devoirs. Avant de t'adresser la parole, il a lu
l'intégralité de l'analyse technique du tech lead (`analysis.md`) et regardé la capture de la page. Il
sait donc déjà ce que la page fait réellement. Son rôle ensuite, c'est de te faire produire une spec
carrée. Il s'appuie sur deux choses.

**1. Un entretien dirigé (le grill-me).** Il t'interroge, mais de façon cadrée :

- une question à la fois, jamais un mur. Tu réponds, il enchaîne.
- avec une réponse recommandée à chaque fois, pour que tu n'aies jamais à partir d'une page blanche. Tu
  valides, tu ajustes, ou tu tranches autrement.
- uniquement des questions métier (qui voit quoi, ce qui s'affiche quand la liste est vide, etc.), jamais
  de technique.
- et il ne te demande pas ce qu'il peut vérifier lui-même. Si la réponse est dans l'analyse, il va la
  chercher au lieu de te la poser.

C'est toi qui possèdes chaque décision. Lui ne fait que transcrire.

**2. Un template imposé.** Il ne rédige pas la spec en texte libre. Il coule tes décisions dans un format
prédéterminé, toujours le même, qui garantit qu'aucune rubrique n'est oubliée :

- **Pourquoi** : l'objectif de la page (avec un schéma du parcours).
- **Cas d'utilisation** : ce que l'utilisateur voit et fait, cas d'erreur compris.
- **Précisions issues du grill-me** : les décisions que tu as tranchées pendant l'entretien.
- **Touchpoints** : un tableau de tout ce qui part de la page, marqué « dans le périmètre » ✅ ou
  « simple navigation » ❌.
- **Scénarios de validation** : les cas concrets qu'un testeur (ou un agent) suivra pour vérifier,
  couverture complète exigée.

Ce format normé rend aussi les specs comparables et homogènes d'une feature à l'autre dans l'usine.

> En résumé : il a fait ses devoirs sur le code, il te cuisine pour combler les trous, puis il restitue
> tes réponses dans un format normé. Pas un texte au feeling, mais une spec structurée que n'importe quel
> dev peut suivre.

### 2.4 Crée ta spec avec `/spec-grill-session entries-list`

1. Dans Claude Code, lance : `/spec-grill-session entries-list`
2. Réponds aux questions une par une. À chaque question, Claude te propose une réponse : valide-la,
   ajuste-la, ou tranche autrement.
3. Quand c'est fini, ouvre `docs/features/entries-list/spec.md` et relis-le.

<details><summary>💡 Tips</summary>

- Tu es le commanditaire (Product Owner) : réponds en métier, pas en technique.
- Si Claude te propose une règle « selon le rôle de l'utilisateur » (par ex. cacher un bouton à certains
  profils), vérifie dans `analysis.md` : l'appli ne gère aucun rôle aujourd'hui. Soit tu gardes le
  comportement actuel (ouvert à tous), soit tu notes « ajouter des rôles » comme une décision à part,
  séparée de la migration. Ne l'écris pas comme si ça existait déjà.
- Pour savoir d'où Claude tient une réponse, demande-lui « sur quoi tu te bases ? ».
</details>

### 2.5 Mesure l'écart avec ta v1 (2-3 min)

Tu as maintenant deux versions sous la main : `spec-v1.md`, écrite seul au feeling à l'étape 1, et
`spec.md`, sortie du grill-me. La valeur du skill se voit en les mettant côte à côte. Plutôt que de la
deviner, fais-la sortir par Claude.

1. Dans Claude Code, demande la comparaison :

   > Compare `spec-v1.md` (que j'ai écrite seul) et `spec.md` (sortie du grill-me). Fais-moi un tableau de
   > ce que la v2 couvre et que la v1 ratait : cas d'erreur, états vides, périmètre, points d'entrée,
   > décisions tranchées.

2. Lis le tableau. Chaque ligne est un trou que tu n'aurais pas comblé seul.
3. Pour chaque ligne, pose-toi une question : si un dev avait découvert ce trou en plein build, qu'est-ce
   que ça aurait coûté ?

C'est ça, la vraie valeur du grill-me. Pas une spec plus longue, mais une spec où chaque cas a été tranché
à l'écrit, au moment où ça ne coûte presque rien. Chaque ligne du tableau, c'est un aller-retour avec le
dev, une recette ratée ou une feature à refaire, évités parce que la question a été posée maintenant.

---

## Étape 3 - Le contrôle qualité (~10 min)

Avant de lâcher ta spec dans la nature, tu la fais relire. Pas par toi, mais par un agent `/spec-review`
qui, lui, va lire le vrai code legacy et débusquer ce que tu as oublié.

### 3.1 Pourquoi faire relire ta spec par un agent indépendant ?

On ne se relit jamais bien soi-même. Tu as écrit cette spec, donc tu as les mêmes angles morts à la
relecture qu'à la rédaction. L'agent de review, lui, arrive avec un regard neuf et un seul job : trouver
tes trous.

Deux choses le rendent utile :

- **Il n'a pas participé à ta rédaction.** Aucun biais, aucun attachement à tes choix. Un rapport sévère
  n'est pas un échec, c'est de la valeur trouvée avant que ça coûte cher.
- **Il relit le vrai code, pas ta prose.** Il confronte ta spec au code legacy et à l'analyse technique,
  pas à ta mémoire. C'est ce qui rend son avis crédible.

Le grill-me a comblé les trous que tu savais devoir poser. La review attrape ceux que tu ne savais même
pas avoir. C'est le contrôle qu'on veut faire passer à une spec avant de la confier à un dev, pendant que
corriger ne coûte encore qu'une ligne de texte.

### 3.2 Lance ton agent de review

1. Lance la relecture : `/spec-review entries-list`
2. Lis son rapport : touchpoints oubliés, cas non couverts, points d'entrée manquants. Chaque remarque
   pointe un fichier précis.
3. Corrige ta `spec.md` avec Claude. Une boucle de correction suffit.

<details><summary>💡 Tips</summary>

- L'agent est là pour trouver des trous : un rapport sévère, c'est utile, pas un échec personnel.
- Compare avec ta `spec-v1.md` : combien de ces trous y étaient déjà ?
</details>

---

## Key takeaways (en groupe)

En groupe, faites ressortir ensemble ce que vous retenez. En une phrase : qu'est-ce que l'analyse fournie,
l'outil et l'agent de vérification t'ont fait trancher ou corriger, que tu n'aurais pas vu en écrivant la
spec seul ?

> ## ✅ Tu as réussi si…
> - tu as une `spec.md` de qualité, avec une section « Précisions issues du grill-me » où tu as tranché
>   des décisions toi-même (dont au moins une décision « hors legacy » mise à part) ;
> - tu as lancé la vérification (le relecteur indépendant) et traité son rapport ;
> - tu peux citer au moins 3 trous comblés depuis ta v1.
