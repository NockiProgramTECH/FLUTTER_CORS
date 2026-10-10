**Navigation :** [← Module 22](./22-integration-gestion-etat.md) · [Sommaire](../README.md) · [Module 24 →](./24-tests-fiabilite.md)

# Module 23 — Bonnes pratiques et erreurs fréquentes

## 1. Objectif pédagogique

Vous apprendrez à reconnaître les erreurs de génération, de schéma, d’intégrité, d’asynchronisme et d’architecture, et à proposer une correction plutôt qu’un contournement destructif.

## 2. Le problème à résoudre

Une application peut compiler sur une base neuve et pourtant échouer chez les personnes qui ont des données depuis plusieurs versions. Les erreurs de configuration, une requête sans filtre ou une ressource non fermée sont souvent plus coûteuses que la syntaxe elle-même.

## 3. Explication du fonctionnement

| Erreur | Cause et conséquence | Correction raisonnée |
|---|---|---|
| Oublier de régénérer | Le `.g.dart` ne correspond plus aux tables; classes/getters manquent ou restent périmés. | Relancer `dart run build_runner build`; ne pas éditer le fichier généré. |
| Mal configurer build_runner | `drift_dev`/`build_runner` absent des dev_dependencies ou mauvais `part`. Aucun générateur ne produit l’API. | Vérifier `pubspec.yaml`, import, chemin de part, versions résolues puis la sortie du générateur. |
| Modifier le schéma sans migration | Base installée reste ancienne; colonnes ou contraintes attendues manquent. | Augmenter `schemaVersion`, créer/tester une migration préservant les données. |
| Confondre modèle Dart et table Drift | Une classe métier n’est pas automatiquement une ligne SQL; nullable/defaults peuvent diverger. | Séparer modèle métier/ligne/table ou configurer explicitement conversion et mapping. |
| Oublier contraintes d’intégrité | Valeurs invalides ou références orphelines peuvent s’enregistrer. | Définir PK, FK, unique/CHECK et activer/tester les FK. |
| Écritures liées sans transaction | Une partie du processus métier reste enregistrée après une erreur. | Regrouper les écritures atomiques dans `transaction`. |
| Trop d’index | Espace et coût d’écriture augmentent, bénéfice incertain. | Indexer selon des requêtes observées et mesurer. |
| Requêtes inutiles | Polling, `get()` répétés ou requête dans `build` sollicitent inutilement le moteur. | Réutiliser résultat/Stream stable, filtrer/paginer et placer l’I/O hors de `build`. |
| `double` pour argent | Arrondis binaires et cumul d’erreur. | Enregistrer des unités entières adaptées (cents) et formater côté présentation. |
| Logique métier dans les widgets | Requêtes relancées, tests difficiles, UI couplée à la persistance. | Déplacer validation/orchestration dans contrôleur/cas d’usage et DAO/repository. |
| Ressources mal fermées | Streams, listeners, timers ou connexion continuent après leur propriétaire. | Désabonner dans `dispose`; fermer la base au niveau de vie de l’application. |
| Asynchronisme mal géré | UI bloquée logiquement, erreur non présentée, mise à jour après destruction du widget. | `await`/Future/Stream, état chargement/erreur, annulation/désabonnement et vérification de vie si nécessaire. |
| Supprimer/recréer la base en production | Les données locales utilisateur disparaissent. | Écrire une migration; réserver la réinitialisation aux bases jetables de développement ou à une action explicite. |

## 4. Petit extrait de code

```dart
// Dangereux : modifie potentiellement toutes les tâches.
await database.update(database.tasks).write(
  const TasksCompanion(done: Value(true)),
);
```

## 5. Explication ligne par ligne

- `update(database.tasks)` cible toute la table; aucun filtre n’est encore présent.
- `.write(...)` applique le companion à chaque ligne correspondante. Sans `where`, toutes les tâches peuvent devenir terminées.
- La forme correcte dépend de l’intention : pour une seule ligne, ajouter `..where((t) => t.id.equals(id))`; pour une action globale, afficher/valider clairement cette intention et vérifier le nombre de lignes affectées.

## 6. Déroulement interne

Les erreurs se propagent à différents moments : génération (code source/outillage), compilation (types et méthodes générées), exécution SQL (contrainte, connexion), puis transition d’état/affichage (erreur non gérée). Identifier **à quelle couche** le problème apparaît réduit les corrections dangereuses.

## 7. Pourquoi cette approche ?

Des contrôles à chaque couche réduisent les erreurs silencieuses : générer tôt, contraindre les données, isoler les opérations, tester les migrations, fermer les ressources. Les bonnes pratiques ne sont pas des rituels : chacune répond à un mode de panne précis.

## 8. Erreur classique

**Erreur :** réparer une erreur de schéma par `deleteDatabase` ou en retirant une contrainte. Cela peut masquer un problème tout en perdant des données. **Correction :** reproduire l’erreur sur une base de test, vérifier la version et le SQL, puis corriger le générateur, la requête ou la migration responsable.

## 9. Questions de compréhension

1. Quel est le bon correctif si un getter Drift n’apparaît pas après avoir ajouté une table?
2. Pourquoi une contrainte manquante est-elle différente d’une validation UI oubliée?
3. Quel dommage peut produire un `UPDATE` sans `where`?
4. Pourquoi une base neuve qui fonctionne ne prouve-t-elle pas qu’un changement de schéma est sûr?

## 10. Petit exercice pratique

- **Objectif :** classer les erreurs par couche.
- **Situation de départ :** (A) classe générée manquante; (B) FK ignorée; (C) crash après une ancienne installation; (D) widget relance `get()` dans `build`.
- **Résultat attendu :** couche affectée et correction pour chaque cas.
- **Consignes :** choisissez parmi génération, configuration SQLite, migration ou architecture/asynchronisme.
- **Critères de réussite :** vous ne répondez pas « supprimer la base » à C et « ajouter setState » à tous les cas.
- **Indice facultatif :** les symptômes apparaissent à des moments différents du cycle de développement.

## 11. Résumé essentiel

- Régénérez les API Drift après une modification de schéma source.
- Une migration préserve les bases déjà installées; la génération seule ne le fait pas.
- Contraintes, transactions, index mesurés et ressources bien fermées évitent des classes de panne distinctes.
- Requêtes et logique métier ne doivent pas être déclenchées aveuglément par `build`.

## 12. Lien avec le module suivant

Les erreurs étant identifiées, le prochain module transforme les attentes de fiabilité en tests ciblés.
