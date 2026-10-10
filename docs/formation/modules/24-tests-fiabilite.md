**Navigation :** [← Module 23](./23-bonnes-pratiques.md) · [Sommaire](../README.md) · [Module 25 →](./25-performances-securite.md)

# Module 24 — Tests et fiabilité

## 1. Objectif pédagogique

Vous saurez tester les opérations, contraintes, relations, transactions, flux et migrations; vous comprendrez pourquoi une base en mémoire convient à certains tests mais pas à tous.

## 2. Le problème à résoudre

Une requête peut sembler correcte dans l’écran mais ne pas respecter une contrainte. Une migration peut créer le bon schéma à vide et perdre les données d’une version antérieure. Il faut vérifier chaque garantie au niveau où elle est appliquée.

## 3. Explication du fonctionnement

Un test de logique métier peut utiliser un faux repository et vérifier une règle sans SQLite. Un test d’intégration de persistance ouvre une vraie base SQLite de test, insère des lignes, exécute une requête et inspecte résultat/contrainte. `NativeDatabase.memory()` donne une base éphémère pratique et rapide; elle vérifie le SQL et le mapping SQLite, mais pas forcément le chemin de fichier, permissions, plugin ou comportement spécifique d’un appareil.

Drift permet d’injecter un `QueryExecutor` dans la base de test. Fermez la base dans `tearDown`. On peut tester `watch` en attendant plusieurs émissions, une transaction en forçant une exception, les contraintes en vérifiant qu’une insertion échoue, et les migrations en partant d’un ancien schéma. `SchemaVerifier` et les tests générés par `make-migrations` comparent le schéma effectif avec le schéma attendu. Pour la persistance après fermeture, utilisez un fichier temporaire, fermez puis rouvrez le même fichier; une base en mémoire ne peut pas prouver cette propriété.

## 4. Petit extrait de code

```dart
setUp(() {
  database = AppDatabase(NativeDatabase.memory());
});
tearDown(() async {
  await database.close();
});
```

## 5. Explication ligne par ligne

- `setUp` prépare une base isolée avant chaque test, donc les lignes d’un test ne contaminent pas le suivant.
- `NativeDatabase.memory()` crée SQLite en mémoire et évite les fichiers de production.
- `AppDatabase(...)` reçoit cet exécuteur par injection; le code métier testé reste le même.
- `tearDown` ferme connexion et flux afin de ne pas laisser de ressource active entre les tests.
- En tests de widgets, Drift documente `DatabaseConnection(..., closeStreamsSynchronously: true)` pour éviter que des abonnements ouverts un tour d’événement ne survivent à la fin d’un test.

## 6. Déroulement interne

Plan de couverture utile :

1. **CRUD/recherche :** résultat et nombre de lignes touchées.
2. **Contraintes/relations :** valeur interdite rejetée; suppression conforme à l’action FK.
3. **Transaction :** provoquer une erreur au milieu et vérifier le rollback.
4. **Réactivité :** vérifier une séquence d’émissions avant et après écriture.
5. **Migration :** créer schéma/ligne à v1, migrer, vérifier schéma et valeur conservée.
6. **Persistance :** écrire dans un fichier temporaire, fermer et rouvrir ce même fichier.

## 7. Pourquoi cette approche ?

Les tests en mémoire rendent les tests SQL fréquents rapides; les tests sur fichiers valident l’ouverture/réouverture et la persistance; les tests de migration démarrent avec de vraies versions historiques. Un test unitaire de logique et un test d’intégration Drift répondent à des questions différentes et sont complémentaires.

## 8. Erreur classique

**Erreur :** tester seulement `onCreate` sur une base neuve après avoir ajouté une colonne. Le test prouve que la cible peut être créée, pas que les anciens utilisateurs peuvent migrer vers elle. **Correction :** créer un jeu de départ au schéma historique, mettre des données, appliquer la migration et vérifier à la fois intégrité et valeurs.

## 9. Questions de compréhension

1. Quel aspect un test `NativeDatabase.memory()` ne prouve-t-il pas sur l’emplacement de stockage de l’application?
2. Pourquoi fermer la base après chaque test?
3. Comment prouver que les modifications d’une transaction sont annulées après erreur?
4. Quelle différence existe entre tester `onCreate` et `v1 → v2`?

## 10. Petit exercice pratique

- **Objectif :** rédiger les assertions d’un test de migration.
- **Situation de départ :** v1 contient une tâche `title = 'Appeler Lina'`; v2 ajoute `done` avec défaut faux.
- **Résultat attendu :** après migration, la tâche existe, garde son titre et a `done == false`.
- **Consignes :** décrivez la base source, l’appel du vérificateur/migration et les assertions; ne testez pas seulement une base vide.
- **Critères de réussite :** au moins une ligne historique est conservée et la nouvelle valeur est vérifiée.
- **Indice facultatif :** les snapshots de schéma permettent de commencer à la version 1.

## 11. Résumé essentiel

- Les tests unitaires vérifient des règles; les tests d’intégration vérifient SQL, contraintes et stockage.
- Une base en mémoire est pratique, mais ne prouve pas la persistance dans un fichier.
- Flux, rollback et contraintes ont des tests dédiés.
- Une base neuve ne remplace jamais un test de migration depuis un schéma publié.

## 12. Lien avec le module suivant

La fiabilité doit également tenir à l’usage réel : volumes, durée de connexion, sécurité, sauvegardes et limites du stockage local.
