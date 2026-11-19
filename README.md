WRITEUP_SQL_Intro_HTB
Writeup — SQL Injection Fundamentals : requêtes SQL de base (HTB Academy)
Module : SQL Injection Fundamentals · Section 6/17Thème : interroger une base MariaDB/MySQL avec SELECT, WHERE, LIKE, ANDBase : employees · SGBD : MariaDB 10.7.3
Auteur : Mouhamed Lo
Objectif
Se connecter à une base de données distante, explorer sa structure (bases, tables, colonnes), puis construire des requêtes SELECT ciblées pour répondre à une question précise sur les employés.
Question finale : What is the last name of the employee whose first name starts with "Bar" AND who was hired on 1990-01-01?
Étape 1 — Connexion à la base
mysql -u root -h 154.57.164.81 -P 30138 -p --skip-ssl
# Mot de passe : password
​
Explication des options :
u root : l'utilisateur.
h 154.57.164.81 : l'adresse du serveur (host).
P 30138 : le port (P majuscule = port ; p minuscule = mot de passe).
p : demande le mot de passe de manière interactive.
-skip-ssl : désactive le TLS (nécessaire ici car le serveur ne le propose pas).
Le serveur répond : MariaDB 10.7.3. L'invite devient MariaDB [(none)]>.
Étape 2 — Lister les bases de données
SHOW DATABASES;
​
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
​
information_schema, mysql, performance_schema et sys sont les bases système. La base applicative est employees.
USE employees;
​
Étape 3 — Lister les tables (premier objectif)
SHOW TABLES;
​
+----------------------+
| Tables_in_employees  |
+----------------------+
| current_dept_emp     |
| departments          |
| dept_emp             |
| dept_emp_latest_date |
| dept_manager         |
| employees            |
| salaries             |
| titles               |
+----------------------+
8 rows in set
​
La table clé est employees.
Étape 4 — Comprendre la structure des données
On regarde d'abord une table simple, departments :
SELECT * FROM departments;
​
Elle contient 9 départements (dept_no, dept_name) : Customer Service, Development, Finance, Human Resources, Marketing, Production, Quality Management, Research, Sales.
Puis la table employees :
SELECT * FROM employees;
-- 654 rows in set
​
Les colonnes sont : emp_no, birth_date, first_name, last_name, gender, hire_date.
Étape 5 — La requête ciblée (et un piège utile)
On veut le nom de famille (last_name) de l'employé dont le prénom commence par « Bar » ET embauché le 1990‑01‑01.
Le piège du joker %
Premier réflexe, j'ai essayé :
SELECT * FROM employees WHERE first_name LIKE '%Bar';   -- renvoie Empty set !
​
C'est faux : '%Bar' signifie « se termine par Bar ». Il fallait inverser le joker :
SELECT * FROM employees WHERE first_name LIKE 'Bar%';
​
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
|  10227 | 1953-10-09 | Barton     | Mitchem   | M      | 1990-01-01 |
|  10395 | 1960-02-23 | Bartek     | Nastansky | F      | 1989-06-05 |
|  10601 | 1956-08-10 | Barton     | Soicher   | F      | 1986-02-21 |
+--------+------------+------------+-----------+--------+------------+
3 rows in set
​
Règle à retenir :
'Bar%' → commence par Bar
'%Bar' → finit par Bar
'%Bar%' → contient Bar
Ajouter la deuxième condition avec AND
On affine avec la date d'embauche :
SELECT last_name
FROM employees
WHERE first_name LIKE 'Bar%'
  AND hire_date = '1990-01-01';
​
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
|  10227 | 1953-10-09 | Barton     | Mitchem   | M      | 1990-01-01 |
+--------+------------+------------+-----------+--------+------------+
​
Explication, mot par mot :
SELECT last_name : on ne demande que le nom de famille.
WHERE : on filtre les lignes.
first_name LIKE 'Bar%' : prénom qui commence par « Bar ».
AND : les deux conditions doivent être vraies en même temps.
hire_date = '1990-01-01' : embauché exactement à cette date (format AAAA-MM-JJ, entre apostrophes).
 Réponse
last_name = Mitchem (Barton Mitchem, emp_no 10227, embauché le 1990‑01‑01).
Notions clés à retenir
SHOW DATABASES / USE / SHOW TABLES / DESCRIBE : les commandes d'exploration d'une base inconnue — le premier réflexe, comme une reconnaissance réseau.
SELECT ... FROM ... WHERE : la structure de toute requête de lecture.
LIKE + % : la recherche par motif. Le sens du % (début/fin/milieu) change tout — c'est le piège qui m'a d'abord donné un résultat vide.
AND / OR : combiner des conditions. AND = toutes vraies ; OR = au moins une.
Dates : format 'AAAA-MM-JJ' entre apostrophes.
Lien avec l'injection SQL
Ces fondations sont indispensables avant d'attaquer l'injection SQL : une injection consiste à détourner une requête comme celle‑ci. Exemple d'une requête de connexion vulnérable :
SELECT * FROM users WHERE username = '<saisie>' AND password = '<saisie>';
​
Un attaquant qui saisit ' OR '1'='1 rend la condition toujours vraie. On y reconnaît exactement le OR, les apostrophes et la logique WHERE vus ici. Maîtriser WHERE, AND, OR, LIKE et les apostrophes, c'est déjà comprendre le mécanisme de l'injection SQL.
Compétences mises en œuvre
Connexion à un SGBD distant (MariaDB/MySQL), exploration d'un schéma de base, requêtes SELECT avec filtrage (WHERE, LIKE, AND), compréhension du joker %, et fondations de l'injection SQL.
