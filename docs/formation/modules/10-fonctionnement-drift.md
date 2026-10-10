**Navigation :** [← Module 9](./09-stockage-local-sqlite.md) · [Sommaire](../README.md) · [Module 11 →](./11-installation-initialisation.md)

# Module 10 — Comprendre le fonctionnement interne de Drift

## 1. Objectif pédagogique

Vous comprendrez le vocabulaire relationnel, le rôle de la génération de code et la relation entre vos classes de tables et le fichier généré `database.g.dart`.

## 2. Le problème à résoudre

Une requête écrite avec des noms SQL bruts peut comporter une faute de colonne ou convertir le mauvais type sans que l’analyseur Dart la détecte. On souhaite décrire la structure une fois et bénéficier d’API cohérentes.

## 3. Explication du fonctionnement

Une **base** contient des tables. Une **table** décrit une catégorie d’enregistrements; une **colonne** un attribut; une **ligne/enregistrement** une occurrence. Une **clé primaire** identifie une ligne sans ambiguïté. Une **clé étrangère** référence la clé d’une autre table. Une **requête SQL** exprime au moteur une lecture ou une écriture.

Le **schéma** est la description des tables, colonnes et contraintes. Un **ORM** (Object-Relational Mapper) fait le lien entre tables relationnelles et objets du langage; Drift est une bibliothèque de persistance relationnelle typée qui génère des classes de lignes/compagnons et comprend des requêtes. Il ne charge pas automatiquement tout un graphe relationnel comme le ferait parfois un ORM « tout compris » : les jointures et accès aux relations restent explicites.

Le développeur écrit des classes qui étendent `Table` et déclare les tables dans `@DriftDatabase`. L’annotation `part 'database.g.dart';` relie un fichier Dart à une **partie générée**. Le générateur crée notamment `_$AppDatabase`, les getters de tables, les classes de lignes et companions. `database.g.dart` ne doit pas être édité à la main : il est dérivé de vos déclarations. La vérification **statique** détecte plusieurs incohérences au moment de l’analyse/génération, mais les contraintes de la base restent appliquées à l’exécution par SQLite.

L’**intégrité référentielle** signifie, par exemple, qu’une tâche ne pointe pas vers un projet inexistant. Déclarer une référence ne suffit pas toujours à l’activer avec SQLite; il faut aussi activer les clés étrangères sur la connexion (module 14).

## 4. Petit extrait de code

```dart
class Projects extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}
@DriftDatabase(tables: [Projects])
class AppDatabase extends _$AppDatabase { /* connexion et version */ }
```

## 5. Explication ligne par ligne

- `Projects extends Table` dit à Drift que cette classe décrit un schéma, pas une instance de projet.
- `integer().autoIncrement()` décrit une colonne entière générée par SQLite et utilisée comme clé primaire.
- `text()` décrit une colonne textuelle; Drift associera une valeur Dart `String`.
- `@DriftDatabase(tables: [Projects])` indique au générateur quelle table inclure dans cette base.
- `_$AppDatabase` est le nom de la superclasse produite à partir du nom de `AppDatabase`; elle expose notamment le getter `projects` et les API d’accès générées.

## 6. Déroulement interne

```mermaid
flowchart LR
  A[Classe Table écrite par le développeur] --> G[Analyse/génération Drift]
  B[@DriftDatabase + part] --> G
  G --> F[database.g.dart généré]
  F --> API[Tables, lignes, companions, base typée]
  API --> Q[Drift construit SQL]
  Q --> SQLite[(Moteur SQLite)]
```

Les noms exacts et API générées dépendent des noms de tables, des options de génération et de la version.

## 7. Pourquoi cette approche ?

Le code généré évite de maintenir manuellement des classes répétitives et offre une correspondance cohérente entre colonnes, types Dart et SQL. Le coût est une étape de génération et des contraintes de compilation : si le schéma change, le fichier généré doit être régénéré puis la base déjà présente doit être migrée.

## 8. Erreur classique

**Erreur :** modifier `database.g.dart` directement ou croire que la classe `Projects` est une ligne de projet. **Correction :** modifiez le schéma source ou le modèle métier, relancez le générateur et distinguez classe de table, classe générée de ligne et objet insérable.

## 9. Questions de compréhension

1. Pourquoi `Projects extends Table` ne représente-t-il pas un projet individuel?
2. Quel travail disparaît si Drift génère les classes de ligne et de compagnon?
3. Une requête typée garantit-elle qu’une migration historique préservera toutes les données? Pourquoi?
4. Quelle différence existe entre une clé primaire et une référence étrangère?

## 10. Petit exercice pratique

- **Objectif :** reconnaître les éléments sources et générés.
- **Situation de départ :** le fichier `.dart` contient une classe `Tasks extends Table`, `@DriftDatabase` et un `part 'database.g.dart'`.
- **Résultat attendu :** citer ce que le développeur écrit et ce que le générateur fournit.
- **Consignes :** classez cinq éléments : `Tasks`, `Task` (ligne), `TasksCompanion`, `_$AppDatabase`, `database.g.dart`.
- **Critères de réussite :** vous identifiez `Tasks` comme définition, le fichier et la superclasse comme générés, et la ligne comme un enregistrement.
- **Indice facultatif :** demandez lequel de ces éléments est la source à éditer.

## 11. Résumé essentiel

- Tables, colonnes et lignes représentent une structure relationnelle.
- Le développeur décrit; le générateur produit les API répétitives et typées.
- Une clé primaire identifie; une clé étrangère relie et contraint.
- Le code typé aide à prévenir les fautes, mais ne remplace ni contraintes SQLite ni tests de migration.

## 12. Lien avec le module suivant

Pour obtenir cette API générée, il faut installer le paquet d’exécution, les outils de développement et configurer l’ouverture de la base.
