Documentation de l'API avec Swagger
-----------------------------------

Installation
^^^^^^^^^^^^

.. code-block::

    yarn add @nestjs/swagger swagger-ui-express

Initialisation
^^^^^^^^^^^^^^

Contenu de main.ts :

.. code-block::

    import { NestFactory } from '@nestjs/core';
    import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
    import { AppModule } from './app.module';

    async function bootstrap() {
        const app = await NestFactory.create(AppModule);

        const options = new DocumentBuilder()
            .setTitle('Tuto nest')
            .setDescription('API articles et auteurs')
            .setVersion('1.0')
            .addTag('tuto-nest')
            .build();
        const document = SwaggerModule.createDocument(app, options);
        SwaggerModule.setup('swagger', app, document);

        await app.listen(3000);
    }
    bootstrap();

Décoration des contrôleurs avec "@ApiTags"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Exemple, contenu de theme.controller.ts :

.. code-block::

    import { Controller } from '@nestjs/common';
    import { ApiTags } from '@nestjs/swagger';
    import { Crud } from '@nestjsx/crud';
    import { Theme } from './theme.entity';
    import { ThemeService } from './theme.service';

    @Crud({
        model: {
            type: Theme,
        },
    })

    @ApiTags('themes')
    @Controller('themes')
    export class ThemeController {
        constructor(public service: ThemeService) {}
    }

Génération automatique des schémas
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Contenu du fichier nest-cli.json :

.. code-block::

    {
        "collection": "@nestjs/schematics",
        "sourceRoot": "src",
        "compilerOptions": {
            "plugins": ["@nestjs/swagger/plugin"]
        }
    }

Ajout du décorateur ``@ApiProperty`` pour les relations Many to Many de nos entités.
Exemple, contenu de theme.entity.js :

.. code-block::

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

Voir la documentation de L'API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

http://localhost:3000/swagger

Remarque : Il est possible, de récupérer le JSON généré par swagger. Pour cela, il suffit d'ajouter ``-json`` à la fin de la route Swagger (http://localhost:3000/swagger-json).
