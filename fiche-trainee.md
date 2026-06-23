# Séance 2 - Faire des specs · Fiche Trainee

## La situation

Tu es PM sur la migration de Ketchup Compta, une appli de comptabilité qu'on fait passer du vieux code
(PHP) vers du neuf. Le tech lead te demande la spec de la page « Toutes les Écritures », pour qu'un dev
puisse la reconstruire sans revenir te voir toutes les cinq minutes.

Tu vas t'y prendre en trois temps : tu écris une première spec à la main, tu la montes en qualité avec
l'outil de l'usine, puis tu la fais vérifier. Tu travailles en binôme, chacun sur sa machine : si un setup coince,
débloquez-vous à deux.

---

## Étape 1 - Une première spec, vite fait (~12 min)

Tu viens de récupérer le ticket. Réflexe naturel : tu ouvres la page et tu demandes à Claude de te la
spécifier.

1. Ouvre l'appli : <http://localhost:8080>, connecte-toi avec `admin` / `admin123`.
2. Va dans le menu Écritures → Toutes les écritures.
3. Dans Claude Code, demande : « Explique-moi ce que fait la page /modules/entries/list.php. »
4. Puis demande : « Écris-moi la spec fonctionnelle de cette page. »
5. Colle sa réponse dans un fichier `spec-v1.md` et garde-le ouvert.

Ne dépasse pas 12 minutes : à ce stade on ne cherche pas la perfection.

<details><summary>💡 Tips</summary>

- Reste sur le fonctionnel : ce que voit et fait l'utilisateur, pas le code.
- Ne complète pas toi-même ce que Claude oublie : on le verra à l'étape suivante.
</details>

---

## Étape 2 - Reprends la spec avec l'outil de l'usine (~18 min)

Cette fois tu pars de l'analyse technique déjà préparée par le tech lead, et tu utilises l'outil qui
t'interroge point par point. C'est le grill-me, une technique popularisée par Matt Pocock : faire remonter
les questions maintenant, au moment de la spec, plutôt que de les subir plus tard pendant le dev.

### 2.1 Récupère le dossier de la séance

Le dojo est un dépôt à part (il s'appuie sur le keiko, mais c'est un autre item). Il est privé : clone-le
avec `gh` (qui t'authentifie automatiquement) :

```bash
gh repo clone Matigon-theodo/dojo-ai-pm-s2 -- --recurse-submodules
cd dojo-ai-pm-s2
```

Si une demande de mot de passe apparaît (ou « authentication failed »), c'est que git n'utilise pas ton
compte GitHub. Corrige-le une fois pour toutes, puis relance la commande de clone :
```bash
gh auth login        # si besoin : choisis GitHub.com, puis HTTPS, puis login via navigateur
gh auth setup-git    # branche git sur ton compte gh
```
Dernier recours si rien ne marche : utilise le dossier `.zip` fourni par ton formateur (aucun git).

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

## Étape 3 - Fais vérifier ta spec (~10 min)

Ta spec te paraît bonne ? C'est le moment de la confier à un regard neuf. Tu lances toi-même un relecteur
indépendant : un agent qui n'a pas participé à ta rédaction, qui relit le vrai code et cherche tes oublis -
exactement ce qu'on veut faire avant de donner la spec à un dev.

1. Lance la relecture : `/spec-review entries-list`
2. Lis son rapport : touchpoints oubliés, cas non couverts, points d'entrée manquants. Chaque remarque
   pointe un fichier précis.
3. Corrige ta `spec.md` avec Claude. Une boucle de correction suffit.

<details><summary>💡 Tips</summary>

- L'agent est là pour trouver des trous : un rapport sévère, c'est utile, pas un échec personnel.
- Compare avec ta `spec-v1.md` : combien de ces trous y étaient déjà ?
</details>

---

## Pour finir

En une phrase : qu'est-ce que l'analyse fournie, l'outil et l'agent de vérification t'ont fait trancher
ou corriger, que tu n'aurais pas vu en écrivant la spec seul ?

---

> ## ✅ Tu as réussi si…
> - tu as une `spec.md` de qualité, avec une section « Précisions issues du grill-me » où tu as tranché des
>   décisions toi-même (dont au moins une décision « hors legacy » mise à part) ;
> - tu as lancé la vérification (le relecteur indépendant) et traité son rapport ;
> - tu peux citer au moins 3 trous comblés depuis ta v1.
