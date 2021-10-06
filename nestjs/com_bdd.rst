Communication avec la base de données
-------------------------------------

La communication avec la base de données se fera à l'aide de services pour chacun des modules.

Création des services
^^^^^^^^^^^^^^^^^^^^^

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


Génération des controleurs
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block::

    nest g controller theme
    nest g controller word

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

Importation du module
^^^^^^^^^^^^^^^^^^^^^

Dans les modules ne pas oublié d'ajouter la partie ``imports`` dans la partie ``@Module``.
Exemple, contenu de theme.module.ts :

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

Remarque : la même chose est à faire pour chaque module.

Réaliser des tests de requêtes dans Postman
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Exemples :

* Get : http://localhost:3000/theme/
* Post : http://localhost:3000/theme/, Body (raw/JSON) : ``{"theme" : "transport"}``

Utilisation du module CRUD de NestJS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CRUD : Create Read Update Delete

En l'état, de l'API peut être fonctionelle si on y ajoute toutes les routes à la main, ce qui est fastidieux.
NestJS fournit un module qui gère tous ces aspects, rendant le code bien plus léger.

Installation :

.. code-block::

    yarn add @nestjsx/crud @nestjsx/crud-typeorm class-transformer class-validator

Exemple, contenu de theme.controller.js :

.. code-block::

    import { Controller } from '@nestjs/common';
    import { Crud } from '@nestjsx/crud';
    import { Theme } from './theme.entity';
    import { ThemeService } from './theme.service';

    @Crud({
        model: {
            type: Theme,
        },
    })

    @Controller('themes')
    export class ThemeController {
        constructor(public service: ThemeService) {}
    }

Remarque : Le nom de la variable ``service`` dans le constructeur des contrôleurs est important, un autre nom provoquerait une erreur.

Exemple, contenu de theme.service.js :

.. code-block::

    import { Injectable } from '@nestjs/common';
    import { InjectRepository } from '@nestjs/typeorm';
    import { TypeOrmCrudService } from '@nestjsx/crud-typeorm';
    import { Theme } from './theme.entity';

    @Injectable()
    export class ThemeService extends TypeOrmCrudService<Theme> {
        constructor(@InjectRepository(Theme) theme) {
            super(theme);
        }
    }