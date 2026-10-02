# Contexte Métier : Audit et Optimisation du Catalogue de Films (Base Sakila)

## 1. Contexte Métier
La société **Sakila** (location de films physiques) souhaite auditer son catalogue pour optimiser sa rentabilité et piloter sa stratégie d'acquisition face à l'évolution du marché.

## 2. Objectifs Stratégiques
* **Auditer la structure :** Analyser la volumétrie et la répartition de l'offre par genre et par classification (`rating`).
* **Mesurer la performance :** Identifier les catégories porteuses et celles qui pèsent inutilement sur les ressources.
* **Maîtriser les risques :** Évaluer la valeur financière immobilisée (`replacement_cost`) pour sécuriser les actifs critiques.
* **Rationaliser les prix :** Vérifier l'alignement des taux de location par rapport aux typologies de films.

## 3. Problématiques Identifiées (Pain Points)
* **Vision fragmentée :** Absence de structure relationnelle consolidée pour analyser globalement l'offre et la demande.
* **Infobésité :** Listes brutes interminables empêchant toute lecture rapide et décision managériale.
* **Manque de hiérarchisation :** Impossibilité d'isoler instantanément les Top/Flop segments pour cibler les actions correctives.

## 4. La Modélisation Relationnelle : Création du Socle de Données

### Le Défi des Données Dispersées
À l'origine, aucune table unique ne permettait de relier directement les films à leurs catégories respectives, rendant toute analyse globale impossible. Il a donc fallu concevoir une vue relationnelle consolidée en s'appuyant sur la table intermédiaire `film_category`.

### Le Choix des Jointures (`INNER JOIN`)
Pour construire ce tableau de travail, nous avons privilégié l'utilisation d'un `INNER JOIN` plutôt qu'un `LEFT` ou `RIGHT JOIN`. Le choix est simple : seuls les films rattachés à une catégorie valide nous intéressent pour notre analyse métier. Cela permet d'exclure d'emblée toute anomalie de structure et d'optimiser les performances de la requête.

### Requête SQL : Constitution de la Table de Travail
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

## 5. Analyse des Objectifs Métier (Focus Top 5)

### A. Volumétrie : Le Top 5 des Genres de Films
Pour structurer notre analyse sans noyer le management, nous nous concentrons exclusivement sur le **Top 5** des catégories les plus représentées du catalogue. 

D'après les données extraites, le volume de films se concentre principalement sur les genres suivants :
* **Animation** (66 films)
* **Action** (64 films)
* **Children / Enfants** (60 films)
* **Comedy / Comédie** (58 films)
* **Classics / Classiques** (57 films)

#### Interprétation Métier
Cette forte concentration montre qu'historiquement, la stratégie d'acquisition de l'entreprise a largement favorisé ces cinq segments (notamment l'animation, l'action et le public jeunesse). 
Cependant, un volume élevé ne garantit pas la rentabilité ou l'appétence réelle des clients. La question stratégique est donc de vérifier si ces genres les plus présents sont également ceux qui performent le mieux et génèrent le plus de valeur pour l'entreprise.

#### Requête SQL : Le Top 5 des genres les plus représentés
```sql
SELECT 
    category.name AS 'Genre de film',
    COUNT(film.film_id) AS 'Nombre de films'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY 'Nombre de films' DESC
LIMIT 5;
