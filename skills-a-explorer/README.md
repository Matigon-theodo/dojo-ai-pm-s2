# Skills à explorer (pour les curieux)

Ces skills sont des **copies de référence** issues du keiko (le tuto de modernisation). Ils sont là pour
que tu **voies comment c'est fait** — pas pour les lancer pendant la séance. La S2 n'utilise qu'un seul
skill : `/spec-grill-session` (à la racine, dans `.claude/skills/`).

> ⚠️ Ce sont des copies pour **lecture**. Certains référencent de l'outillage keiko (tracer, agents…) qui
> n'est pas embarqué ici : ne t'attends pas à ce qu'ils tournent tels quels dans ce dépôt.

Ouvre le `SKILL.md` de chacun. Ils suivent la **chaîne de modernisation**, de la spec au code livré :

## 1. Spécifier (là où tu interviens en tant que PM)

- **create-spec/** — le skill complet dont `spec-grill-session` est la version séance : il analyse un
  *touchpoint* legacy et produit `spec.md` (fonctionnel) + `analysis.md` (technique). Contient aussi :
  - `page-discovery.md` — **c'est là qu'est documenté `agent-browser`** (l'outil qui ouvre l'app, se
    connecte et capture les écrans/snapshots).
  - `template-spec.md` / `template-analysis.md` — les gabarits des deux livrables.
- **grill-me/** — le skill générique « cuisine-moi » : interviewer une décision/un plan question par
  question. C'est le moteur du grill-me que tu as vécu en séance.

## 2. Analyser le legacy / tracer le code

- **cosmic-tracing/** — la référence du *call-graph tracing* COSMIC (comment on construit le graphe
  d'appels et on compte les mouvements de données).
- **cosmic-tracing-php/** — le supplément PHP (c'est ce qui s'applique à Ketchup Compta).

> `bin/tracer` (l'outil qui produit le graphe) n'est pas embarqué dans le dojo : on t'a déjà fourni le
> résultat dans `docs/features/entries-list/analysis.md`.

## 3. Planifier la migration

- **create-migration-plan/** — transformer la spec + l'analyse en plan de migration (mapping
  legacy → nouvelle archi).
- **create-tickets/** — découper le plan en tickets indépendants (tranches verticales).

## 4. Produire le code

- **prime-context/** — charger les standards/conventions du projet avant de coder.
- **implement-ticket/** — implémenter un ticket dans son périmètre strict.
- **validate/** — formater, linter, tester en parallèle jusqu'à ce que tout passe au vert.
- **commit/** — créer des commits atomiques (Conventional Commits), ordonnés par couche de dépendance.
- **open-pr/** — ouvrir une PR avec description liée au plan.

---

Curieux d'aller plus loin ? Le keiko complet contient bien d'autres skills (découverte des touchpoints,
PRD, ADR, revue de code, etc.). Demande à ton tech lead pour y accéder.
