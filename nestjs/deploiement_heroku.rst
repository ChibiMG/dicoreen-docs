Déploiement du backend sur Heroku
=================================

Le projet a été déployé sur Heroku, un PaaS (Plateform as a Service) permettant de deployer des applications sur un cloud facilement.

.. note::

    Il faut avoir créer un compte sur le site https://www.heroku.com/

.. note::

    Il faut avoir déployé le projet sur Github.
    
Tuto 1 : https://dev.to/rosyshrestha/deploy-nestjs-typescript-app-to-heroku-27e

Création de l'application
-------------------------

Sur l'interface Heroku créer une nouvelle app :

- Choisir un nom
- Choisir une région (ici Europe)
- Choisir une méthode de déploiement, si le code est déployé sur github, il est possible de choisir le deploiement via un repository de cette plateforme.

Définition du port d'écoute
------------------------------

Heroku définit une variable d'environnement ``PORT``, numéro de port sur lequel l'application doit écouter.
Il est a modifier dans ``main.js`` du projet :

.. code-block::
    :caption: main.js

    await app.listen(process.env.PORT || 3000); 

Créer un nouveau fichier nommé ``Procfile`` à la racine du projet et ajouter le code ci-dessous. Il permet à Heroku de connaître la commande pour lancer l'application :

.. code-block::
    :caption: Procfile

    web: yarn start:prod

Définition des variables globales dans Heroku
---------------------------------------------

Sur l'interface heroku, **dans Settings > Config Vars > Reveal Config vars** ajouter les variables que contient le fichier ``.env`` de votre projet en les adaptant au contexte de mise en production si besoin.

Ajout d'une base de données
---------------------------

Sur Heroku, dans l'onglet *ressources*, il est possible d'ajouter un module Postgres au projet. Une fois ajouté, Heroku expose, dans les variables d'environnement de l'application, une URL contenant toutes les informations nécéssaire pour la connexion. Cette variable n'a plus qu'à être utilisé dans le fichier ``app.module.ts`` lors de la configuration de l'ORM.

.. code-block::
    :caption: app.module.ts
    :emphasize-lines: 3
    :linenos:

    TypeOrmModule.forRoot({
      type: 'postgres',
      url: process.env.DATABASE_URL || "postgres://postgres:abc@localhost:5432/dicoreen",
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
