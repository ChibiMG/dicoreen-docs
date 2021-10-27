Intégration de TypeORM
======================

.. note::

    PostgresQL doit être installé et avoir un base de données pour l'application.

.. tip::

    Postman peut être installé pour faire des requêtes sur l'API.

Utiliser TypeORM pour décrire les différentes entités de l'API. Nous allons stocker les données dans une base de données PostgreSQL.
Un ORM (Object Relational Mapping) permet de manipuler les données via des entités et non en passant par des requêtes sur la base.

Documentation : https://typeorm.io/#/

Installation
------------

.. code-block::

    yarn add @nestjs/typeorm typeorm node-postgres

Modifier le fichier ``app.module.ts`` :

.. code-block::
    :emphasize-lines: 7-16
    :linenos:
    :caption: app.module.ts

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

Création des entités
--------------------
Une entité doit être créé dans un module. Exemple, création de l'entité ``word`` :

.. code-block::

    nest g module theme

Exemple de contenu à mettre dans le fichier entité ``theme/theme.entity.ts`` :

.. code-block::
    :linenos:
    :caption: theme.entity.ts

    import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from 'typeorm';
    import { Word } from '../word/word.entity';

    @Entity()
    export class Theme {
        @PrimaryGeneratedColumn()
        id: number;

        // Theme's name
        @Column()
        theme: string;

        // Theme's image
        @Column({
            nullable: true,
        })
        image: string;

        //Theme's words bi-directional relation
        @ManyToMany(() => Word, word => word.themes)
        words: Word[];
    }

Le champ ``words`` est une relation *many to many* entre deux entités (https://typeorm.io/#/relations).
