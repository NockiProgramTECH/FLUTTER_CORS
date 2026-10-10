**Navigation :** [← Sommaire](./README.md) · [Exercices finaux](./exercices-finaux.md) · [Références](./references.md)

# PARTIE IV — CORRIGÉS REGROUPÉS

> Ces corrigés donnent le raisonnement attendu, pas une unique solution syntaxique. Revenez aux modules si votre conception diffère mais respecte les mêmes mécanismes.

## Corrigés des exercices pratiques des modules

- **M01 :** la donnée est le booléen `done`; l’action la passe de faux à vrai; l’UI recalcule l’état de la case. Si la tâche doit être conservée au redémarrage, il faut aussi persister ce changement. Le dessin de la case n’est pas la donnée.
- **M02 :** un `StatefulWidget` avec le booléen dans son objet `State` suffit. L’appui le change dans `setState`, puis `build` lit la nouvelle valeur et choisit l’icône/décoration correspondante.
- **M03 :** même type/clé signifie que `State` peut être conservé; Flutter met à jour `widget`, appelle `didUpdateWidget`, puis `build`. Le brouillon ne disparaît pas automatiquement. Une clé/type différent peut entraîner un nouvel état.
- **M04 :** liste de tâches = persistant; filtre = état d’écran temporaire sauf restauration voulue; panneau = local; total terminé = dérivé de la liste filtrée, pas un compteur indépendant.
- **M05 :** le parent garde `count`; il donne le compteur au texte et donne un callback au bouton. Le callback demande l’incrément; mutation et `setState` restent dans le parent.
- **M06 :** A → `setState`; B → une dépendance partagée de portée application (Provider/Riverpod, ou `InheritedWidget` selon contexte); C → BLoC peut rendre événements/transitions traçables, Cubit reste possible si les méthodes sont suffisamment explicites. Le bon choix dépend de besoin d’audit et de complexité.
- **M07 :** préférence lue une fois → `Future`; liste observée → `Stream`. Un Stream peut émettre plusieurs instantanés; l’abonné doit être annulé quand son propriétaire disparaît.
- **M08 :** l’UI transmet le titre; le contrôleur/cas d’usage valide; repository délègue au DAO Drift; la base enregistre puis un résultat/flux revient. Le widget ne lance pas d’écriture dans `build`.
- **M09 :** thème isolé peut rester dans une préférence/fichier; tâches relationnelles dans SQLite locale; partage multi-appareil demande un serveur et synchronisation. Le serveur ne remplace pas forcément le cache local.
- **M10 :** `Tasks` décrit la table; `Task` est une ligne générée; `TasksCompanion` représente une écriture partielle; `_$AppDatabase` et `database.g.dart` sont générés. On édite la source, puis on régénère.
- **M11 :** vérifier `drift_dev` et `build_runner` sous dev_dependencies, les imports, le nom de `part`, l’annotation et le chemin du fichier; relancer build_runner. Le générateur produit la superclasse.
- **M12 :** identifiant/priorité = entier; titre = texte/String; statut booléen = bool Dart stocké 0/1; mesure approximative = double/REAL; date = `DateTime` avec option fixée; petite empreinte = `Uint8List`/BLOB. Utiliser `int64()`/BigInt si la grandeur Web l’exige.
- **M13 :** titre obligatoire avec validation/contrainte; statut obligatoire avec défaut initial; priorité non négative par CHECK; `projectId` nullable car l’absence de projet est autorisée. Un défaut ne rend pas une colonne nullable.
- **M14 :** `Tasks.projectId` référence `Projects.id`; ProjectTags porte `projectId` et `tagId`, avec PK composée. Cascade convient aux associations jetables; la suppression d’un projet exige un choix métier et test.
- **M15 :** insert renvoie un identifiant; lecture ponctuelle un Future de lignes; update/delete ont un `where` et retournent le nombre affecté. Upsert gère conflit insertion/mise à jour; ce n’est pas équivalent à un update par PK.
- **M16 :** appliquer le filtre `projectId == p AND status == done`, trier `createdAt` en décroissant, limiter à dix. Recherche exacte compare l’égalité; partielle utilise `contains`/`like` lié en paramètre.
- **M17 :** statut = chaîne stable ou converter; coût en cents = entier; date = DateTime avec format explicite; étiquettes recherchables = table liée et index adapté. JSON texte peut être acceptable pour une donnée opaque, pas pour une relation interrogée.
- **M18 :** mettre l’en-tête et ses lignes dans une transaction. Une exception pendant l’écriture annule le bloc; SQLite ne peut pas annuler un appel réseau extérieur.
- **M19 :** un candidat est `(projectId, done, createdAt)` si les requêtes commencent par projet puis statut et trient par date. Mesurer la cardinalité, l’ordre réel, la taille et coût en écriture; un index ne sert pas automatiquement les recherches par `done` seule.
- **M20 :** augmenter la version et ajouter la migration v1→v2. Priorité historique prend zéro (via défaut/migration appropriée). Test : démarrer avec une ligne v1, migrer, vérifier identifiant/titre/statut conservés et `priority == 0`.
- **M21 :** après INSERT via Drift, la table suivie est signalée; Drift relance le SELECT; le Stream émet la nouvelle liste; StreamBuilder reconstruit. Les écritures externes ne sont pas nécessairement observées.
- **M22 :** la base est l’unique source durable; filtre local dans `setState` ou un provider. Commands passent par contrôleur/repository; les tâches reviennent par `watch`. Toute approche peut convenir si elle ne duplique pas la liste.
- **M23 :** A → génération/build; B → PRAGMA FK et tests d’intégrité; C → migration depuis la vraie version installée; D → créer/conserver un Stream/Future en dehors de `build` et traiter chargement/erreur.
- **M24 :** démarrer à v1, insérer la tâche, appliquer migration, relire ligne et assertions sur titre et `done == false`. Une base vide ne vérifie aucune conservation historique.
- **M25 :** stocker le jeton via un mécanisme sécurisé adapté plutôt qu’en texte de tâche; conserver/cache des données dans SQLite; serveur distant avec règles de synchronisation arbitre et partage; sauvegarde cohérente et migration après restauration. Drift ne fournit pas de chiffrement ni de résolution de conflit par défaut.

## Corrigés du questionnaire de gestion d’état

1. La variable Dart change; il faut un mécanisme observé comme `setState`, notifier, provider, cubit ou émission de flux pour que la partie d’UI concernée relise l’état.
2. Un widget est une configuration immutable souvent recréée pendant `build`; Flutter réconcilie type/clé/position et peut associer le nouveau widget au même `Element` et au même objet `State`.
3. `initState` attache une première fois; `didUpdateWidget` remplace l’abonnement si la source reçue change; `dispose` annule le dernier abonnement. Une dépendance héritée qui change se traite dans `didChangeDependencies`.
4. Un total est fonction de la liste. Le stocker en double introduit la possibilité d’un désaccord après ajout/suppression; le calculer préserve une source de vérité unique.
5. La donnée courante descend en paramètre; l’enfant remonte une intention avec callback. Le propriétaire décide puis reconstruit.
6. `ChangeNotifier` conserve/annonce des changements; Provider expose et gère une valeur dans une portée widget et simplifie l’écoute. On peut utiliser l’un sans l’autre.
7. `Future` convient au résultat ponctuel; `Stream` aux émissions successives observées. L’export lit une fois; une fiche modifiée en direct pourrait être observée avec Stream.
8. Filtre et formulaire sont états de présentation temporaires; validation est logique métier; liste durable dans Drift derrière repository/DAO. On peut choisir Provider/Riverpod/BLoC selon la portée, sans confondre couche et stockage.

## Corrigés du questionnaire Drift/SQLite

1. SQLite est le moteur et stocke/exécute les opérations; Drift est une bibliothèque Dart qui décrit schéma, génère une API typée, exécute les requêtes et peut les observer.
2. Le builder Drift exprime le schéma, le type Dart exprime les valeurs manipulées, SQLite a sa représentation/affinité (`INTEGER`, `TEXT`, `REAL`, `BLOB`). Drift effectue les conversions configurées.
3. Le companion sait distinguer « ne pas toucher » (`Value.absent`) de « mettre à null » (`Value(null)`), impossible avec un simple champ nullable dans une mise à jour partielle.
4. Absence de filtre = toutes les lignes peuvent être affectées. Vérifier le `where`, la condition métier et le nombre de lignes touchées.
5. Insert crée et échoue normalement en conflit; `update(...).replace(row)` met à jour la ligne ciblée par PK; upsert insère ou met à jour selon une contrainte de conflit.
6. SQLite peut ne pas appliquer les FK sans `PRAGMA foreign_keys = ON`; il faut activer et tester le mécanisme sur la connexion.
7. L’index accélère possiblement recherche/tri alignés, mais coûte espace/écritures et dépend de l’ordre des colonnes et des requêtes réelles.
8. La classe Dart décrit la cible; la base sur appareil conserve son ancien SQL. Il faut une conversion explicite versionnée pour faire évoluer les lignes existantes.
9. `SchemaVerifier` peut partir d’un ancien schéma et comparer résultat après migration à la cible; un test neuf n’exécute pas le chemin ancien.
10. `get()` retourne une photographie ponctuelle via `Future`; `watch()` expose une suite de résultats via `Stream`.

## Corrigés des diagnostics

- **D1 :** le callback change la mémoire seulement. Mettre la mutation dans `setState(() => count++)` ou utiliser le mécanisme d’état de l’architecture choisie; Flutter pourra ensuite reconstruire le widget.
- **D2 :** `build_runner` ne trouve pas l’exécuteur/générateur Drift si `build_runner` et `drift_dev` ne sont pas des dev_dependencies compatibles. Les ajouter, vérifier `part`/annotation/imports et générer; `drift` seul est la bibliothèque d’exécution.
- **D3 :** il manque une migration et l’augmentation de `schemaVersion`. Le code généré connaît v2, mais le fichier déjà installé reste v1. Migrer/tester les données historiques, ne pas réinstaller en production.
- **D4 :** aucun `where`: l’écriture touche toutes les lignes. Ajouter le filtre sur `id`, vérifier les lignes affectées et couvrir ce cas par un test.
- **D5 :** pas nécessairement. Drift suit heuristiquement les écritures passées par ses API; une écriture externe ne le notifie pas automatiquement. Utiliser une seule couche d’écriture Drift ou appeler explicitement `notifyUpdates` si l’intégration externe est maîtrisée et les tables correctement déclarées.
- **D6 :** chaque `build` peut produire une nouvelle instance de Stream et remplacer l’abonnement du `StreamBuilder`. Construire le Stream dans `initState`, un provider stable ou un propriétaire équivalent, et le remplacer seulement si les paramètres de requête changent.

## Corrigés des choix de conception

- **C1 :** `setState` dans un `StatefulWidget` est proportionné : portée unique et durée temporaire. Un provider global ajoute une portée/dépendance non nécessaire.
- **C2 :** un état partagé au niveau assez haut pour tous les écrans, exposé via Provider/ChangeNotifier ou Riverpod. La préférence doit être persistée si elle doit survivre au redémarrage; le mécanisme de partage ne l’enregistre pas automatiquement.
- **C3 :** BLoC rend les événements reçus et transitions explicites, utile si traçabilité d’événements est requise. Cubit peut suffire si des méthodes `authorize`, `cancel` etc. donnent un flux clair et les événements n’ont pas besoin d’être séparés. Tester les transitions et ne pas choisir uniquement par mode.
- **C4 :** Drift/SQLite pour la source locale hors ligne et durable. Cela ne fournit pas en soi synchronisation, compte multi-appareil, sauvegarde distante ni résolution de conflits.
- **C5 :** `Future<T>` pour une lecture unique; si l’objet peut changer pendant l’affichage, un `Stream<T>`/`watch` adapté réémet et demande une UI qui suit le flux.

## Corrigés relations, contraintes et index

- **R1 :** la FK `Tasks.projectId` est côté plusieurs; une clé étrangère active vers `Projects.id` empêche une valeur sans parent. Si chaque tâche doit avoir un projet, colonne non nullable.
- **R2 :** `ProjectTags(projectId FK, tagId FK)`; PK composée `(projectId, tagId)` ou unicité équivalente empêche le doublon de relation. Les deux FK garantissent des parents existants si activées.
- **R3 :** `CASCADE` supprime les tâches si un projet et ses enfants n’ont pas d’existence autonome; `RESTRICT` bloque la suppression si des tâches doivent être traitées d’abord; `SET NULL` conserve les tâches non rattachées et exige que `projectId` soit nullable.
- **R4 :** candidat `(projectId, status, createdAt)`; l’ordre de tri peut nécessiter direction/configuration adaptée et le choix dépend de la cardinalité, des requêtes par statut seul, du volume et du plan SQLite. Mesurer plutôt que déclarer cet index universel.
- **R5 :** unicité composée `(userId, name)`; un index ordinaire accélère mais permet des doublons, donc contrainte `UNIQUE` ou index unique est nécessaire pour l’intégrité.

## Corrigés des migrations

- **MIG1 :** porter `schemaVersion` à 2; ajouter `dueDate` nullable avec migration (`m.addColumn` si applicable). Les anciennes lignes ont `NULL`; les tests vérifient leur titre et la nullité.
- **MIG2 :** SQLite doit donner une valeur à toutes les lignes existantes; un `NOT NULL` sans défaut/backfill peut échouer. Introduire `priority` avec défaut 0 ou reconstruire/transformer, vérifier données, puis test v1→v2 qui conserve les lignes et assigne 0.
- **MIG3 :** définir un mapping exhaustif `'open' → ...`, `'done' → ...`, prévoir valeurs inconnues, nouvelle contrainte, transformation (`columnTransformer` ou séquence de migration) et tester chaque ancienne valeur, y compris cas null/corrompu. Une simple conversion de type SQL peut être insuffisante.
- **MIG4 :** créer la base au schéma v1 avec plusieurs lignes réalistes; exécuter le chemin step-by-step jusqu’à v3; lire ensuite avec le schéma actuel et vérifier identifiant, titre, statut, cardinalité, nouvelles valeurs et contraintes. Le test v1→v3 valide que les versions intermédiaires sont enchaînées.

## Corrigés des flux réactifs

- **FL1 :** liste en direct → `watch`; export ponctuel → `get`/`Future`, pas un abonnement inutile.
- **FL2 :** action utilisateur → insertion Drift → notification de table observée → nouvel instantané → reconstruction de `StreamBuilder`.
- **FL3 :** la modification hors Drift ne passe pas par le mécanisme d’updates heuristique du watcher. Le meilleur contournement est de centraliser les écritures dans Drift; sinon, appeler `notifyUpdates` avec les tables/kinds concernés et tester sa bonne utilisation.
- **FL4 :** tâches dans SQLite via Drift; filtre dans un état temporaire `setState` ou provider. `setState` ne sauvegarde pas en stockage durable et disparaît avec la durée de vie de l’état.

## Corrigé guidé du mini-projet

Une solution conforme peut garder les fichiers et responsabilités proposés, mais l’essentiel est le flux :

1. **Modèle :** statut persistant en code stable, par exemple `open`, `inProgress`, `done`; ne pas changer le sens d’une valeur historique sans migration. Titre validé, dates et statut enregistrés.
2. **Base :** `AppDatabase` en singleton/injection de niveau application; `schemaVersion` commence à 1; fichier généré non édité manuellement.
3. **CRUD :** opérations asynchrones explicites; `UPDATE`/`DELETE` limités par identifiant; erreurs de contraintes et de connexion renvoyées à l’état de l’écran.
4. **Règles :** valider le titre dans le cas d’usage/repository et conserver une contrainte de base si la règle doit s’appliquer à tous les auteurs.
5. **Recherche/filtre :** construire un WHERE Drift; combiner projet et statut avec `&`; tri déterministe; pagination si volume; ajouter index après analyse des requêtes.
6. **Flux :** repository retourne `watchTasks`; StreamBuilder/StreamProvider garde un Stream stable; ajout/modification via Drift déclenche une nouvelle émission; l’UI traite attente, erreur et données.
7. **Relations :** FK côté tâche; activer `PRAGMA foreign_keys = ON`; décider entre `CASCADE`, `RESTRICT` ou `SET NULL`, avec tests.
8. **Migration :** v2 augmente `schemaVersion`; ajoute `priority` avec valeur historique déterministe; snapshots et `stepByStep` permettent de tester v1→v2 sans effacer la base.
9. **Tests :** base mémoire pour CRUD/contraintes; fixture v1 avec données pour migrations; flux avec séquence d’émissions; fichier temporaire fermé/réouvert pour prouver la persistance.

La priorité du projet n’est pas de choisir le gestionnaire « le plus célèbre », mais d’assurer un seul propriétaire pour chaque donnée, un chemin testable pour chaque opération et une migration démontrée sur des données anciennes.
