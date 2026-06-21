# Analyse legacy : `entries-list`

**Spec globale** : [spec.md](spec.md)

---

## Touchpoint: `page:/modules/entries/list.php`

### Résumé métier

Page de consultation listant **toutes les écritures comptables** déjà passées. Elle affiche un tableau
paginé (30 par page) trié par date décroissante, avec deux filtres (journal et recherche texte sur le
libellé ou le numéro de pièce) et un compteur du total. C'est le point d'entrée vers la consultation
d'une écriture (lien « Voir ») et vers la création d'une nouvelle écriture. La page est en **lecture
seule** : aucune écriture n'y est créée, modifiée ou supprimée.

---

### Comportement frontend

```
Page /modules/entries/list.php  (layout authentifié : header + footer)
├── Titre : "Toutes les Écritures"
├── Bouton : "Nouvelle écriture"            → GET /modules/entries/edit.php   (navigation)
├── Formulaire de filtres                   (method=get, action="")
│   ├── Select "Journal :"                  options = "Tous" + chaque journal actif ({code} - {label})
│   ├── Input texte "Recherche :"           placeholder "Libellé ou numéro"
│   ├── Bouton : "Filtrer"                  → GET /modules/entries/list.php?journal_id=&search=
│   └── Lien : "Reset"                      → GET /modules/entries/list.php
├── SI au moins une écriture :
│   ├── Tableau des écritures
│   │   ├── Colonne : Date                  (jj/mm/aaaa, via format_date)
│   │   ├── Colonne : Journal               (code du journal)
│   │   ├── Colonne : N° Pièce              lien → /modules/entries/edit.php?id={id}   (navigation)
│   │   ├── Colonne : Libellé
│   │   ├── Colonne : Débit                 (montant "0,00", via format_money)
│   │   ├── Colonne : Crédit                (montant "0,00", via format_money)
│   │   ├── Colonne : Créé par              (username du créateur)
│   │   └── Colonne : Actions → "Voir"      → /modules/entries/edit.php?id={id}        (navigation)
│   └── Liens de pagination                 (affichés seulement si total_pages > 1)
├── SINON :
│   └── Texte : "Aucune écriture trouvée."
└── Pied : "Total : {n} écritures"
```

---

### Call graph

```
  /modules/entries/list.php
└─   www/modules/entries/list.php:1-121 [E:1 R:4 X:1]
   ├─   www/footer.php:1-13
   ├─   www/header.php:1-82
   │  ├─ auth_is_logged_in  www/lib/auth.php:80-83
   │  │  └─ auth_start_session  www/lib/auth.php:13-17
   │  ├─ auth_start_session  www/lib/auth.php:13-17 (already seen)
   │  ├─ auth_username  www/lib/auth.php:96-99
   │  │  └─ auth_start_session  www/lib/auth.php:13-17 (already seen)
   │  ├─ _compta_transform_output  www/lib/utils.php:11-30
   │  ├─ get_flash  www/lib/utils.php:46-56
   │  │  └─ auth_start_session  www/lib/auth.php:13-17 (already seen)
   │  └─ h  www/lib/utils.php:125-127
   ├─ require_login  www/lib/auth.php:104-110
   │  ├─ auth_is_logged_in  www/lib/auth.php:80-83 (already seen)
   │  └─ set_flash  www/lib/utils.php:35-41
   │     └─ auth_start_session  www/lib/auth.php:13-17 (already seen)
   ├─ db_escape  www/lib/db.php:213-227
   │  └─ db_connect  www/lib/db.php:87-113
   │     └─ get_db_path  www/lib/db.php:70-80
   │        └─ is_test_mode  www/lib/db.php:57-63
   ├─ db_fetch_all  www/lib/db.php:169-179
   │  └─ SQLiteResult.fetchAll  www/lib/db.php:37-41
   ├─ db_fetch_assoc  www/lib/db.php:152-162
   │  └─ SQLiteResult.fetch  www/lib/db.php:30-35
   ├─ db_query  www/lib/db.php:120-145
   │  └─ db_connect  www/lib/db.php:87-113 (already seen)
   ├─ format_date  www/lib/utils.php:68-73
   ├─ format_money  www/lib/utils.php:61-63
   ├─ get  www/lib/utils.php:139-141 [E:1]
   ├─ get_journals  www/lib/utils.php:214-222 [R:1]
   │  ├─ db_fetch_all  www/lib/db.php:169-179 (already seen)
   │  └─ db_query  www/lib/db.php:120-145 (already seen)
   ├─ h  www/lib/utils.php:125-127 (already seen)
   ├─ paginate  www/lib/utils.php:161-175
   └─ pagination_links  www/lib/utils.php:180-209

26 functions, 462 loc, 8 CFP (E:2 R:5 W:0 X:1)
```

---

### Logique

```
── AUTHENTIFICATION ──
  require_login()
    SI utilisateur non connecté → flash "Veuillez vous connecter." + redirect /login.php (exit)
  (aucun contrôle de rôle. ⚠️ L'application n'a AUCUNE notion de rôle : la table `users` ne contient
   que id, username, password_hash, created_at — pas de colonne `role`, et aucun `require_role` dans le
   code. Tout utilisateur connecté a donc exactement les mêmes droits. Toute idée de permission par rôle
   serait une décision produit NOUVELLE, absente du legacy.)

── LECTURE DES FILTRES ──
  journal_id = get('journal_id', '')        depuis la query string
  search     = get('search', '')            depuis la query string
  page       = max(1, intval(get('page', 1)))

── CONSTRUCTION DE LA CLAUSE WHERE ──
  where = "1=1"
  SI journal_id non vide :
    journal_id = intval(journal_id)         → cast entier (neutralise toute injection)
    where += " AND e.journal_id = {journal_id}"
  SI search non vide :
    search_esc = db_escape(search)          → échappement via PDO::quote
    where += " AND (e.label LIKE '%{search_esc}%' OR e.piece_number LIKE '%{search_esc}%')"

── PAGINATION ──
  total      = SELECT COUNT(*) FROM entries e WHERE {where}
  pagination = paginate(total, page, 30)    page clampée dans [1, total_pages], offset calculé

── REQUÊTE PRINCIPALE ──
  entries = SELECT e.*, j.code, j.label, u.username
            FROM entries e
            LEFT JOIN journals j ON e.journal_id = j.id
            LEFT JOIN users u    ON e.created_by = u.id
            WHERE {where}
            ORDER BY e.entry_date DESC, e.id DESC
            LIMIT {per_page} OFFSET {offset}
  journals = get_journals()                 SELECT * FROM journals WHERE is_active=1 ORDER BY code

── RENDU ──
  SI entries non vide → tableau + liens de pagination
  SINON               → "Aucune écriture trouvée."
  Toujours            → "Total : {total} écritures"
  (aucune persistance, aucun effet de bord)
```

---

### Template graph

    header.php (layout authentifié)                   www/header.php
    ├── (menu de navigation + flash + nom utilisateur)
    Corps de la page (inline dans le script)          www/modules/entries/list.php
    └── footer.php                                     www/footer.php

---

### Request

#### Path Parameters

Aucun.

#### Query Parameters

| Parameter    | Type   | Default | Description                                                        |
| ------------ | ------ | ------- | ------------------------------------------------------------------ |
| `journal_id` | int    | `''`    | Filtre sur un journal (id). Vide = tous les journaux.              |
| `search`     | string | `''`    | Recherche sur le libellé OU le numéro de pièce (LIKE `%...%`).     |
| `page`       | int    | `1`     | Numéro de page (30 écritures/page). Clampé dans `[1, total_pages]`. |

#### Request Body

Aucun (méthode GET uniquement).

---

### Response

Page HTML complète (HTTP 200), rendue côté serveur dans le layout authentifié. Si l'utilisateur n'est
pas connecté : redirection HTTP vers `/login.php` (aucun contenu de liste rendu).

---

### Base de données

#### Tables utilisées

| Table      | Opération | Description                                                       |
| ---------- | --------- | ----------------------------------------------------------------- |
| `entries`  | SELECT    | Écritures comptables : comptage (pagination) + liste paginée.     |
| `journals` | SELECT    | Jointure pour le code/libellé du journal + alimentation du filtre. |
| `users`    | SELECT    | Jointure pour le nom de l'utilisateur créateur.                   |

#### Champs utilisés

| Champ                | Opération | Description                                          |
| -------------------- | --------- | ---------------------------------------------------- |
| `entries.id`         | READ      | Identifiant, utilisé dans les liens « Voir ».        |
| `entries.entry_date` | READ      | Date affichée + tri principal (DESC).                |
| `entries.journal_id` | READ      | Filtre + jointure journal.                           |
| `entries.piece_number` | READ    | N° de pièce affiché + cible de recherche.            |
| `entries.label`      | READ      | Libellé affiché + cible de recherche.                |
| `entries.total_debit` | READ     | Montant débit affiché.                               |
| `entries.total_credit` | READ    | Montant crédit affiché.                              |
| `entries.created_by` | READ      | Jointure utilisateur.                                |
| `journals.id`        | READ      | Clé de jointure.                                     |
| `journals.code`      | READ      | Code affiché + tri du filtre.                        |
| `journals.label`     | READ      | Libellé affiché dans le filtre.                      |
| `journals.is_active` | READ      | Seuls les journaux actifs alimentent le filtre.      |
| `users.username`     | READ      | Nom du créateur affiché.                             |

---

### Codes d'erreur

Aucun code d'erreur applicatif explicite (page de lecture). Comportements de bord :

| Condition                          | Comportement                                              |
| ---------------------------------- | -------------------------------------------------------- |
| Utilisateur non connecté           | Flash « Veuillez vous connecter. » + redirect `/login.php` |
| `journal_id` non numérique         | `intval` → `0` → aucune écriture ne correspond (liste vide) |
| Recherche sans résultat            | « Aucune écriture trouvée. »                             |
| `page` hors bornes                 | Ramenée dans `[1, total_pages]` par `paginate`           |

---

### Autres effets de bord externes

Aucun (pas d'appel API externe, pas d'email, pas d'écriture fichier, pas de modification en base).
