Intégration de Multer pour la gestion des images
------------------------------------------------

Prérequis : avoir un bucket sur https://www.scaleway.com/fr/ et une clé API.
Quickstart Scaleway : https://www.scaleway.com/en/docs/storage/object/quickstart/
Tuto 1 : https://wanago.io/2020/08/03/api-nestjs-uploading-public-files-to-amazon-s3/

Installation
^^^^^^^^^^^^

.. code-block::

    npm i -D @types/multer

Mise en place
^^^^^^^^^^^^^

Créer un fichier ``.env`` à la racine du projet et ajouter le code ci dessous.
Il permet d'ajouter la configuration vers le bucket : la région, la clé d'accès, la clé secrète et le nom du bucket.

.. code-block::

    AWS_REGION=fr-par
    AWS_ACCESS_KEY_ID=*****
    AWS_SECRET_ACCESS_KEY=*****
    AWS_PUBLIC_BUCKET_NAME=dicoreen-bucket
    AWS_ENDPOINT=https://dicoreen-bucket.s3.fr-par.scw.cloud

Installer le module config permettant d'accèder au ConfigService :

.. code-block::

    yarn add @nestjs/config

Ajouter le module dans app.module.ts :

.. code-block::

    @Module({
        imports: [
            // ...
            ConfigModule.forRoot(),
            // ...
        ],
        controllers: [AppController],
        providers: [AppService],
    })

Ajouter le sdk pour se connecter à AWS (Amazon Web Services) :

.. code-block::

    yarn add aws-sdk @types/aws-sdk

Ajouter la configuration du sdk dans main.ts :

.. code-block::

    import { ConfigService } from '@nestjs/config';
    // ...
    import { config } from 'aws-sdk';
    // ...

    async function bootstrap() {
        // ...

        const configService = app.get(ConfigService);
        config.update({
            accessKeyId: configService.get('AWS_ACCESS_KEY_ID'),
            secretAccessKey: configService.get('AWS_SECRET_ACCESS_KEY'),
            region: configService.get('AWS_REGION'),
            s3BucketEndpoint: true,
        });

        await app.listen(3000);
    }
    bootstrap();

Ajouter UUID (Universally Unique IDentifier) pour avour un nom unique pour le fichier puis créer un service et un module pour la gestion des fichiers :

.. code-block::

    yarn add uuid @types/uuid
    nest g service file
    nest g module file

Ajouter le code ci-desspus dans file.service.ts pour envoyer le fichier vers le bucket :

.. code-block::

    // import ...
    import { v4 as uuid } from 'uuid';

    @Injectable()
    export class FileService {
    constructor(private readonly configService: ConfigService) {}
    
    async uploadFile(dataBuffer: Buffer) {
        const s3 = new S3({ endpoint: this.configService.get('AWS_ENDPOINT'), });
        // Renommer le fichier par un uuid unique
        const key = uuid();
        // Uploader le fichier sur le bucket
        await s3.upload({Bucket: this.configService.get('AWS_PUBLIC_BUCKET_NAME'), Body: dataBuffer, Key: key}).promise();
        // Renvoyer du nom du fichier
        return key;
    }
    }

Remarque : ``v4 as uuid`` renomme la variable.

Dans file.module.ts ajouter les imports, exports et providers :

.. code-block::

    // import ...

    @Module({
    imports: [ConfigModule],
    providers: [FileService],
    exports: [FileService],
    })
    export class FileModule {}

Dans theme.module.ts ajouter l'importe de FileModule :

.. code-block::

    // import ...
    import { FileModule } from 'src/file/file.module';

    @Module({
    imports: [
        TypeOrmModule.forFeature([Theme]),
        FileModule,
    ],
    providers: [ThemeService],
    controllers: [ThemeController]
    })
    export class ThemeModule {}

Dans theme.service.ts ajouter la méthode qui permet envoyer le fichier sur S3 et ajouter son chemin dans le theme.

.. code-block::

    // import ...
    import { FileService } from '../file/file.service';
    import { Repository } from 'typeorm';

    @Injectable()
    export class ThemeService extends TypeOrmCrudService<Theme> {
        constructor(
            @InjectRepository(Theme) private themeRepository : Repository<Theme>,
            private readonly fileService: FileService) {
            super(themeRepository);
        }

        /**
        * Envoyer le fichier sur S3 et ajouter son chemin dans le theme
        * @param theme 
        * @param imageBuffer 
        * @returns 
        */
        async addImage(theme: Theme, imageBuffer: Buffer) {
            const image = await this.fileService.uploadFile(imageBuffer);
            await this.themeRepository.update(theme.id, { image });
            return 'ok';
        }
    }

Enfin, ajouter la requête post d'ajout d'image dans theme.controller.ts :

.. code-block::

    import { Controller, HttpException, HttpStatus, Param, Post, UploadedFile, UseInterceptors } from '@nestjs/common';
    import { FileInterceptor } from '@nestjs/platform-express';
    // ...

    @Crud({
        model: {
            type: Theme,
        },
    })

    @ApiTags('themes')
    @Controller('themes')
    export class ThemeController {
        constructor(public service: ThemeService) {}

        @UseInterceptors(FileInterceptor('file'))
        @Post(':id/image')
        async uploadFile(
            @UploadedFile() file: Express.Multer.File,
            @Param('id') id: number,
        ) {
            var theme = await this.service.findOne(id);
            if(!theme) {
            //lancer une exception
            throw new HttpException('Not found', HttpStatus.NOT_FOUND);
            }
            //lancer le service et retourner "ok"
            return this.service.addImage(theme, file.buffer);
        }
    }

Il faut utiliser ce principe pour les images de l'entité Word.

Tests
^^^^^

Lien : https://github.com/nestjs/nest/tree/master/sample/29-file-upload

Le test à faire sur un cmd depuis un dossier contenant le fichier à envoyer :

.. code-block::

    curl http://localhost:3000/themes/6/image -F "file=@./pj_1.PNG" -F "name=test"
