# Cas Pratique SQL & Business Intelligence : Analyse du Catalogue de Films (Base Sakila)

## Contexte et Objectif
Ce cas pratique s'adresse aux Data Analysts et équipes Business Intelligence souhaitant exploiter la base relationnelle **Sakila**. L'objectif est d'extraire la structure du catalogue, de mesurer la volumétrie et d'analyser les indicateurs clés de performance (KPIs).

---

## 1. Modélisation Relationnelle : La Table de Liaison
Pour relier les tables `film` et `category` sans tomber dans des jointures en cascade complexes, on utilise la table intermédiaire `film_category`.

```sql
SELECT 
    film.title AS 'Titre', 
    film.description, 
    film.rental_rate AS 'Taux de location',
    film.replacement_cost AS 'Coût de remplacement',
    film.rating AS 'Notation', 
    film.rental_duration AS 'Temps de location', 
    film_category.category_id AS 'Identifiant de Categorie',
    category.name AS 'Genre de film'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id;


---

## 2. Indicateurs Clés et Requêtes Métier (Agrégations)
### A. Répartition des films par genre (Volumétrie)
SELECT 
    category.name AS 'Genre de film',
    COUNT(film.film_id) AS 'Nombre de films'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY 'Nombre de films' DESC;

### B. Segmentation par classification (Rating)
SELECT
    film.rating AS 'Notation',
    COUNT(film.film_id) AS 'Nombre de films'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY film.rating
ORDER BY 'Nombre de films' DESC;

### C. Analyse de la durée de location par genre
SELECT
    category.name AS 'Genre de film',
    SUM(film.rental_duration) AS 'Durée totale'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY SUM(rental_duration) DESC;

### D. Taux de location moyen par catégorie
SELECT
    category.name AS 'Genre de film',
    AVG(rental_rate) AS 'Moyenne taux de location'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY AVG(rental_rate) DESC;

### E. Évaluation financière du coût de remplacement
SELECT
    category.name AS 'Genre de film',
    SUM(replacement_cost) AS 'Coût de remplacement'
FROM film
INNER JOIN film_category
    ON film.film_id = film_category.film_id
INNER JOIN category
    ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY SUM(replacement_cost) DESC;
