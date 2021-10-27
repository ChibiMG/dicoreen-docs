Documentation de l'API avec Swagger
===================================

Swagger permet de concevoir, créer, documenter et utiliser les services Web RESTful.
Ici nous utilisons un module Swagger pour NestJS dans le but de générer automatiquement une documentation pour notre API.

L'accès en local à la documentation Swagger se fait par le lien http://localhost:3000/swagger.

.. tip::

    Il est possible, de récupérer le JSON généré par swagger. Pour cela, il suffit d'ajouter ``-json`` à la fin de la route Swagger (http://localhost:3000/swagger-json).


Installation
------------

.. code-block::

    yarn add @nestjs/swagger swagger-ui-express

Initialisation
--------------

Initialiser la documentation swagger dans le ``main.ts`` :

.. code-block::
    :emphasize-lines: 8-15
    :linenos:
    :caption: main.ts

    import { NestFactory } from '@nestjs/core';
    import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
    import { AppModule } from './app.module';

    async function bootstrap() {
        const app = await NestFactory.create(AppModule);

        const options = new DocumentBuilder()
            .setTitle('Dicoréen NestJS')
            .setDescription('Documentation de l\'API')
            .setVersion('1.0')
            .addTag('dicoreen')
            .build();
        const document = SwaggerModule.createDocument(app, options);
        SwaggerModule.setup('swagger', app, document);

        await app.listen(3000);
    }
    bootstrap();

Décoration des contrôleurs avec "@ApiTags"
------------------------------------------

L'annotation ``@ApiTags`` permet de catégoriser les controllers de modules.

Exemple, contenu de ``theme.controller.ts`` :

.. code-block::
    :emphasize-lines: 19
    :linenos:
    :caption: theme.controller.ts

    // import ...
    import { ApiTags } from '@nestjs/swagger';
    // ...

    @Crud({
        model: {
            type: Theme,
        },
        query: {
            join: {
                themes: {
                    eager: true,
                    allow: [],
                }
            }
        }
    })

    @ApiTags('themes')
    @Controller('themes')
    export class ThemeController {
        constructor(public service: ThemeService) {}
    }

Génération automatique des schémas
----------------------------------

Enregistrer le plugin Swagger dans la configuration de NestJS, fichier ``nest-cli.json`` :

.. code-block::
    :emphasize-lines: 4-6
    :linenos:
    :caption: nest-cli.json

    {
        "collection": "@nestjs/schematics",
        "sourceRoot": "src",
        "compilerOptions": {
            "plugins": ["@nestjs/swagger/plugin"]
        }
    }

Ajout du décorateur "@ApiProperty"
----------------------------------

L'annotation ``@ApiProperty`` est ajoutée pour générer les schémas lors de l'utilisation de relations entre des entités.

Exemple, contenu de ``theme.entity.js`` :

.. code-block::
    :emphasize-lines: 21
    :linenos:
    :caption: theme.entity.ts

    import { ApiProperty } from '@nestjsx/crud/lib/crud';
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
        @ApiProperty({ type: () => Word })
        @ManyToMany(() => Word, word => word.themes)
        words: Word[];
    }
