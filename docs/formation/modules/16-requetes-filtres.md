**Navigation :** [← Module 15](./15-crud.md) · [Sommaire](../README.md) · [Module 17 →](./17-types-donnees.md)

# Module 16 — Rechercher et filtrer les données

## 1. Objectif pédagogique

Vous saurez composer des `SELECT` lisibles pour filtrer, combiner, rechercher, trier, limiter, paginer et compter, sans charger en Dart toutes les lignes par défaut.

## 2. Le problème à résoudre

Une liste de milliers de tâches ne doit afficher que les tâches ouvertes d’un projet donné, les plus récentes d’abord, par paquets de vingt. Charger toute la base puis filtrer dans un widget gaspille mémoire, travail et temps.

## 3. Explication du fonctionnement

Drift expose les colonnes générées comme des expressions SQL typées. `equals` teste l’égalité; les expressions numériques comme `isBiggerThanValue` et `isSmallerThanValue` comparent des seuils. Les expressions se combinent avec `&` (AND) et `|` (OR), pas `&&`/`||` de Dart. Une recherche exacte compare la valeur complète; une recherche partielle utilise `contains` ou `like`.

`orderBy` définit l’ordre, `limit(n, offset: k)` limite une fenêtre. La pagination par offset est simple mais les grands offsets peuvent devenir coûteux et des insertions entre deux pages peuvent décaler les résultats; une pagination par curseur (dernier identifiant/date + ordre stable) est souvent préférable pour de grandes listes. `isNull()` recherche les valeurs absentes; `equals(null)` n’exprime pas la même chose en SQL. Pour compter sans charger les lignes, utiliser une expression d’agrégation telle que `countAll()`.

Une requête personnalisée SQL est pertinente pour une fonction complexe ou un CTE difficile à exprimer dans l’API Dart. Il faut lier les paramètres, pas concaténer une saisie utilisateur dans la chaîne SQL. Avec une requête SQL personnalisée observée par Drift, indiquez aussi les tables lues (`readsFrom`) pour que les flux sachent quoi surveiller.

## 4. Petit extrait de code

```dart
final query = select(tasks)
  ..where((t) => t.projectId.equals(projectId) & t.done.equals(false))
  ..orderBy([(t) => OrderingTerm(
      expression: t.createdAt, mode: OrderingMode.desc)])
  ..limit(20, offset: page * 20);
final pageOfTasks = await query.get();
```

## 5. Explication ligne par ligne

- `select(tasks)` prépare une lecture de la table, sans exécuter immédiatement la requête.
- `projectId.equals(projectId)` et `done.equals(false)` forment une expression SQL AND grâce à `&`; le `projectId` externe est transmis comme valeur liée.
- `orderBy` trie par date de création décroissante (`desc`); une clé secondaire peut stabiliser l’ordre lorsque plusieurs dates sont identiques.
- `limit` demande vingt lignes et saute les lignes des pages précédentes; ce mode simple a les limites d’offset évoquées plus haut.
- `get()` exécute maintenant la requête et retourne un `Future<List<Task>>`.

Autres mécanismes : `title.equals(term)` correspond à un titre exact; `title.contains(term)` à une recherche partielle; `priority.isBiggerThanValue(2)` compare numériquement; `t.done.equals(false) | t.priority.equals(5)` exprime un OR; `dueDate.isNull()` cherche une date absente. Une requête par identifiant est un filtre d’égalité sur la clé primaire.

Pour compter : `final c = countAll(); final row = await (selectOnly(tasks)..addColumns([c])).getSingle(); final total = row.read(c)!;`. `selectOnly` demande une projection/agrégation plutôt que de matérialiser toutes les tâches.

## 6. Déroulement interne

Drift transforme ces expressions en SQL et passe leurs valeurs comme paramètres au moteur. SQLite peut exploiter un index compatible, filtre les lignes puis applique tri/limite. Seuls les résultats demandés sont convertis en objets Dart. Sans index utile, SQLite peut devoir parcourir une grande partie de la table; le module 19 explique comment décider d’en créer un.

## 7. Pourquoi cette approche ?

Filtrer et paginer au niveau de la base limite le volume transmis à Dart. Le constructeur typé réduit les erreurs de noms de colonnes et permet de composer des filtres. Un index ne rend pas toute requête gratuite; il doit être évalué avec le volume et l’ordre réellement demandés.

## 8. Erreur classique

**Erreur :** charger toutes les tâches avec `get()` puis filtrer dans `build`, ou écrire `a && b` sur des expressions Drift. **Correction :** composer les expressions dans la requête et utiliser `&`/`|`; limiter les résultats utiles et garder les accès I/O hors de `build`.

## 9. Questions de compréhension

1. Pourquoi `&` est-il utilisé plutôt que `&&` pour joindre deux expressions de colonnes?
2. Quelle différence existe entre `title.equals('Planifier')` et une recherche qui contient ce mot?
3. Pourquoi `isNull()` est-il nécessaire pour chercher une date absente?
4. Quel problème une pagination par offset peut-elle rencontrer si de nouvelles tâches sont insérées entre deux pages?

## 10. Petit exercice pratique

- **Objectif :** spécifier une requête de tâches terminées d’un projet.
- **Situation de départ :** la table possède `projectId`, `done`, `title` et `createdAt`.
- **Résultat attendu :** filtre projet ET terminée, tri récent d’abord, maximum dix résultats.
- **Consignes :** décrivez le `where`, l’ordre et la limite; choisissez une recherche exacte ou partielle pour un titre donné.
- **Critères de réussite :** expressions composées par `&`, tri décroissant et résultat borné.
- **Indice facultatif :** gardez la requête côté Drift au lieu de filtrer la liste complète après `get()`.

## 11. Résumé essentiel

- Le constructeur Drift décrit une requête; `get()` l’exécute une fois.
- `&` et `|` construisent AND et OR SQL; les comparaisons ont des méthodes d’expression dédiées.
- `orderBy`, `limit` et offset organisent la lecture; le curseur est souvent mieux adapté aux longues listes.
- Le comptage peut être fait par agrégation; les valeurs dynamiques doivent rester des paramètres liés.

## 12. Lien avec le module suivant

La sélection des lignes est comprise; nous allons choisir des représentations fiables pour les types métier tels que statuts, dates, listes et montants.
