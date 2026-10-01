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
