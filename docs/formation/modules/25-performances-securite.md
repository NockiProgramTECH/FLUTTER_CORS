**Navigation :** [← Module 24](./24-tests-fiabilite.md) · [Sommaire](../README.md) · [Exercices finaux →](../exercices-finaux.md)

# Module 25 — Performances, sécurité et cycle de vie

## 1. Objectif pédagogique

Vous saurez garder l’UI réactive, limiter les coûts de requête, gérer connexion et sauvegarde, protéger les données sensibles et reconnaître quand un serveur reste indispensable.

## 2. Le problème à résoudre

Une liste de centaines de milliers de tâches, une connexion ouverte par écran ou une base contenant des informations sensibles peuvent dégrader l’expérience ou exposer des données. « Local » ne rend pas automatiquement ces problèmes impossibles.

## 3. Explication du fonctionnement

Les API Drift sont asynchrones (`Future`/`Stream`), ce qui permet d’attendre les I/O sans bloquer la logique de l’interface. **Asynchrone ne signifie pas automatiquement que toute opération coûteuse tourne sur un isolate séparé** : le comportement dépend de l’exécuteur et de la plateforme. Pour du gros travail, vérifiez le backend choisi et utilisez un exécuteur adapté (par exemple une base native exécutée en arrière-plan selon le setup Drift).

Optimisez d’abord les requêtes réelles : sélectionnez les colonnes utiles, filtrez, indexez selon les plans et paginez. Évitez les milliers de petites requêtes (pattern N+1); une jointure ou un lot approprié peut réduire le travail. Ouvrez une base au niveau d’un repository/app, réutilisez sa connexion et fermez-la quand son propriétaire global s’arrête, pas à chaque `build`.

Une sauvegarde doit tenir compte de SQLite, de la plateforme, de l’état de la connexion et de la stratégie de restauration. Ne copiez pas aveuglément le fichier pendant des écritures actives, notamment avec WAL; utilisez une méthode de sauvegarde cohérente ou fermez/checkpoint selon la stratégie retenue. Toute restauration doit ensuite tolérer des migrations.

Le bac à sable d’application limite l’accès ordinaire aux fichiers mais **n’est pas un chiffrement de base**. Drift et SQLite ne chiffrent pas automatiquement. Pour les données sensibles, réduisez ce qui est stocké, protégez les clés/tokens via les mécanismes système sécurisés, examinez les sauvegardes/logs et, si nécessaire, adoptez une solution SQLite chiffrée compatible avec plateforme et exécuteur (par exemple une distribution SQLCipher). Le chiffrement implique gestion de clés, compatibilité et récupération; ce n’est pas une simple annotation Drift.

Une base locale seule ne permet pas partage multi-appareil, sauvegarde centrale, synchronisation et arbitrage de conflits. Une architecture hybride (source distante + cache SQLite) demande un protocole de synchronisation explicite.

## 4. Petit extrait de code

```dart
final page = await (select(tasks)
  ..orderBy([(t) => OrderingTerm(expression: t.id)])
  ..limit(pageSize, offset: offset)).get();
```

## 5. Explication ligne par ligne

- `select(tasks)` construit la lecture des tâches pour la page courante.
- `orderBy(id)` donne un ordre stable; sans ordre, SQL ne garantit pas une pagination répétable.
- `limit` borne le nombre de lignes transférées en Dart; un offset élevé peut devenir coûteux, auquel cas un curseur est à privilégier.
- `await` suspend la fonction asynchrone jusqu’au résultat sans demander à Flutter de bloquer la frame de l’UI.

## 6. Déroulement interne

Pour une liste visible, l’UI demande une page, SQLite exécute la requête, Drift mappe seulement les résultats, puis l’écran affiche le lot. La connexion demeure gérée par un propriétaire stable. Au redémarrage, la base locale peut relire les lignes; une synchronisation réseau est une autre séquence, avec conflits et erreurs supplémentaires.

## 7. Pourquoi cette approche ?

La pagination et les flux limités gardent l’usage mémoire proportionné. Une connexion bien gérée évite de rouvrir des fichiers ou multiplier les ressources. La sécurité doit être évaluée par sensibilité et menace réelle; un stockage local peut être suffisant pour des préférences non sensibles et insuffisant pour des secrets ou des données partagées.

## 8. Erreur classique

**Erreur :** « Toutes les API sont async, donc le CPU ne peut pas bloquer l’interface » ou « SQLite est dans un fichier privé, donc la base est chiffrée ». **Correction :** choisir/vérifier l’exécuteur, limiter le travail lourd, et décider séparément du chiffrement, des clés, sauvegardes et synchronisation.

## 9. Questions de compréhension

1. Pourquoi faut-il un ordre stable pour paginer?
2. Une méthode `Future` garantit-elle toujours l’exécution sur un autre isolate?
3. Pourquoi une copie du fichier de base pendant une écriture peut-elle être incohérente?
4. Quel service doit départager des modifications faites sur deux appareils hors ligne?

## 10. Petit exercice pratique

- **Objectif :** définir une stratégie pour des données hors ligne sensibles.
- **Situation de départ :** l’app conserve localement des tâches et un jeton de session; plusieurs appareils doivent voir les mêmes tâches.
- **Résultat attendu :** SQLite pour le cache/données locales, stockage sécurisé du secret, serveur et protocole de sync; pas de promesse de chiffrement implicite.
- **Consignes :** indiquez l’accès, la sauvegarde, la connexion et le rôle du serveur.
- **Critères de réussite :** les données persistées ne sont pas confondues avec la clé secrète ou la source distante.
- **Indice facultatif :** Drift ne décide pas qui gagne lors d’un conflit multi-appareil.

## 11. Résumé essentiel

- Les opérations asynchrones évitent d’attendre de façon bloquante, mais le coût CPU/exécuteur reste à vérifier.
- Filtrer, paginer et indexer selon le besoin réel; réutiliser une connexion à durée de vie maîtrisée.
- Sauvegarder une base ouverte demande une méthode cohérente; les restaurations doivent être migrables.
- SQLite/Drift n’apportent pas automatiquement chiffrement, synchronisation multi-appareil ou serveur central.

## 12. Lien avec le module suivant

Le parcours pédagogique est terminé. Les exercices finaux vont vous faire réutiliser les choix de portée, de schéma, de requêtes, de migrations et de flux sur un même petit produit.
