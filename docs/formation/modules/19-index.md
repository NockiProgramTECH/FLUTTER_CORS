**Navigation :** [← Module 18](./18-transactions.md) · [Sommaire](../README.md) · [Module 20 →](./20-migrations.md)

# Module 19 — Comprendre les index

## 1. Objectif pédagogique

Vous comprendrez le rôle des index, leur effet sur lectures et écritures, les index simples/composites/uniques et la manière de les aligner sur les requêtes réelles.

## 2. Le problème à résoudre

Chercher un produit parmi des centaines de milliers par code-barres ou filtrer les tâches d’un projet par statut pourrait obliger SQLite à examiner beaucoup de lignes à chaque requête.

## 3. Explication du fonctionnement

Un **index** est une structure de recherche auxiliaire maintenue par la base, qui associe les valeurs indexées aux lignes correspondantes. Sans index adapté, le moteur peut parcourir chaque ligne. Avec index, une recherche/tri bien aligné peut réduire le travail; en contrepartie, chaque insertion, modification de valeur indexée et suppression doit aussi mettre à jour l’index et consomme de l’espace.

Un index **simple** porte sur une colonne. Un **composite** porte sur plusieurs colonnes dans un ordre précis : `(projectId, done)` est utile pour `projectId` seul, ou `projectId + done`, mais ne remplace pas nécessairement un index dédié à `done` seul. Un index **unique** interdit aussi les doublons. `UNIQUE` exprime d’abord une règle d’intégrité; l’index ordinaire n’exprime pas cette règle.

Les clés primaires et colonnes uniques sont indexées pour garantir l’unicité. La clé référencée côté parent doit être une clé primaire/unique. La colonne enfant de FK, elle, peut nécessiter un index séparé selon les requêtes et suppressions; SQLite ne crée pas automatiquement un index performant sur toute colonne enfant. Les colonnes peu distinctes ou très peu lues ne gagnent pas forcément à être indexées.

## 4. Petit extrait de code

```dart
@TableIndex(name: 'tasks_project_done', columns: {#projectId, #done})
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get projectId => integer()();
  BoolColumn get done => boolean()();
}
```

## 5. Explication ligne par ligne

- `@TableIndex` déclare un index nommé; le nom doit être unique dans le schéma SQL.
- `columns: {#projectId, #done}` décrit l’ordre des colonnes indexées (dans les API Drift actuelles, des symboles identifient les getters). Vérifiez l’exemple de la version résolue si l’annotation évolue.
- La requête `WHERE project_id = ? AND done = ?` peut bénéficier de cet index, surtout quand la table est assez grande.
- Il ne faut pas déclarer le même index en plus de l’index primaire auto-généré sur `id` sans raison; Drift/SQLite indexent déjà les clés nécessaires à l’unicité.

## 6. Déroulement interne

Lors d’une insertion, SQLite ajoute la ligne et l’entrée correspondante dans l’index. Lors d’une recherche compatible, le moteur peut consulter cette structure puis récupérer les lignes. L’optimiseur décide du plan réel : un index ne garantit pas l’accélération si la requête retourne presque toute la table, si l’ordre des colonnes ne convient pas, ou si le coût de l’index dépasse le bénéfice.

## 7. Pourquoi cette approche ?

Un index choisi depuis des requêtes observées rend les recherches fréquentes plus rapides. Les contraintes `UNIQUE` et les index d’accès répondent à des objectifs distincts, même si une structure d’index réalise souvent l’unicité. Mesurez sur des volumes réalistes et privilégiez les filtres/tri qui reviennent dans l’usage réel.

## 8. Erreur classique

**Erreur :** indexer toutes les colonnes « au cas où ». Cela augmente taille et coût des écritures, sans aider tous les plans SQL. **Correction :** lister les requêtes fréquentes, indexer leur filtre/ordre composite utile, puis mesurer.

## 9. Questions de compréhension

1. Pourquoi un index peut-il accélérer un `SELECT` mais ralentir une insertion?
2. Un index unique et un index ordinaire garantissent-ils la même chose?
3. Un index `(projectId, done)` est-il forcément optimal pour les seules tâches terminées de tous les projets?
4. Pourquoi indexer la clé primaire n’est-il généralement pas nécessaire manuellement?

## 10. Petit exercice pratique

- **Objectif :** justifier un index de tâche.
- **Situation de départ :** une page demande souvent les tâches d’un projet, ouvertes d’abord, triées par date; les tâches sont nombreuses.
- **Résultat attendu :** proposer un index candidat et noter une limite à vérifier.
- **Consignes :** comparez `(projectId, done, createdAt)` et `createdAt` seul; reliez le choix à la requête et au coût d’écriture.
- **Critères de réussite :** la justification cite l’ordre de filtrage/requête et n’indexe pas toutes les colonnes par principe.
- **Indice facultatif :** un index doit servir le début de la requête réelle.

## 11. Résumé essentiel

- Un index accélère certaines recherches/ordres, mais coûte de l’espace et des mises à jour.
- Index simple, composite et unique n’ont pas exactement le même contrat.
- Un index composite dépend de l’ordre de ses colonnes.
- La clé étrangère côté enfant peut nécessiter un index, selon le workload; aucune colonne ne doit être indexée sans raison.

## 12. Lien avec le module suivant

Les index font partie du schéma; toute évolution de schéma sur une base déjà présente exige une migration versionnée.
