# Points d'entrée — Ketchup Compta (legacy)

> **Ce que c'est.** Pour chaque page, **par où on y arrive** : menu de navigation, redirections après
> une action, boutons « Retour »/« Annuler », liens depuis d'autres écrans.
>
> **Pourquoi un document à part.** Les points d'entrée sont l'**inverse** du call graph de l'analyse :
> le call graph descend (ce qu'une page *appelle*), les points d'entrée remontent (qui *navigue vers*
> la page). Le tracer ne remonte pas les appelants — on obtient ces chemins par **recherche inverse**
> (grep des références à l'URL). Ce fichier est **projet-wide** et sert de source aux specs (`spec.md`),
> qui doivent lister les points d'entrée de leur page.
>
> Format : pour chaque page cible, la liste des `source:ligne` qui y mènent, avec le type de lien.

---

## `page:/dashboard.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `login.php:14` | redirect | Déjà connecté → tableau de bord |
| `login.php:29` | redirect | Après connexion réussie |
| `header.php:42` | menu | « Tableau de bord » |

## `page:/modules/entries/list.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:47` | menu | « Toutes les écritures » |
| `modules/entries/edit.php:38` | redirect | Écriture introuvable (id inexistant) → retour liste avec « Pièce introuvable. » (le post-enregistrement, `edit.php:119`, renvoie vers `edit.php?id=`, PAS vers la liste) |
| `modules/entries/edit.php:189` | bouton | « Retour à la liste » |
| `modules/entries/edit.php:270` | bouton | « Annuler » |

## `page:/modules/entries/edit.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:46` | menu | « Nouvelle écriture » |
| `dashboard.php:65` | lien rapide | Raccourci tableau de bord |
| `modules/entries/list.php:49` | bouton | « Nouvelle écriture » (+ liens « Voir » / n° pièce avec `?id=`) |

## `page:/modules/setup/accounts.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:62` | menu | « Plan comptable » |

## `page:/modules/setup/journals.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:63` | menu | « Journaux » |

## `page:/modules/setup/company.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:61` | menu | « Société » |

## `page:/modules/admin/users.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:64` | menu | « Utilisateurs » |

## `page:/modules/reports/ledger.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:53` | menu | « Grand livre » |
| `dashboard.php:67` | lien rapide | Raccourci tableau de bord |

## `page:/modules/reports/journal.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:55` | menu | « Journal » |

## `page:/modules/reports/trial_balance.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:54` | menu | « Balance » |
| `dashboard.php:66` | lien rapide | Raccourci tableau de bord |

## `page:/login.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `index.php:34` | bouton | « Se connecter » (landing page) |
| `logout.php:12` | redirect | Après déconnexion |

## `page:/logout.php`

| Source | Type | Libellé / contexte |
| ------ | ---- | ------------------ |
| `header.php:34` | lien | « Déconnexion » (barre utilisateur) |

---

> Méthode de (re)génération : `grep -rn` des `href="…"`, `redirect('…')` et `action="…"` pointant vers
> des pages `.php`, regroupés par cible. À rafraîchir si la navigation change.
