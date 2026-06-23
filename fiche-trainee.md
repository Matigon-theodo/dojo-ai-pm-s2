# Séance 2 - Faire des specs · Fiche Trainee

## La situation

Tu es PM sur la migration de Ketchup Compta, une appli de comptabilité qu'on fait passer du vieux PHP vers
du neuf. Le tech lead débarque à ton bureau : il lui faut la spec de la page « Toutes les Écritures »,
assez précise pour qu'un dev la reconstruise sans revenir te voir toutes les cinq minutes.

Tu vas y aller en trois temps : d'abord une spec écrite au feeling, puis la même passée à l'outil de
l'usine, et enfin une relecture qui traque tes oublis. En binôme, chacun sa machine : si un setup coince,
débloquez-vous à deux.

---

## Étape 1 - Le réflexe (~10 min)

Le ticket vient de tomber. Ton premier réflexe, comme tout le monde : ouvrir la page, regarder ce qu'elle
fait, et demander à Claude de t'en écrire la spec. Vas-y, sans filet.

1. Ouvre l'appli : <http://localhost:8080>, connecte-toi avec `admin` / `admin123`.
2. Va dans le menu Écritures → Toutes les écritures.
3. La page vit dans le fichier `/modules/entries/list.php`. Avant de spécifier quoi que ce soit, tu veux
   comprendre ce qu'elle fait vraiment : à toi de trouver la bonne question à poser à Claude Code.
4. Une fois les idées claires, c'est la spec fonctionnelle de la page que tu veux. Demande-la, à ta façon.
5. Colle sa réponse dans un fichier `spec-v1.md` et garde-le ouvert.

Ne dépasse pas 10 minutes : à ce stade on ne cherche pas la perfection, on cherche ton point de départ.

<details><summary>💡 Tips</summary>

- Reste sur le fonctionnel : ce que voit et fait l'utilisateur, pas le code.
- Ne complète pas toi-même ce que Claude oublie : on le verra à l'étape suivante.
</details>

---

## Étape 2 - L'usine entre en jeu (~20 min)

Rejouer la même chose à la main ne servirait à rien. Cette fois tu changes de méthode : tu pars de
l'analyse technique que le tech lead a déjà préparée, et tu laisses l'outil de l'usine t'interroger point
par point. C'est le grill-me, une technique popularisée par Matt Pocock : faire sortir les questions
maintenant, pendant la spec, plutôt que de les subir plus tard en plein dev.

### 2.1 Récupère le dossier de la séance

Pour utiliser l'usine IA standard de Theodo, on travaille dans un dépôt dédié : le dojo s'appuie sur le
keiko, mais c'est un autre item. Clone-le et entre dedans. S'il te demande un identifiant ou un mot de
passe, ne le remplis pas : c'est que le dépôt est privé → déplie le premier toggle ci-dessous.

```bash
git clone --recurse-submodules https://github.com/Matigon-theodo/dojo-ai-pm-s2.git
cd dojo-ai-pm-s2
```

<details><summary>🔑 Git me demande un identifiant ou un mot de passe ?</summary>

Le dépôt est privé : Git a besoin que tu sois authentifié. GitHub n'accepte plus le mot de passe de
compte, donc on passe par le GitHub CLI (`gh`), qui gère ça sans manipuler de mot de passe.

1. Installe `gh`, sans Homebrew :
   - Sur Mac : télécharge l'installeur `.pkg` macOS depuis cli.github.com (ou la page Releases
     `github.com/cli/cli/releases`), puis double-clique.
   - Sur Windows : dans PowerShell, lance `winget install --id GitHub.cli --source winget`.
2. Authentifie-toi (quel que soit l'OS) :
   - `gh auth login`, puis choisis GitHub.com, HTTPS, et Login with a web browser.
   - `gh auth setup-git` pour brancher Git sur ces identifiants.
3. Relance le clone. Il doit passer cette fois.

Tu veux quand même installer `gh` via Homebrew ? Déplie le toggle suivant. Toujours bloqué après
authentification ? Ce n'est plus l'authentification mais l'accès : demande au propriétaire du dépôt
`Matigon-theodo` de t'ajouter en lecture.
</details>

<details><summary>🍺 Installer Homebrew (dernier recours)</summary>

À ne faire que si tu veux installer `gh` via `brew` et que tu n'as pas Homebrew. C'est l'option la plus
lourde, et l'installeur réclame ton mot de passe Mac en cours de route (Claude Code ne peut pas le saisir
à ta place).

1. Installe Homebrew dans le Terminal :
   `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
   Appuie sur Entrée pour confirmer, puis tape ton mot de passe Mac (il reste invisible à l'écran, c'est
   normal).
2. Ajoute `brew` au PATH (Mac Apple Silicon, M1 à M4), trois commandes :
   - `echo >> ~/.zprofile`
   - `echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile`
   - `eval "$(/opt/homebrew/bin/brew shellenv)"`
   Sur Mac Intel, recopie plutôt la ligne exacte affichée par l'installeur à la fin (le chemin est
   `/usr/local` au lieu de `/opt/homebrew`).
3. Installe `gh` avec `brew install gh`, puis reviens au toggle précédent pour l'authentification.
</details>

Garde ta `spec-v1.md` de l'étape 1 sous la main pour la comparer en fin de séance (tu peux la copier ici,
en adaptant le chemin) :

```bash
cp <chemin>/spec-v1.md .
```

Vérifie que tu as bien le dossier pré-mâché (tu dois voir `analysis.md` et le dossier `browser/`) :

```bash
ls docs/features/entries-list/
```

### 2.2 Lance le grill-me

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

---

## Étape 3 - Le contrôle qualité (~10 min)

Avant de lâcher ta spec dans la nature, tu la fais relire. Pas par toi, mais par un agent indépendant qui
n'a pas participé à ta rédaction : il va lire le vrai code legacy et débusquer ce que tu as oublié. C'est
toi qui le déclenches, c'est un geste à part entière.

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
> - tu as lancé toi-même la relecture (`/spec-review`) et traité son rapport ;
> - tu peux citer au moins 3 trous comblés depuis ta v1.
