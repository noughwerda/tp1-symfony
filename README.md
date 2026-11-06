# TP1 — Introduction au Framework Symfony 7.4

**Module :** Développement Web — Framework PHP
**Auteur :** nourwerda

Ce dépôt contient le travail du TP1 : contrôleurs et routes, templates Twig, Doctrine ORM et mini-projet de gestion de tâches.

---

## Question 1 — `symfony check:requirements`

**Résultat obtenu sur ma machine (Ubuntu 24.04) :**
 [OK]                                         
 Your system is ready to run Symfony projects 

**Avertissements affichés et impact :**

 « Aucun avertissement : toutes les vérifications obligatoires sont validées ».

**Explication générale :**

- Les vérifications obligatoires doivent toutes être validées, sinon Symfony ne fonctionne pas correctement.
- Les avertissements (recommandations)ne bloquent pas l'application. Ils signalent une configuration PHP qui peut **réduire les performances** ou limiter certaines fonctions.


## Question 2 — Rôle du dossier `public/`

- Le dossier `public/` est la racine web: c'est le seul dossier accessible depuis le navigateur
- Il contient `index.php` (le contrôleur frontal) et les fichiers publics (CSS, JS, images).
- `index.php` s'y trouve car toutes les requêtes passent par ce fichier unique, qui démarre Symfony puis lance le routage.
- C'est aussi une question de sécurité : le code (`src/`), la configuration (`config/`), le fichier `.env` (mots de passe) et `vendor/` restent hors de portée des visiteurs.



## Question 3 — Variable qui contrôle le mode de l'application

- La variable est APP_ENV  définie dans le fichier `.env`.
- Valeurs principales :
  - `dev` : développement. Barre de débogage, profiler, erreurs détaillées.
  - `prod` : production. Cache optimisé, pas de détails d'erreur affichés aux visiteurs.
- Ne pas confondre avec `DATABASE_URL`, qui contient seulement la connexion à la base de données.

---

## Question 4 — Que se passe-t-il avec `/profil/abc` ?

- On obtient une erreur **404 Not Found** (« No route found »).
- **Pourquoi :** la route `/profil/{id}` a la contrainte `requirements: ['id' => '\d+']`. Le motif `\d+` accepte **uniquement des chiffres**. La valeur `abc` ne correspond pas, donc aucune route ne correspond à l'URL.
- La valeur par défaut (`defaults: ['id' => 1]`) sert seulement quand l'id est **absent** (`/profil`). Elle ne remplace pas une valeur **invalide**.

---

## Question 5 — Nombre de routes définies

- Résultat de `php bin/console debug:router` : 

 app_accueil                ANY      /accueil                           
  app_bonjour                ANY      /bonjour/{prenom}                  
  app_profil                 ANY      /profil/{id}                       
  app_article_nouveau        ANY      /articles/nouveau                  
  app_articles               ANY      /articles                          
  app_article_detail         ANY      /articles/{id}                     
  app_taches                 ANY      /taches                            
  app_tache_ajouter          ANY      /taches/ajouter                    
  app_tache_detail           ANY      /taches/{id}                       
 -------------------------- -------- ----------------------------------- 
- **Routes que j'ai créées** ( app_accueil                ANY      /accueil                           
  app_bonjour                ANY      /bonjour/{prenom}                  
  app_profil                 ANY      /profil/{id}                       
  app_article_nouveau        ANY      /articles/nouveau                  
  app_articles               ANY      /articles                          
  app_article_detail         ANY      /articles/{id}                     
  app_taches                 ANY      /taches                            
  app_tache_ajouter          ANY      /taches/ajouter                    
  app_tache_detail           ANY      /taches/{id} 
- 

---

## Question 6 — `loop.index` et `loop.index0`

- `loop.index` est le numéro de l'itération en cours dans la boucle `{% for %}`, qui commence à 1 (1, 2, 3...).
- `loop.index0` est le même compteur, mais il commence à 0 (0, 1, 2...).
- Différence : **1 de décalage**. `loop.index` convient pour **afficher** un numéro à l'utilisateur, `loop.index0` pour travailler avec des **positions de tableau**.

---

## Question 7 — Fichier de migration

- La requête contenue dans `migrations/` est un **`CREATE TABLE article`**. Elle **crée la table `article`** :

```sql
CREATE TABLE article (
  id INT AUTO_INCREMENT NOT NULL,
  titre VARCHAR(255) NOT NULL,
  contenu LONGTEXT NOT NULL,
  auteur VARCHAR(100) NOT NULL,
  date_creation DATETIME NOT NULL,
  publie TINYINT(1) NOT NULL,
  PRIMARY KEY(id)
) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB
```

- Colonnes : `id` (clé primaire auto-incrémentée), `titre`, `contenu`, `auteur`, `date_creation`, `publie` (booléen MySQL).
- La méthode `up()` **exécute** la migration, la méthode `down()` l'**annule** (supprime la table).
- Doctrine transforme `dateCreation` (PHP) en `date_creation` (MySQL).
- Ma migration peut aussi contenir un `CREATE TABLE messenger_messages` (table du composant Messenger).

---

## Question 8 — ParamConverter

- Le **ParamConverter** (aujourd'hui appelé **EntityValueResolver**) **convertit automatiquement** le paramètre `{id}` de l'URL en **objet** `Article`.
- Symfony fait lui-même la recherche par id. Si l'article n'existe pas, il renvoie automatiquement une **404**.
- **Avantages** par rapport à une recherche manuelle avec le Repository :
  - moins de code dans le contrôleur
  - pas besoin d'écrire le test « article introuvable » + `createNotFoundException`
  - contrôleur plus lisible

Exemple de recherche manuelle :

```php
public function detail(int $id, ArticleRepository $repo): Response
{
    $article = $repo->find($id);
    if (!$article) {
        throw $this->createNotFoundException('Article introuvable');
    }
    return $this->render('articles/detail.html.twig', ['article' => $article]);
}
```