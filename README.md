# Cas Pratique SQL & Business Intelligence : Analyse du Catalogue de Films (Base Sakila)

## 1. Contexte du projet
Dans le cadre de la gestion stratégique d'un catalogue de vidéoclub (basé sur la célèbre base de données relationnelle **Sakila**), l'objectif est d'analyser la structure, la volumétrie et les grandes tendances du catalogue de films. En tant que Data Analyst, il s'agit d'aider l'équipe de management à mieux comprendre la répartition de l'offre pour orienter les décisions d'achat, de gestion des stocks et de mise en avant des produits.

---

## 2. Le Premier Grand Défi : La construction du tableau de base (Modélisation relationnelle)
La difficulté majeure de ce type de base de données réside dans le fait que les films et leurs catégories ne sont pas directement liés par une simple colonne, mais via une **table de liaison** (`film_category`). 

Pour relever ce défi, la première étape indispensable a consisté à assembler les tables pour obtenir un tableau global et structuré, regroupant les informations clés de chaque film et son genre associé :

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

3. L'Analyse Métier & Les Indicateurs Clés (Requêtes Agrégées)
À partir de ce socle solide, nous avons basculé sur la deuxième partie de l'analyse en utilisant les fonctions d'agrégation (COUNT, SUM, AVG) associées à GROUP BY et ORDER BY pour faire parler les données :

A. Volumétrie : Nombre de films par genre

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
