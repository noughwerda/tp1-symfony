# TP3 — Réponses aux questions

## Question 1 : Password Hasher et algorithme `auto`

Un **Password Hasher** est un composant qui transforme un mot de passe en une empreinte (hash) irréversible, avec un sel aléatoire. On ne peut pas retrouver le mot de passe à partir du hash : on compare seulement le hash du mot de passe saisi avec celui de la base.

Symfony utilise l'algorithme `auto` par défaut car il choisit automatiquement le meilleur algorithme disponible sur le serveur (bcrypt, argon2id...). Si l'algorithme recommandé change avec le temps, Symfony peut mettre à jour les hash existants sans modifier la configuration.

## Question 2 : `providers` et `UserProvider`

La section `providers` indique à Symfony où et comment charger les utilisateurs. Dans notre projet : l'entité `App\Entity\User`, recherchée par la propriété `email`.

Le **UserProvider** a deux rôles :
- charger l'utilisateur depuis la base de données à partir de son identifiant (l'email) lors de la connexion ;
- recharger l'utilisateur depuis la session à chaque requête.

## Question 3 : `plainPassword` au lieu de `password`

Le champ `plainPassword` n'existe pas en base de données. Il sert seulement à récupérer le mot de passe saisi dans le formulaire. Le contrôleur le hache avec `UserPasswordHasherInterface`, puis enregistre le résultat dans le champ `password`.

Si on stockait le mot de passe en clair : en cas de fuite de la base de données, tous les mots de passe seraient lisibles, et comme beaucoup de personnes réutilisent leur mot de passe sur d'autres sites, leurs autres comptes seraient aussi en danger.

## Question 4 : l'objet `app` dans Twig

`app` est une variable globale fournie automatiquement par Symfony à tous les templates Twig. Elle donne accès à l'utilisateur, à la requête, à la session et aux messages flash.

`app.user` vient du système de sécurité : c'est l'utilisateur actuellement connecté (notre entité `User`). Si personne n'est connecté, `app.user` vaut `null`.

## Question 5 : Authentification vs Autorisation

- **Authentification** : vérifier l'identité de l'utilisateur (« Qui es-tu ? »). Exemple : email + mot de passe sur la page de connexion.
- **Autorisation** : vérifier les droits de l'utilisateur connecté (« Que peux-tu faire ? »). Exemple : seul un utilisateur avec `ROLE_USER` peut accéder à `/articles/nouveau`.

L'authentification vient toujours en premier, l'autorisation ensuite.

## Question 6 : Hiérarchie de rôles

La hiérarchie de rôles permet de dire qu'un rôle inclut automatiquement les droits d'un autre rôle. Ainsi, un `ROLE_ADMIN` possède aussi tout ce que peut faire un `ROLE_USER`, sans devoir lui donner les deux rôles.

Configuration dans `config/packages/security.yaml` :

```yaml
security:
    role_hierarchy:
        ROLE_ADMIN: ROLE_USER
```