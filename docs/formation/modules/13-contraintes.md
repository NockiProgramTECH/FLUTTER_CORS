**Navigation :** [← Module 12](./12-types-colonnes.md) · [Sommaire](../README.md) · [Module 14 →](./14-relations.md)

# Module 13 — Définir des colonnes et leurs contraintes

## 1. Objectif pédagogique

Vous comprendrez la différence entre nullabilité, défaut, clé, unicité, `CHECK` et référence étrangère, et verrez comment Drift exprime ces règles.

## 2. Le problème à résoudre

Une table accepte par défaut des requêtes valides pour sa structure, mais peut encore contenir des titres vides, des priorités négatives, un slug répété ou une tâche associée à un projet inexistant. Une règle d’intégrité doit empêcher les données incorrectes, même si plusieurs écrans écrivent dans la base.

## 3. Explication du fonctionnement

- **Nullable** (`nullable()`) permet de représenter l’absence par `null`; ce n’est ni une valeur par défaut ni une règle métier.
- **Default SQL** (`withDefault`) fournit une valeur lorsque l’insertion omet la colonne. Le moteur peut l’appliquer aussi aux écritures hors Drift. `clientDefault` calcule en Dart pour les insertions Drift, mais n’inscrit pas ce défaut dans le schéma SQL.
- **Clé primaire** identifie une ligne; `autoIncrement()` configure couramment une clé entière auto-générée.
- **Unique** interdit les doublons sur une colonne avec `unique()` ou sur un ensemble via `uniqueKeys`.
- **CHECK** refuse une ligne si une expression est fausse. Attention : l’ajout d’une règle doit aussi être compatible avec les lignes déjà présentes.
- **Clé étrangère** relie une colonne à une clé cible; `onDelete`/`onUpdate` définissent les actions en cas de suppression/modification de la cible.
- **Contrainte multi-colonnes** peut être une clé primaire composée ou une unicité composée. Les contraintes personnalisées sont possibles, mais une contrainte SQL personnalisée peut remplacer des contraintes Drift; vérifiez notamment la présence de `NOT NULL`.

## 4. Petit extrait de code

```dart
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 120)();
  BoolColumn get done => boolean().withDefault(const Constant(false))();
  late final Column<int> priority = integer()
      .check(priority.isBiggerOrEqualValue(0))();
  TextColumn get slug => text().unique()();
  IntColumn get projectId => integer().nullable()
      .references(Projects, #id, onDelete: KeyAction.setNull)();
}
```

## 5. Explication ligne par ligne

- `id` identifie automatiquement chaque tâche; Drift sait qu’il s’agit de la clé primaire auto-incrémentée.
- `title` accepte des longueurs indiquées. Pour compatibilité historique, `withLength` réalise cette validation en Dart; ce n’est pas toujours un `CHECK` SQL appliqué à tous les clients. Une contrainte SQL renforcée peut être nécessaire si plusieurs outils écrivent.
- `done` n’est pas nullable; si l’insertion omet la valeur, SQLite utilise `false` converti en `0`.
- `priority` possède un `CHECK` empêchant les valeurs négatives; le type `Column<int>` rend les expressions numériques disponibles au moment de la définition. C’est une expression Drift, pas du SQL brut.
- `slug` est unique pour toutes les lignes.
- `projectId` peut être absent; s’il référence un projet, la suppression de celui-ci met cette colonne à `NULL`. Une valeur `NULL` est nécessaire pour que `SET NULL` soit cohérent.

Pour une règle composée, on peut déclarer `@override List<Set<Column>> get uniqueKeys => [{projectId, title}];` afin d’interdire deux tâches portant le même titre dans un projet. Les règles `onUpdate` et `onDelete` se configurent avec les actions `KeyAction` appropriées (par exemple `cascade`, `restrict` ou `setNull` selon le cas).

## 6. Déroulement interne

À l’insertion, Drift vérifie les valeurs et émet une requête. SQLite applique alors les contraintes du schéma : valeur obligatoire, clé unique, `CHECK`, références activées. Une erreur de contrainte devient une erreur asynchrone à gérer par l’appelant. Les valeurs par défaut ne se substituent pas à une contrainte : un défaut fournit une valeur en cas d’omission; une contrainte rejette une valeur invalide.

## 7. Pourquoi cette approche ?

Les contraintes protègent l’intégrité au niveau le plus proche du stockage et s’appliquent à tous les chemins d’écriture concernés. Elles simplifient la logique applicative et les migrations. Mais il faut les adapter aux données historiques et activer les clés étrangères SQLite avant de compter sur elles.

## 8. Erreur classique

**Erreur :** croire que `nullable()` signifie « facultatif avec valeur par défaut », ou qu’une validation dans un widget protège toute écriture. **Correction :** exprimer les règles durables dans le schéma; réserver la nullabilité à l’absence réellement permise; gérer l’échec de contrainte dans le code.

## 9. Questions de compréhension

1. Si une colonne est `nullable`, est-elle automatiquement renseignée avec `false`?
2. Quelle différence existe entre omettre une colonne avec un défaut et tenter d’insérer une valeur interdite par `CHECK`?
3. Pourquoi une colonne référencée avec `onDelete: setNull` doit-elle pouvoir être nulle?
4. À quoi sert une unicité sur `(projectId, title)` que `unique()` sur `title` ne représenterait pas?

## 10. Petit exercice pratique

- **Objectif :** choisir contraintes pour les tâches.
- **Situation de départ :** chaque tâche a un titre non vide, un statut initial « ouverte », une priorité positive ou nulle, et peut être sans projet.
- **Résultat attendu :** associer une stratégie à chacun de ces quatre besoins.
- **Consignes :** indiquez colonne obligatoire, défaut, `CHECK`, nullabilité ou référence; expliquez chaque choix.
- **Critères de réussite :** un défaut n’est pas confondu avec une contrainte; la référence optionnelle peut être nulle.
- **Indice facultatif :** une tâche sans projet est un cas autorisé, pas une erreur.

## 11. Résumé essentiel

- Nullable représente l’absence; le défaut fournit une valeur en cas d’omission; les contraintes limitent les valeurs acceptables.
- Une clé primaire identifie; `unique` interdit les doublons; `CHECK` applique un prédicat.
- Les clés étrangères et leurs actions protègent les relations si SQLite les active.
- Toute nouvelle contrainte doit être testée contre les données historiques.

## 12. Lien avec le module suivant

Nous pouvons maintenant relier des tables : les contraintes de référence prennent tout leur sens dans les relations entre projets et tâches.
