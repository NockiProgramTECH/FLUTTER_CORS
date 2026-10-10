**Navigation :** [← Module 10](./10-fonctionnement-drift.md) · [Sommaire](../README.md) · [Module 12 →](./12-types-colonnes.md)

# Module 11 — Installer et initialiser Drift

## 1. Objectif pédagogique

Vous comprendrez quelles dépendances sont nécessaires, pourquoi la génération est distincte de l’exécution, comment déclarer une base et comment ouvrir une connexion SQLite avec Drift.

## 2. Le problème à résoudre

Une classe `AppDatabase extends _$AppDatabase` ne compile pas avant que `_$AppDatabase` existe. Il faut que l’outil de génération lise les déclarations et crée les parties manquantes, et qu’un exécuteur ouvre un fichier SQLite à l’emplacement approprié.

## 3. Explication du fonctionnement

Une **dépendance d’exécution** est nécessaire quand l’application fonctionne : `drift` contient les API de base; `drift_flutter` fournit l’intégration Flutter qui ouvre un exécuteur local. Une **dépendance de développement** sert à construire/analyser/tester le code, mais n’est pas une bibliothèque appelée à chaque insertion : `drift_dev` contient le générateur/outillage et `build_runner` exécute les générateurs Dart.

Au moment de la documentation vérifiée, le guide officiel montrait Drift 2.35.0, `drift_flutter` 0.3.1 et `build_runner` 2.16.1. Ce sont des exemples de versions à la date de vérification, pas une promesse que ce sont toujours les dernières. Le SDK résout des versions compatibles dans `pubspec.lock`.

## 4. Petit extrait de code

Dépendances (versions illustratives vérifiées le 10-10-2026) :

```yaml
dependencies:
  drift: ^2.35.0
  drift_flutter: ^0.3.1
  path_provider: ^2.1.6 # si vous choisissez explicitement un répertoire
dev_dependencies:
  drift_dev: ^2.35.0
  build_runner: ^2.16.1
```

Squelette à compléter dans `database.dart` :

```dart
part 'database.g.dart';
@DriftDatabase(tables: [Tasks])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(driftDatabase(name: 'tasks'));
  @override int get schemaVersion => 1;
}
```

Génération ponctuelle :

```sh
dart run build_runner build
```

## 5. Explication ligne par ligne

- `drift` est importé par le code qui déclare tables et requêtes; `drift_flutter` apporte ici `driftDatabase`.
- `drift_dev` et `build_runner` sont des outils de développement : s’ils manquent, `dart run build_runner` ne peut pas trouver/exécuter le générateur Drift.
- `part` désigne le fichier de code généré associé au fichier Dart.
- `@DriftDatabase` inscrit les tables dans cette base; `_$AppDatabase` est créé par génération.
- `driftDatabase(name: 'tasks')` ouvre un exécuteur nommé. Le stockage par défaut est géré par l’intégration; pour un emplacement personnalisé, le guide officiel montre notamment `DriftNativeOptions` et `path_provider`.
- `schemaVersion` identifie la version du schéma persisté; elle ne remplace pas les migrations.
- `dart run build_runner build` fait une génération ponctuelle. `dart run build_runner watch` reste actif en développement et régénère lors de changements.

## 6. Déroulement interne

1. `flutter pub get`/`dart pub get` résout les versions.
2. Le générateur lit les tables et l’annotation.
3. `build_runner` produit `database.g.dart`.
4. Dart analyse la classe générée et votre base.
5. À l’exécution, le constructeur donne un `QueryExecutor` à la classe Drift; à la première ouverture, Drift initialise une nouvelle base et déclenche `onCreate` de la stratégie de migration.

Pour les migrations guidées, configurez aussi `drift_dev` dans `build.yaml`, puis utilisez l’outil dédié `dart run drift_dev make-migrations`; ce n’est pas une commande qui remplace la génération ordinaire du code Dart.

## 7. Pourquoi cette approche ?

La génération au moment du développement permet de détecter tôt un schéma incohérent et évite d’écrire à la main les classes de lignes et de colonnes. Le coût est qu’une déclaration modifiée exige une régénération. Les commandes actuelles utilisent `dart run`; d’anciens tutoriels peuvent montrer `flutter pub run`, des paquets `moor` ou d’autres API de connexion.

## 8. Erreur classique

**Erreur :** installer `drift`, mais oublier `drift_dev` ou `build_runner`, puis lancer la génération; ou éditer le `.g.dart` manquant. **Correction :** vérifiez les deux `dev_dependencies`, le chemin `part` et le nom du fichier, puis lancez `dart run build_runner build`. Les avertissements avant la première génération sont attendus.

## 9. Questions de compréhension

1. Pourquoi l’application a-t-elle besoin de `drift`, alors que l’outil de génération n’est pas une dépendance d’exécution?
2. Que cherche le compilateur quand il rencontre `extends _$AppDatabase`?
3. Quelle différence existe entre `build` et `watch` de build_runner?
4. Si un ancien tutoriel déclare `moor_generator`, pourquoi faut-il vérifier sa date et sa version?

## 10. Petit exercice pratique

- **Objectif :** diagnostiquer une génération introuvable.
- **Situation de départ :** Drift est dans `dependencies`, `database.g.dart` manque et `dart run build_runner build` indique qu’aucune cible utile n’existe.
- **Résultat attendu :** une liste courte de vérifications.
- **Consignes :** contrôlez emplacement du paquet générateur, `build_runner`, import, `part` et déclaration de base; ne supprimez aucune donnée.
- **Critères de réussite :** vous identifiez les dépendances de développement et le code généré comme les points à vérifier.
- **Indice facultatif :** le moteur SQLite n’est pas l’outil qui produit le fichier Dart.

## 11. Résumé essentiel

- `drift`/l’intégration de plateforme servent à l’exécution; `drift_dev` et `build_runner` servent à la génération.
- La classe générée `_$AppDatabase` apparaît après le générateur.
- `build` génère une fois; `watch` génère en continu pendant le développement.
- Les détails de connexion, options de stockage et versions doivent être vérifiés dans le guide correspondant à la version du projet.

## 12. Lien avec le module suivant

La connexion est en place; voyons quels types Dart peuvent être représentés par les colonnes SQLite et quelles conversions Drift effectue.
