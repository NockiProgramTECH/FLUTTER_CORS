**Navigation :** [← Module 14](./14-relations.md) · [Sommaire](../README.md) · [Module 16 →](./16-requetes-filtres.md)

# Module 15 — Insérer, lire, modifier et supprimer des données

## 1. Objectif pédagogique

Vous maîtriserez le sens des opérations CRUD Drift, leurs valeurs de retour et leur comportement asynchrone, et distinguerez insert, remplacement, mise à jour et upsert.

## 2. Le problème à résoudre

Une liste de tâches doit créer une ligne, la relire, changer un statut et éventuellement supprimer une ligne sans effacer accidentellement toute la table.

## 3. Explication du fonctionnement

CRUD signifie **Create, Read, Update, Delete**. Une insertion se fait par `into(table).insert(companion)` et retourne généralement le `rowid`/identifiant inséré. Le **companion** est un objet partiel : il distingue une colonne absente d’une colonne explicitement mise à `null`. Une lecture `get()` exécute une requête ponctuelle et retourne un `Future<List<Row>>`. Une mise à jour par `write` retourne le nombre de lignes affectées. Une suppression par `go()` retourne également le nombre de lignes touchées.

Les fonctions sont asynchrones : utiliser `await` dans une fonction `async`, gérer erreurs et chargement. Attention : un `UPDATE` ou `DELETE` sans `where` peut toucher toutes les lignes.

`insertAll` dans un `batch` est adapté aux insertions en volume et réduit le coût de préparation/répétition de requêtes. L’insertion normale échoue sur un conflit de clé. `insertOnConflictUpdate` ou `onConflict: DoUpdate(...)` fait un **upsert** (insère ou met à jour au conflit); sa cible de conflit doit correspondre à une clé primaire/unique. `update(table).replace(row)` remplace les champs d’une ligne identifiée par sa clé primaire via UPDATE. Ce n’est pas la même opération que `INSERT OR REPLACE` SQLite, qui peut supprimer/réinsérer la ligne et affecter références ou déclencheurs. Choisissez la sémantique consciemment.

## 4. Petit extrait de code

```dart
Future<int> addTask(String title) =>
    into(tasks).insert(TasksCompanion.insert(title: title));
Future<List<Task>> readTasks() => select(tasks).get();
Future<int> markDone(int id) => (update(tasks)
  ..where((t) => t.id.equals(id)))
  .write(const TasksCompanion(done: Value(true)));
Future<int> removeTask(int id) => (delete(tasks)
  ..where((t) => t.id.equals(id))).go();
```

## 5. Explication ligne par ligne

- `addTask` insère un companion. Ici, l’exemple suppose que seul `title` est obligatoire; la clé auto-incrémentée et `done` par défaut sont omises.
- `readTasks` exécute un `SELECT` une fois. Le `Future` n’est pas encore une liste synchronisée après son achèvement.
- `markDone` construit une mise à jour limitée à l’identifiant demandé. Le companion partiel ne modifie que `done`; `Value(true)` signifie que le champ est présent et doit être mis à jour.
- `removeTask` cible la ligne par identifiant. Le nombre retourné permet de savoir combien de lignes ont été supprimées.
- Les appels sont asynchrones; la couche qui les utilise doit `await` et représenter les erreurs possibles.

Pour plusieurs lignes, `batch((b) { b.insertAll(tasks, companions); })` regroupe l’opération. Pour un enregistrement complet, `update(tasks).replace(row)` cible la clé primaire du `row`. Pour créer ou modifier selon un conflit, utilisez l’API d’upsert, pas une séquence naïve « lire puis insérer » vulnérable à deux écritures concurrentes.

## 6. Déroulement interne

Drift convertit le companion en colonnes et paramètres SQL, puis SQLite applique valeurs par défaut, clés et contraintes. L’opération retourne un résultat asynchrone après l’exécution. Un échec (contrainte violée, connexion fermée, problème d’exécuteur) se propage sous forme d’erreur; l’interface peut alors afficher le bon état au lieu de supposer que l’écriture a réussi.

## 7. Pourquoi cette approche ?

Les companions rendent explicite la différence entre champ absent et champ mis à `null`, ce qui est crucial pour une mise à jour partielle. La génération des lignes réduit les conversions manuelles. Les opérations groupées accélèrent les gros lots, mais ne remplacent pas toujours une transaction métier (module 18).

## 8. Erreur classique

**Erreur :** appeler `delete(tasks).go()` en pensant supprimer seulement la ligne visible, ou utiliser une insertion normale comme si elle mettait à jour. **Correction :** ajouter et tester le filtre `where`; choisir entre insertion, update par clé et upsert selon le résultat métier voulu.

## 9. Questions de compréhension

1. Quel est le sens différent de `Value.absent()` et `Value(null)` lors d’une mise à jour?
2. Quel résultat produit `get()` et pourquoi n’est-il pas une source qui se réactualise?
3. Pourquoi un upsert évite-t-il une course possible entre lecture et insertion?
4. Que peut provoquer `DELETE` sans `where`?

## 10. Petit exercice pratique

- **Objectif :** écrire le plan CRUD d’une tâche.
- **Situation de départ :** table `Tasks` avec `id` auto-incrémenté, `title` requis et `done` par défaut à faux.
- **Résultat attendu :** signatures pour ajouter, lire, marquer terminée et supprimer par identifiant.
- **Consignes :** indiquez les types de retour asynchrones et ajoutez un filtre d’identifiant aux deux opérations destructives/modificatrices.
- **Critères de réussite :** insertion distincte de l’upsert, `where` présent pour UPDATE/DELETE.
- **Indice facultatif :** les méthodes de requête sont exécutées et retournent des `Future`.

## 11. Résumé essentiel

- `insert`, `get`, `write` et `go` réalisent respectivement création, lecture ponctuelle, mise à jour et suppression.
- Les companions représentent des écritures partielles et distinguent absent de `null`.
- `insertAll` en batch est utile pour les lots; upsert traite un conflit; update/replace modifie une ligne ciblée.
- Un `UPDATE`/`DELETE` sans filtre est dangereux; toute opération Drift est à gérer comme asynchrone.

## 12. Lien avec le module suivant

Les opérations CRUD existent; le prochain module apprend à préciser quelles lignes lire, comment les combiner, trier et paginer.
