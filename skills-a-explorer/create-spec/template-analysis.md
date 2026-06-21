# Analyse legacy : `<feature>`

**Spec globale** : [spec.md](spec.md)

[INSTRUCTIONS: Un seul fichier `analysis.md` regroupe TOUS les touchpoints in-scope.
Pour chaque touchpoint in-scope, ajouter une section `## Touchpoint: <id>` puis les sous-sections ci-dessous.
Les sections marquées [PAGE uniquement] sont omises pour les touchpoints non-PAGE.
Les sections marquées [PAGE/ENDPOINT uniquement] sont omises pour command/listener/database.]

---

## Touchpoint: `[node-id]`

### Résumé métier

[2-3 phrases expliquant la logique métier]

---

### Comportement frontend [PAGE uniquement]

[Omettre cette section pour les touchpoints non-PAGE.
Arbre simplifié de la page à partir du snapshot déjà capturé.
Règles :
- Garder uniquement la structure : sections, formulaires, tableaux, boutons, liens
- Pour les tableaux : lister chaque colonne avec son format d'affichage (ex. `dd/mm/yyyy`, `0,00 €`)
- Pour les CTA (boutons/liens) : noter l'URL ou action cible (ex. `→ POST /order/scancontrol`)
- Pour les compteurs dynamiques : utiliser `{n}` (ex. `"{n} Commandes sans scans"`)
- Supprimer éléments décoratifs, icônes, classes CSS, attributs HTML
- Notation arbre ASCII (`├──`, `└──`, `│`)]

```
Page /order/scancontrol
├── Colonne 1 : "Liste des bons de commandes à traiter"
│   ├── Tableau fichiers scannés
│   │   ├── Colonne : Fichier (nom image, ex. "20_20.jpg")
│   │   ├── Colonne : Déjà scanné (oui/non)
│   │   ├── CTA : "A intégrer" → GET /order/scancontrol/dl_fichier={filename}
│   │   └── CTA : "Détacher" → lien par fichier
│   ├── Bouton : "Intégrer" → POST /order/scancontrol
│   ├── Lien : "0 scan(s) à corriger" → /order/scancontrol/mode=correct
│   └── Lien : "Audit" → /order/scancontrol/mode=audit
├── Colonne 2 : "{n} Commandes sans scans"
│   ├── Filtres marché
│   │   ├── Checkboxes : Allemagne, Belgique FL, Belgique WA, France, Pays-Bas
│   │   └── Bouton : "Filtrer"
│   └── Tableau commandes
│       ├── Colonne : Numéro (ex. "V0033017")
│       ├── Colonne : Client
│       ├── Colonne : Patient
│       ├── Colonne : Date de saisie (dd/mm/yyyy)
│       └── Colonne : Etat
└── Colonne 3 : "{n} Scans sans commandes"
    ├── Bouton : "Purger" → suppression des scans orphelins
    ├── Tableau scans orphelins
    │   ├── Colonne : Numéro de commande
    │   ├── Colonne : Date d'intégration (dd/mm/yyyy)
    │   └── CTA : "Existant" → GET /order/scancontrol/order={orderID}
    └── Section : "Historique Purge"
```

---

### Call graph

[Section déjà générée dans le fichier via `>> analysis.md`. Ne pas réécrire.]

```
[Sortie de `bin/tracer tree` — ne pas réécrire, déjà injecté via shell redirect]
```

---

### Logique

[Pseudocode concis décrivant la validation, le traitement et les effets de bord.
Pour les gros endpoints, utiliser des en-têtes `── SECTION ──` pour regrouper la logique.]

```
── VALIDATION ──
  entity EXISTS                    → 404 ENTITY_NOT_FOUND
  entity.state = ACTIVE            → 400 INVALID_STATE
  user.canWrite(entity)            → 403 FORBIDDEN

── TRAITEMENT ──
  multiplier = config.get() ?? 1.0
  price = base × multiplier × (1 + fee)
  IF price > maxPrice THEN price = maxPrice
  price = round(price, 2)

── PERSISTANCE ──
  entity.price = price
  entity.updatedAt = now()
  save(entity)

── APPELS EXTERNES ──
  orderAPI.submit(entity)          → 502 EXTERNAL_API_ERROR (rollback)

── EFFETS DE BORD ──
  publish(ENTITY_UPDATED)
  s3.audit(entity)
```

---

### Template graph [PAGE uniquement]

[Omettre cette section pour les touchpoints non-PAGE.
Arbre des templates rendus depuis le point d'entrée du controller.
Deux colonnes : nom logique + chemin fichier.]

    Layout                                            layout.ext
    ├── Filtres                                       filters.ext
    │   ├── FiltreDate                                date-filter.ext
    │   ├── FiltreIntervalle                          range-filter.ext
    ├── Liste des éléments                            list.ext
    │   └── Ligne                                     list-item.ext
    └── Bouton valider                                submit.ext

---

### Request [PAGE/ENDPOINT uniquement]

#### Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `id`      | long | Entity ID   |

#### Query Parameters

| Parameter | Type   | Default | Description |
| --------- | ------ | ------- | ----------- |
| `filter`  | string | `null`  | Optional    |

#### Request Body

[Table des champs du body si applicable.]

| Champ | Type | Description |
|-------|------|-------------|
| `fieldA` | string | Description |

---

### Response

[Décrire le type de réponse réel : rendu HTML, JSON, texte brut, fichier binaire, redirection, etc.]

---

### Base de données

#### Tables utilisées

[Uniquement les tables utilisées par cet endpoint]

| Table    | Opération      | Description          |
| -------- | -------------- | -------------------- |
| `entity` | SELECT, UPDATE | Table entité principale |

#### Champs utilisés

[Uniquement les champs utilisés par cet endpoint]

| Champ          | Opération    | Description          |
| -------------- | ------------ | -------------------- |
| `entity.field` | READ, UPDATE | Champ entité principal |

---

### Codes d'erreur

| Code                 | Déclencheur                          |
| -------------------- | ------------------------------------ |
| `ENTITY_NOT_FOUND`   | Aucune entité avec cet ID           |
| `INVALID_STATE`      | L'état de l'entité n'est pas ACTIVE  |
| `EXTERNAL_API_ERROR` | L'API externe a retourné non-200     |

---

### Autres effets de bord externes

[Appels API externes, envoi d'emails, écriture fichiers, etc.
Omettre cette section si aucun.]

---

## Touchpoint: `[node-id suivant]`

[Répéter la structure ci-dessus pour chaque touchpoint in-scope]
