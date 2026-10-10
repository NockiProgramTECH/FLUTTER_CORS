**Navigation :** [← Sommaire](./README.md)

# Références officielles et notes de version

Liens consultés pour vérifier les concepts et exemples le 10 octobre 2026 :

- [Cycle de vie de `State` — API Flutter](https://api.flutter.dev/flutter/widgets/State-class.html)
- [Gestion d’état simple avec `ChangeNotifier` et Provider — documentation Flutter](https://docs.flutter.dev/data-and-backend/state-mgmt/simple)
- [Package `provider` — pub.dev](https://pub.dev/packages/provider)
- [Références et `ref.watch` — documentation Riverpod](https://riverpod.dev/docs/concepts2/refs)
- [Concepts Cubit et BLoC — documentation bloc](https://bloclibrary.dev/bloc-concepts/)
- [Configuration et démarrage Drift](https://drift.simonbinder.eu/setup/)
- [Tables, colonnes, contraintes, index et dates Drift](https://drift.simonbinder.eu/dart_api/tables/)
- [Requêtes de sélection Drift](https://drift.simonbinder.eu/dart_api/select/)
- [Écritures Drift](https://drift.simonbinder.eu/dart_api/writes/)
- [Flux de requêtes Drift](https://drift.simonbinder.eu/dart_api/streams/)
- [Convertisseurs de types Drift](https://drift.simonbinder.eu/type_converters/)
- [Migrations Drift](https://drift.simonbinder.eu/migrations/)
- [API du migrateur et TableMigration](https://drift.simonbinder.eu/migrations/api/)
- [Tests de migrations et `SchemaVerifier`](https://drift.simonbinder.eu/migrations/tests/)
- [Tests de bases Drift](https://drift.simonbinder.eu/testing/)

**Détails sensibles à la version :** les exemples de dépendances Drift du guide consulté indiquaient `drift`/`drift_dev` 2.35.0, `drift_flutter` 0.3.1 et `build_runner` 2.16.1; les dates peuvent s’afficher comme `INTEGER` timestamp Unix en secondes par défaut historique ou en `TEXT` ISO-8601 selon l’option du générateur; les migrations guidées s’appuient sur la commande `dart run drift_dev make-migrations`; les noms et comportements d’API générées doivent être vérifiés après résolution des dépendances. L’API `MigrationStrategy` consultée documente `onCreate`, `onUpgrade` et `beforeOpen` (pas de callback séparé `onDowngrade`). Pour tout exemple réel, relancez l’analyseur et les tests sous la version de Drift réellement installée.
