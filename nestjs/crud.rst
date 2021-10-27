Utilisation du module CRUD de NestJS
====================================

CRUD : Create Read Update Delete

En l'état, de l'API peut être fonctionelle si nous y ajoutons toutes les routes à la main, ce qui est fastidieux.
NestJS fournit un module qui gère tous ces aspects, rendant le code bien plus léger.

Documentation : https://github.com/nestjsx/crud

Installation
------------

.. code-block::

    yarn add @nestjsx/crud @nestjsx/crud-typeorm class-transformer class-validator

Importation dans les controllers
--------------------------------

Exemple, contenu de ``theme.controller.js`` :

.. code-block::
    :linenos:
    :caption: theme.controller.ts

    import { Controller } from '@nestjs/common';
    import { Crud } from '@nestjsx/crud';
    import { Theme } from './theme.entity';
    import { ThemeService } from './theme.service';

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

    @Controller('themes')
    export class ThemeController {
        constructor(public service: ThemeService) {}
    }

L'attribut ``eager`` définit un type de relation permettant de charger automatiquement les entités liées à l'entité utilisée (https://typeorm.io/#/eager-and-lazy-relations/eager-relations). L'attribut ``allow`` a été rajouter pour la relation *many to many* (tuto : https://medium.com/@rodrigo.tornaciole/nest-js-many-to-many-relationship-using-typeorm-and-crud-ec6ed79274f0).

.. note::
    Le nom de la variable ``service`` dans le constructeur des contrôleurs est important, un autre nom provoquerait une erreur.

Importation dans les services
-----------------------------

Exemple, contenu de ``theme.service.js`` :

.. code-block::
    :linenos:
    :caption: theme.service.ts

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

Réalisation de requêtes tests dans Postman
------------------------------------------

Exemples de requêtes :

* Get : http://localhost:3000/theme/
* Post : http://localhost:3000/theme/, Body (raw/JSON) : ``{"theme" : "transport"}``