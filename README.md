# Audit de Performance du Catalogue Sakila : Projet Data Analysis & Business Intelligence

## 1. Contexte Métier
La société **Sakila** (location de films physiques) souhaite auditer son catalogue pour optimiser sa rentabilité et piloter sa stratégie d'acquisition face à l'évolution du marché.

## 2. Objectifs Stratégiques
- **Auditer la structure :** Analyser la volumétrie et la répartition de l'offre par genre et par classification (`rating`).
- **Mesurer la performance :** Identifier les catégories porteuses et celles qui pèsent inutilement sur les ressources.
- **Maîtriser les risques :** Évaluer la valeur financière immobilisée (`replacement_cost`) pour sécuriser les actifs critiques.
- **Rationaliser les prix :** Vérifier l'alignement des taux de location par rapport aux typologies de films.

## 3. Problématiques Identifiées (Pain Points)
- **Vision fragmentée :** Absence de structure relationnelle consolidée pour analyser globalement l'offre et la demande.
- **Infobésité :** Listes brutes interminables empêchant toute lecture rapide et décision managériale.
- **Manque de hiérarchisation :** Impossibilité d'isoler instantanément les Top/Flop segments pour cibler les actions correctives.

---

## 4. La Modélisation Relationnelle : Création du Socle de Données

### Le Défi des Données Dispersées
À l'origine, aucune table unique ne permettait de relier directement les films à leurs catégories respectives, rendant toute analyse globale impossible. Il a donc fallu concevoir une vue relationnelle consolidée en s'appuyant sur la table intermédiaire `film_category`.

### Le Choix des Jointures
* **Pour l'offre (Catalogue) :** Utilisation d'un `INNER JOIN` entre `film`, `film_category` et `category` pour isoler les films valides rattachés à un genre.
* **Pour l'activité (Transition vers l'Étape 2 & 3) :** Utilisation méthodique de `LEFT JOIN` partant de la table maîtresse (`film`) pour l'inventaire et les transactions, garantissant une vision exhaustive sans perte de données (gestion des films orphelins).

### Requête SQL : Constitution de la Table de Travail Initiale
```sql
SELECT 
    film.title AS 'Titre', 
    film.description, 
    film.rental_rate AS 'Taux de location',
    film.replacement_cost AS 'Coût de remplacement',
    film.rating AS 'Notation', 
    film.rental_duration AS 'Temps de location', 
    film_category.category_id AS 'Identifiant Categorie',
    category.name AS 'Genre de film'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id;

5. Étape 1 : Analyse de l'Offre et Volumétrie (Le Top 5 des Genres)
Pour structurer notre analyse sans noyer le management, nous nous concentrons exclusivement sur le Top 5 des catégories les plus représentées du catalogue :

Animation (66 films)

Action (64 films)

Children / Enfants (60 films)

Comedy / Comédie (58 films)

Classics / Classiques (57 films)

Interprétation Métier
Cette forte concentration montre qu'historiquement, la stratégie d'acquisition a largement favorisé ces cinq segments. Cependant, un volume élevé ne garantit pas la rentabilité ou l'appétence réelle des clients.

La limite logique identifiée : Ce premier niveau macro ne nous dit pas combien de fois les films ont été réellement loués. Cela nous pousse à faire évoluer notre modèle vers l'analyse de la demande (Étape 2).

Requête SQL : Le Top 5 des genres les plus représentés
SQL
SELECT 
    category.name AS 'Genre de film',
    COUNT(film.film_id) AS 'Nombre de films'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY COUNT(film.film_id) DESC
LIMIT 5;
6. Analyse Stratégique : Le Taux de Location Moyen (Top 5)
Analyse de la politique tarifaire à travers le Top 5 des catégories affichant le taux de location moyen le plus élevé :

Games : 3,25

Travel : 3,23

Sci-Fi : 3,21

Comedy : 3,16

Sport : 3,12

Interprétation Métier
Les genres les plus représentés en volume ne sont pas systématiquement ceux qui affichent les taux unitaires les plus forts. Des catégories comme Games ou Sci-Fi portent une valeur tarifaire supérieure, offrant un levier clair pour rééquilibrer la politique d'acquisition.

Requête SQL : Taux de location moyen par genre (Top 5)
SQL
SELECT
    category.name AS 'Genre de film',
    AVG(rental_rate) AS 'Moyenne taux de location'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY AVG(rental_rate) DESC
LIMIT 5;
7. Étape 2 : L'Analyse de la Demande et des Volumes (Top 5 des films les plus loués)
Pour dépasser la simple structure du catalogue, nous croisons la table maîtresse film avec l'inventaire physique (inventory) et l'historique des transactions (rental).

Résultat du Top 5 des films en volume de locations :
Bucket Brotherhood : 34 locations

Rocketeer Mother : 33 locations

Forward Temple : 32 locations

Juggernaut Gravitate : 32 locations

Ridgemont Submarine : 32 locations

Interprétation Métier
Ces films constituent les véritables locomotives de l'activité. Une gestion rigoureuse des stocks s'impose pour les protéger de toute rupture préjudiciable.

Requête SQL : Top 5 des films les plus loués
SQL
SELECT 
    film.film_id AS "Identifiant film",
    film.title AS "Titre",
    COUNT(rental.rental_id) AS "Nombre de locations"
FROM film
LEFT JOIN inventory 
    ON film.film_id = inventory.film_id
LEFT JOIN rental 
    ON rental.inventory_id = inventory.inventory_id
GROUP BY film.film_id, film.title
ORDER BY COUNT(rental.rental_id) DESC
LIMIT 5;
8. Étape 3 : La Consécration Financière (Top 5 du Chiffre d'Affaires)
Dernier palier de l'investigation : évaluer la contribution financière réelle du catalogue en intégrant la table des paiements (payment).

La Vigilance Technique (Le Piège des Jointures)
L'erreur à éviter : Relier les paiements via l'identifiant client (customer_id) aurait corrompu mathématiquement les montants en additionnant l'historique global des clients au lieu de cibler la transaction précise.

La bonne pratique : Remplacement par une liaison rigoureuse sur l'acte de location (rental_id).

Requête SQL : Top 5 du Chiffre d'Affaires par Film
SQL
SELECT 
    film.film_id AS "Identifiant film",
    film.title AS "Titre",
    SUM(payment.amount) AS "Montant"
FROM film
LEFT JOIN inventory 
    ON film.film_id = inventory.film_id
LEFT JOIN rental 
    ON rental.inventory_id = inventory.inventory_id
LEFT JOIN payment 
    ON payment.rental_id = rental.rental_id
GROUP BY film.film_id, film.title
ORDER BY SUM(payment.amount) DESC
LIMIT 5;
9. Conclusion & Posture Data Analyst
Ce projet démontre une démarche analytique complète de niveau professionnel :

Investigation progressive : Du diagnostic de l'offre (catalogue) vers l'analyse de l'usage (volumes), pour finir par la rentabilité financière (CA).

Rigueur technique : Maîtrise des jointures (INNER vs LEFT JOIN) et sécurisation des clés de liaison pour garantir l'intégrité absolue des données.

Traduction managériale : Transformation de requêtes SQL brutes en recommandations stratégiques concrètes pour la gestion de stock et l'optimisation des revenus.
