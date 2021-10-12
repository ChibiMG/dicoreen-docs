Backend avec NestJS
===================

NestJS utilise le langage TypeScript.
Tuto 1 : https://www.premieroctet.com/blog/bootstraper-api-avec-nestjs
Documentation : https://docs.nestjs.com/

Installation du CLI
-------------------

Prérequis : avoir installé npm et yarn et vérifier leurs présences dans les variables d'environnement.

.. code-block::

    yarn global add @nestjs/cli

Création du projet
------------------

Créer le dossier ou sera mis en place le projet et y accèder en ligne de commande.

.. code-block::

    nest new dicoreen-back

Génération
----------

.. code-block::

    yarn start --watch

Le paramètre ``--watch`` permet de relancer automatiquement le serveur après des changements au sein des fichiers du projet.
Après le lancement, il est accessible sur http://localhost:3000.

Contenu
-------

* *main.ts* : ce fichier contient le point d'entrée de l'application, où l'on va récupérer le module principal, puis lancer le serveur sur un port particulier (3000 par défaut).
* *app.controller.ts* : un contrôleur contient la définition des routes pour un module en particulier. Ici le contrôleur ne contient qu'une route, et cette route effectue un appel vers un service et en retourne son résultat. La classe est décorée à l'aide du décorateur @Controller, qui prend en paramètre optionnel une chaîne de caractère, indiquant un préfixe à utiliser pour les routes du contrôleur.
* *app.service.ts* : un service contient la logique de récupération et de traitement de donnée, par exemple depuis une base de données.
* *app.module.ts* : un module exporte à la fois les contrôleurs utilisés (controllers), les services utilisés au sein de ce dernier (providers), les services à fournir pour une utilisation dans d'autres modules (exports) et les modules utilisés au sein des contrôleurs et des services (imports).

.. toctree::
   :maxdepth: 2
   :caption: Contenu :

   integration_typeorm
   com_bdd
   doc_swagger
   image_part.rst
