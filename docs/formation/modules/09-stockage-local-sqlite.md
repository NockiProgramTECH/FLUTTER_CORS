**Navigation :** [← Module 8](./08-architecture.md) · [Sommaire](../README.md) · [Module 10 →](./10-fonctionnement-drift.md)

# Module 9 — Comprendre le stockage local et les bases de données

## 1. Objectif pédagogique

Vous saurez comparer mémoire, fichier, base locale et serveur distant, décrire SQLite et situer Drift dans le chemin de lecture et d’écriture.

## 2. Le problème à résoudre

Après redémarrage, une variable de compteur revient à sa valeur initiale. L’utilisateur, lui, s’attend à retrouver les projets et tâches enregistrés hors ligne.

## 3. Explication du fonctionnement

Une variable Dart vit en mémoire pendant l’exécution; la mémoire de processus est généralement perdue à la fermeture. Un fichier conserve des octets mais demande de définir soi-même format, validation, recherche, mises à jour concurrentes et cohérence. Un **serveur distant** stocke les données hors de l’appareil et peut servir plusieurs appareils/utilisateurs, mais demande réseau, authentification, synchronisation et gestion de conflits.

Une base **SQLite** est un moteur relationnel embarqué : tables, contraintes, requêtes SQL et transactions, généralement stockés dans un fichier sur l’appareil. **Drift** est une bibliothèque Dart/Flutter construite autour de bases relationnelles, souvent SQLite : elle décrit le schéma en Dart ou SQL, génère des classes/requêtes typées et fournit des requêtes réactives. Drift n’est pas SQLite et ne remplace pas le moteur; c’est l’interface et l’outillage qui aident à l’utiliser.

Une base relationnelle est préférable à un simple JSON lorsque l’application doit filtrer, trier, joindre projets et tâches, imposer des identifiants uniques ou modifier plusieurs lignes de façon fiable. JSON reste pratique pour une préférence petite et isolée. Le stockage local est rapide et hors ligne, mais l’utilisateur peut perdre l’appareil, désinstaller l’app ou restaurer une ancienne sauvegarde; il ne synchronise rien par lui-même et n’est pas chiffré automatiquement.

## 4. Petit extrait de code

```text
Interface → logique → repository/DAO → Drift → SQLite → fichier de l'appareil
Interface ← logique ← repository/DAO ← Drift ← SQLite ← fichier de l'appareil
```

## 5. Explication ligne par ligne

- Dans le premier sens, l’UI exprime une action; la logique délègue; Drift traduit une opération typée en SQL; SQLite applique les contraintes et écrit les pages de la base.
- Dans le sens retour, SQLite renvoie des lignes; Drift les convertit en valeurs/classes Dart; le dépôt expose le résultat; l’état de présentation actualise l’UI.
- Le fichier est un détail de stockage du moteur. L’application doit utiliser l’exécuteur et le chemin adaptés à sa plateforme, pas écrire directement dans le fichier pendant que SQLite l’utilise.

## 6. Déroulement interne

Une insertion doit traverser la couche logique jusqu’à SQLite; une lecture suit le chemin inverse. Une fermeture correcte de la connexion libère les ressources, mais ne supprime pas le fichier de base. Au prochain lancement, Drift rouvre le même emplacement et peut donc relire les lignes.

## 7. Pourquoi cette approche ?

SQLite permet de conserver des données structurées, de rechercher efficacement et d’assurer l’atomicité des opérations. Le coût est la conception de schéma et de migrations. Une base locale convient aux données propres à l’appareil et à un mode hors ligne; elle ne remplace pas un serveur quand il faut partager, sauvegarder centralement ou arbitrer des modifications entre appareils.

## 8. Erreur classique

**Erreur :** croire que Drift fournit automatiquement un compte utilisateur, une synchronisation distante ou un chiffrement. **Correction :** Drift s’occupe de l’accès et de la réactivité aux données; réseau, synchronisation, sécurité et sauvegardes doivent être conçus séparément.

## 9. Questions de compréhension

1. Pourquoi une variable globale ne suffit-elle pas à conserver des tâches après fermeture du processus?
2. Dans quel cas un fichier JSON peut-il rester un meilleur choix qu’une base?
3. Quelle partie exécute réellement les transactions SQL : Drift ou SQLite?
4. Pourquoi des tables Projet et Tâche sont-elles plus faciles à filtrer que des objets sérialisés dans un seul gros fichier?

## 10. Petit exercice pratique

- **Objectif :** choisir un support approprié.
- **Situation de départ :** l’app doit mémoriser un thème, des tâches avec statut/projet, et envoyer les mêmes tâches à plusieurs appareils.
- **Résultat attendu :** proposer un support pour chaque besoin et signaler ce qu’il ne fait pas automatiquement.
- **Consignes :** comparez préférence/fichier, SQLite+Drift et serveur distant; une donnée peut être copiée/synchronisée entre plusieurs supports.
- **Critères de réussite :** la base locale est distinguée du serveur et de la persistance en mémoire.
- **Indice facultatif :** demandez si les données sont relationnelles, volumineuses ou partagées.

## 11. Résumé essentiel

- Mémoire temporaire, fichier, SQLite et serveur ont des durées de vie et responsabilités différentes.
- SQLite est le moteur relationnel local; Drift est une bibliothèque d’accès typé et d’outillage.
- Une base relationnelle aide à filtrer, relier, contraindre et transactionner les données.
- Local ne signifie ni sauvegardé, ni synchronisé, ni chiffré.

## 12. Lien avec le module suivant

Nous avons situé les outils. Voyons maintenant les objets que Drift construit autour des tables, des lignes, des clés et du code généré.
