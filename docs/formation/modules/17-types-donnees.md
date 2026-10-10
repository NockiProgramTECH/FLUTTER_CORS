**Navigation :** [← Module 16](./16-requetes-filtres.md) · [Sommaire](../README.md) · [Module 18 →](./18-transactions.md)

# Module 17 — Exploiter les différents types de données

## 1. Objectif pédagogique

Vous saurez manipuler nombres, texte, booléens, dates, valeurs nulles, collections, BLOB, énumérations et types personnalisés, en choisissant un stockage durable et interprétable.

## 2. Le problème à résoudre

Une tâche contient un statut, une date de création, des étiquettes et parfois un coût. Un choix de stockage trop rapide peut provoquer arrondis, données impossibles à interroger ou incompatibilités après évolution du code.

## 3. Explication du fonctionnement

Les entiers conviennent aux compteurs, identifiants et unités discrètes. Les `double` sont approximatifs : une somme monétaire en flottants peut accumuler des erreurs binaires (`0.1` n’a pas une représentation binaire exacte). Pour une devise à deux décimales, stockez souvent les **centimes** en entier, et précisez devise et règle d’arrondi si plusieurs devises/échelles sont possibles.

Les booléens et les dates sont convertis selon le schéma/option Drift (module 12). Les valeurs nulles expriment une absence réelle, pas « zéro » ni chaîne vide. Une liste de petites options opaques peut être encodée en JSON via un `TypeConverter`; une collection que l’on doit filtrer, relier, dédupliquer ou mettre à jour indépendamment devrait plutôt être normalisée en plusieurs lignes/tables.

Les énumérations peuvent être stockées comme indice entier (`intEnum`) ou nom texte (`textEnum`). L’indice est compact mais changer l’ordre des valeurs réinterprète les données historiques; si l’indice est utilisé, ajoutez les nouvelles valeurs à la fin ou migrez explicitement. Un stockage textuel est plus lisible et moins sensible à l’ordre, mais renommer une valeur persistée est aussi une migration métier.

Un `TypeConverter<DartType, SqlType>` définit `toSql` et `fromSql`. Drift peut alors présenter un type riche en Dart et sa représentation simple à SQLite. Les convertisseurs JSON nécessitent une attention supplémentaire si l’on souhaite aussi sérialiser les lignes en JSON.

## 4. Petit extrait de code

```dart
enum TaskStatus { open, done }
class TaskStatusConverter extends TypeConverter<TaskStatus, String> {
  const TaskStatusConverter();
  @override TaskStatus fromSql(String value) => TaskStatus.values.byName(value);
  @override String toSql(TaskStatus value) => value.name;
}
TextColumn get status => text().map(const TaskStatusConverter())();
```

## 5. Explication ligne par ligne

- `TaskStatus` est un type riche côté Dart, avec deux états nommés.
- `TypeConverter<TaskStatus, String>` indique que les objets Dart `TaskStatus` sont encodés comme texte SQL.
- `fromSql` transforme le texte lu en valeur Dart; si une valeur historique ne correspond plus, l’erreur doit être anticipée par une migration ou une stratégie de lecture compatible.
- `toSql` transforme l’énumération en nom texte stable pour l’insertion.
- `.map(...)` connecte le convertisseur à la colonne. Le code qui lit/écrit manipule `TaskStatus`, alors que SQLite contient une `String`.

## 6. Déroulement interne

Pour un statut, la chaîne `open` est écrite en SQL; Drift la convertit à la lecture en `TaskStatus.open`. Pour un montant de 12,34 dans une devise à deux décimales, l’application peut stocker `1234` centimes, puis formatter `1234` en `12,34` pour l’UI. Pour une liste d’étiquettes filtrable, une table `TaskTags(taskId, tagId)` conserve mieux l’intégrité qu’un texte JSON opaque.

## 7. Pourquoi cette approche ?

Un stockage choisi selon l’usage évite les pertes de précision et facilite les recherches. Les convertisseurs sont utiles aux types métier précis, mais cachent une représentation persistée : leur convention doit être stable et testée. Une colonne JSON compacte simplifie les données peu structurées mais affaiblit les contraintes et les recherches relationnelles.

## 8. Erreur classique

**Erreur :** stocker un prix comme `double`, ou utiliser l’indice d’une enum puis insérer une nouvelle valeur au milieu. **Correction :** utiliser une unité entière appropriée pour l’argent; rendre les codes de statut persistés stables et migrer les changements incompatibles.

## 9. Questions de compréhension

1. Pourquoi `double` peut-il afficher une petite différence après des additions d’argent?
2. Quand une collection JSON est-elle acceptable et quand vaut-il mieux créer une table?
3. Pourquoi déplacer une valeur au milieu d’une enum entière peut-il corrompre l’interprétation historique?
4. Quel travail Drift délègue-t-il à `toSql` et `fromSql`?

## 10. Petit exercice pratique

- **Objectif :** choisir les représentations métier.
- **Situation de départ :** chaque tâche a un statut, un coût en devise à deux décimales, une date de création et plusieurs étiquettes recherchables.
- **Résultat attendu :** texte enum/converter stable, montant en unité entière, DateTime avec format explicite, relation de tags normalisée.
- **Consignes :** justifiez pourquoi ne pas mettre toutes les étiquettes dans un seul champ texte.
- **Critères de réussite :** vous évitez le flottant financier et permettez une recherche par étiquette.
- **Indice facultatif :** une donnée recherchable et partageable entre tâches est un bon candidat de relation.

## 11. Résumé essentiel

- `double` est approximatif; un montant monétaire se stocke souvent en unités entières mineures.
- Nullable signifie absence; chaîne vide et zéro sont d’autres valeurs.
- Les collections requêtables se modélisent généralement par tables liées; JSON convient à des objets peu interrogés.
- Les convertisseurs relient une valeur Dart à son type SQL; les conventions et enums doivent rester compatibles avec les anciennes lignes.

## 12. Lien avec le module suivant

Même si chaque valeur est bien typée, plusieurs écritures doivent parfois être validées ensemble : c’est le rôle d’une transaction.
