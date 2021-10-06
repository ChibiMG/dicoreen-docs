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

Création des entités
^^^^^^^^^^^^^^^^^^^^

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

    // Word's themes bi-directional relation
    @ManyToMany(() => Theme, theme => theme.words)
    @JoinTable()
    themes: Theme[];
    }