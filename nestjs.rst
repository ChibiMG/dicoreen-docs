Backend avec NestJS
===================

.. NestJS utilise le langage TypeScript.
.. Tuto 1 : https://www.premieroctet.com/blog/bootstraper-api-avec-nestjs

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

Intégration de TypeORM
----------------------

Prérequis : avoir installer PostgreSQL et créer la base dedans (ici "dicoreen").
Suplément : installer Postman pour faire des requêtes.

Utiliser TypeORM pour décrire les différentes entités de l'API. Nous allons stocker les données dans une base de données PostgreSQL.
Documentation : https://typeorm.io/#/

Installation
^^^^^^^^^^^^

.. code-block::

    yarn add @nestjs/typeorm typeorm mysql

Modifier le fichier app.module.ts :

.. code-block::

    import { Module } from '@nestjs/common';
    import { AppService } from './app.service';
    import { TypeOrmModule } from '@nestjs/typeorm';

    @Module({
    imports: [
        TypeOrmModule.forRoot({
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        username: 'postgres',
        password: 'abc',
        database: 'dicoreen',
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: true,
        }),
    ],
    controllers: [],
    providers: [AppService],
    })
    export class AppModule {}

Création des entités word, theme et conjugation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block::

    nest g module word
    nest g module theme
    nest g module conjugation

Exemple, contenu de l'entité word :

.. code-block::

    import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from 'typeorm';
    import { Theme } from '../theme/theme.entity';

    @Entity()
    export class Word {
    @PrimaryGeneratedColumn()
    id: number;

    // Word in french
    @Column()
    french: string;

    // Word in korean
    @Column()
    korean: string;

    // Word's type (name, verb...)
    @Column()
    type: string;

    // Example using the word
    @Column()
    example: string;

    // Word's image
    @Column()
    image: string;

    // Word's themes
    @ManyToMany(() => Theme)
        @JoinTable()
        themes: Theme[];
    }

Communication avec la base de données
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

La communication avec la base de données se fera à l'aide de services pour chacun des modules.

Créer à la main de services :

.. code-block::

    nest g service word
    nest g service theme

Exemple, contenu de theme.service.ts :

.. code-block::

    import { Injectable } from '@nestjs/common';
    import { InjectRepository } from '@nestjs/typeorm';
    import { Repository } from 'typeorm';
    import { Theme } from './theme.entity';

    @Injectable()
    export class ThemeService {
        constructor(
            @InjectRepository(Theme)
            private readonly themeRepository: Repository<Theme>,
        ) {}
        
        create(theme: Theme): Promise<Theme> {
            return this.themeRepository.save(theme);
        }
        
        findAll(): Promise<Theme[]> {
            return this.themeRepository.find();
        }

        findOne(id: number): Promise<Theme> {
            return this.themeRepository.findOne(id);
        }
    }


Générer le controleur de theme :

.. code-block::

    nest g controller theme

Exemple, contenu de theme.controller.ts :

.. code-block::

    import { Body, Controller, Get, Param, Post } from '@nestjs/common';
    import { Theme } from './theme.entity';
    import { ThemeService } from './theme.service';

    @Controller('theme')
    export class ThemeController {
        constructor(private readonly themeService: ThemeService) {}

        @Get()
        getThemes() {
            return this.themeService.findAll();
        }

        @Post()
        createTheme(@Body() body: Theme) {
            return this.themeService.create(body);
        }

        @Get(':id')
        getTheme(@Param('id') id: number) {
            return this.themeService.findOne(id);
        }
    }

Dans theme.module.ts ne pas oublié d'ajouter la partie ``imports`` dans la partie ``@Module`` :

.. code-block::

    import { Module } from '@nestjs/common';
    import { ThemeService } from './theme.service';
    import { ThemeController } from './theme.controller';
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { Theme } from './theme.entity';

    @Module({
    imports: [TypeOrmModule.forFeature([Theme])],
    providers: [ThemeService],
    controllers: [ThemeController]
    })
    export class ThemeModule {}

Réaliser des tests de requêtes dans Postman, exemples :

* Get : http://localhost:3000/theme/
* Post : http://localhost:3000/theme/, Body (raw/JSON) : ``{"theme" : "transport"}``
