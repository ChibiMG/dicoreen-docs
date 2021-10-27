Modules, controleurs et services
================================

Les modules
-----------

Les modules organisent la structure de l'application.

Exemple, création du module ``word`` :

.. code-block::

    nest g module theme

Exemple de contenu pour ``theme.module.ts`` :

.. code-block::
    :linenos:
    :caption: theme.module.ts

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

Les modules exportent à la fois les contrôleurs utilisés (``controllers``), les services utilisés au sein de ce dernier (``providers``), les services à fournir pour une utilisation dans d'autres modules (``exports``) et les modules utilisés au sein des contrôleurs et des services (``imports``).

Les services
------------

La communication avec la base de données se fera à l'aide de services pour chacun des modules. Les services utilisent les *repository* pour manipuler les entités.

Exemple, création du service ``theme`` :

.. code-block::

    nest g service theme

Exemple de contenu pour ``theme/theme.service.ts`` :

.. code-block::
    :linenos:
    :caption: theme.service.ts

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

Les services doivent être définit dans l'annotation ``@Module`` dans son champ ``providers``.

Les controleurs
---------------

Les controleurs servent à definir les routes de l'API et les actions associées.

Exemple, création du controleur ``theme`` :

.. code-block::

    nest g controller theme

Exemple de contenu pour ``theme/theme.controller.ts`` :

.. code-block::
    :linenos:
    :caption: theme.controller.ts

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

Les controleurs doivent être définit dans l'annotation ``@Module`` dans son champ ``controllers``.
