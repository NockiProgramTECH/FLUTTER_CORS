**Navigation :** [← Module 11](./11-installation-initialisation.md) · [Sommaire](../README.md) · [Module 13 →](./13-contraintes.md)

# Module 12 — Les principaux types de colonnes

## 1. Objectif pédagogique

Vous saurez distinguer un type de colonne Drift, un type Dart et une représentation SQLite. Vous connaîtrez les types usuels, notamment booléens, dates et données binaires.

## 2. Le problème à résoudre

Une tâche peut contenir un titre, un statut, une date, une priorité et parfois des octets. Le type que l’on manipule en Dart n’est pas toujours le type de stockage que SQLite écrit dans son fichier.

## 3. Explication du fonctionnement

Drift fournit des builders de colonnes : `integer()`, `text()`, `boolean()`, `real()`, `dateTime()`, `blob()` et d’autres. Leurs types Dart et leurs types de stockage sont liés, mais pas identiques : par exemple un `bool` Dart est stocké comme un entier SQLite `0` ou `1`. En SQLite, les noms de type expriment surtout une affinité/forme de stockage; une valeur peut être convertie par les règles du moteur. Les options strictes sont distinctes.

| Builder / colonne Drift | Valeur Dart | Stockage SQLite courant | Usage et limites | Exemple de déclaration |
|---|---|---|---|---|
| `IntColumn` / `integer()` | `int` | `INTEGER` signé 64 bits | Compteur, identifiant, cents; limites de précision de `int` sur compilation JavaScript pour très grandes valeurs. Drift propose `int64()`/`BigInt` pour le besoin compatible Web. | `integer()()` |
| `TextColumn` / `text()` | `String` | `TEXT` | Titres, noms, statuts textuels; prévoir une limite/validation métier. | `text()()` |
| `BoolColumn` / `boolean()` | `bool` | `INTEGER` : 0/1 | Drapeau actif/terminé; Drift convertit entre `false/true` et `0/1`. | `boolean()()` |
| `RealColumn` / `real()` | `double` | `REAL` | Mesures approximatives; flottants impropres à l’égalité exacte des montants. | `real()()` |
| `DateTimeColumn` / `dateTime()` | `DateTime` | `INTEGER` ou `TEXT` selon option | SQLite n’a pas de type date dédié. Valeur par défaut Drift historique : timestamp Unix en secondes; l’option texte ISO-8601 est recommandée par la documentation actuelle pour les nouveaux projets et demande une migration si un stockage existant change. | `dateTime().nullable()()` |
| `BlobColumn` / `blob()` | `Uint8List` | `BLOB` | Octets, petits fichiers binaires ou données chiffrées; grosses images généralement mieux dans un fichier, la base conservant leur chemin et métadonnées. | `blob()()` |
| Colonne avec référence | généralement `int` (ou type de la clé cible) | type SQL de la clé + `REFERENCES` | Identifiant du projet référencé; l’intégrité dépend de l’activation des clés étrangères SQLite. | `integer().references(Projects, #id)()` |

Les colonnes Drift incluent aussi `int64()` pour les entiers 64 bits représentés en Dart `BigInt` (utile notamment sur Web), les colonnes d’énumération `intEnum`/`textEnum`, les types personnalisés via `customType` et les convertisseurs. Ce sont des colonnes typées Drift, pas de simples variables Dart.

## 4. Petit extrait de code

```dart
IntColumn get priority => integer()();
TextColumn get title => text()();
BoolColumn get done => boolean().withDefault(const Constant(false))();
RealColumn get ratio => real()();
DateTimeColumn get createdAt => dateTime()();
BlobColumn get payload => blob().nullable()();
```

## 5. Explication ligne par ligne

- `priority` est un `int` en Dart et une valeur de stockage `INTEGER` dans SQLite.
- `title` devient un `String`/`TEXT`.
- `done` est exposé comme booléen Dart; la base reçoit 0/1. Le défaut est ajouté dans le schéma SQL.
- `ratio` devient un `double`/`REAL`, avec les limites d’approximation des flottants.
- `createdAt` devient un `DateTime`; la représentation dépend de l’option Drift de stockage des dates.
- `payload` est un `Uint8List`/`BLOB`; `nullable()` autorise l’absence de contenu (`null`).
- Le double appel final `()()` termine la déclaration du builder en colonne; il est spécifique à la DSL Drift.

## 6. Déroulement interne

Pour `done`, une insertion passe de `true` en Dart à la représentation entière SQLite `1`; une lecture reconstruit un `bool`. Pour une date, Drift encode/décode selon l’option de génération : timestamp Unix par défaut historique (en secondes, sans conserver à lui seul un fuseau ni toute précision sous-seconde) ou texte ISO-8601 si configuré avec `store_date_time_values_as_text: true` dans `build.yaml` sous les options de `drift_dev`. Modifier cette option sur une base déjà utilisée peut nécessiter de convertir les lignes historiques; ce n’est pas une simple refactorisation Dart.

## 7. Pourquoi cette approche ?

Des colonnes typées rendent l’API lisible et permettent à Drift de convertir automatiquement entre stockage et valeurs Dart. Un BLOB évite la base pour les fichiers volumineux : son contenu gonfle la base, complique sauvegardes et accès. Le choix du type doit préserver le sens et la précision des données.

## 8. Erreur classique

**Erreur :** penser qu’une colonne `BoolColumn` est physiquement un type BOOLEAN natif SQLite, ou supposer que `DateTime` stocke automatiquement son fuseau. **Correction :** vérifier la représentation SQL et les options Drift; choisir et migrer la convention de date explicitement.

## 9. Questions de compréhension

1. Pourquoi `BoolColumn` peut-il retourner un `bool` tout en stockant `0` ou `1`?
2. Quelle donnée risque d’être perdue avec un timestamp Unix en secondes?
3. Pourquoi une image de plusieurs mégaoctets n’est-elle pas toujours une bonne valeur BLOB?
4. Pour un entier immense compilé vers JavaScript, quelle distinction Drift offre-t-il entre `integer()` et `int64()`?

## 10. Petit exercice pratique

- **Objectif :** choisir les colonnes d’une tâche.
- **Situation de départ :** une tâche contient un identifiant, un titre, un état terminé, un rapport mesuré, une date de création et une petite empreinte binaire.
- **Résultat attendu :** associer à chaque champ un type Drift et le type Dart correspondant.
- **Consignes :** utilisez le tableau; pour la date, indiquez qu’il faut fixer le format de stockage.
- **Critères de réussite :** booléen et date sont traités comme des conversions, pas comme des types SQLite magiques.
- **Indice facultatif :** l’empreinte est courte, donc un BLOB est plausible; une photo entière mérite une autre réflexion.

## 11. Résumé essentiel

- Les builders Drift décrivent des colonnes; les types Dart sont la vue utilisée par le code.
- `bool` se stocke en 0/1; `DateTime` en entier ou texte selon options; `Uint8List` en BLOB.
- Les timestamps par défaut historiques ont des limites de précision/fuseau; passer au texte peut exiger migration.
- Les types de colonnes, types Dart et représentations SQLite ne sont pas interchangeables.

## 12. Lien avec le module suivant

Les types disent quelles valeurs sont représentables; les contraintes disent quelles valeurs ont le droit d’entrer dans la base.
