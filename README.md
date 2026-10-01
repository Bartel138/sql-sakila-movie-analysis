h1. Cas Pratique SQL & Business Intelligence : Analyse du Catalogue de Films (Base Sakila)

h2. 1. Contexte du projet
Dans le cadre de la gestion stratégique d'un catalogue de vidéoclub (basé sur la célèbre base de données relationnelle **Sakila**), l'objectif est d'analyser la structure, la volumétrie et les grandes tendances du catalogue de films. En tant que Data Analyst, il s'agit d'aider l'équipe de management à mieux comprendre la répartition de l'offre pour orienter les décisions d'achat, de gestion des stocks et de mise en avant des produits.

---

h2. 2. Le Premier Grand Défi : La construction du tableau de base (Modélisation relationnelle)
La difficulté majeure de ce type de base de données réside dans le fait que les films et leurs catégories ne sont pas directement liés par une simple colonne, mais via une **table de liaison** (`film_category`). 

Pour relever ce défi, la première étape indispensable a consisté à assembler les tables pour obtenir un tableau global et structuré, regroupant les informations clés de chaque film et son genre associé :

<pre><code class="sql">
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
</code></pre>

---

h2. 3. L'Analyse Métier & Les Indicateurs Clés (Requêtes Agrégées)
À partir de ce socle solide, nous avons basculé sur la deuxième partie de l'analyse en utilisant les fonctions d'agrégation (`COUNT`, `SUM`, `AVG`) associées à `GROUP BY` et `ORDER BY` pour faire parler les données :

h3. A. Volumétrie : Nombre de films par genre
<pre><code class="sql">
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
</code></pre>

h3. B. Classification par notation (`Rating`)
<pre><code class="sql">
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
</code></pre>

h3. C. Analyse de la durée potentielle de location
<pre><code class="sql">
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
</code></pre>

h3. D. Analyse tarifaire : Taux de location moyen
<pre><code class="sql">
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
</code></pre>

h3. E. Gestion des risques : Coût de remplacement
<pre><code class="sql">
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
</code></pre>

---

h2. 4. Conclusion & Choix Stratégiques
Ce cas met en lumière une approche analytique rigoureuse et professionnelle :
# *La modélisation :* Maîtrise des tables de liaison et des jointures multiples pour structurer l'information brute sans créer d'usines à gaz de jointures en cascade superflues.
# *L'analyse descriptive :* Exploitation des indicateurs clés pour orienter la stratégie de gestion du catalogue.

*Perspectives :* Les aspects financiers transactionnels (revenus réels) pourront être abordés ultérieurement via des vues SQL ou des tables intermédiaires optimisées pour garantir performance et propreté de code en entreprise.
