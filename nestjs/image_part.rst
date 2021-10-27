Envoi d'images sur Amazon S3
============================

Nous souhaitons créer une route ``/themes/{id}/image`` qui permet à l'utilisateur d'envoyer une image pour un thème donné, et qui enregistre cette image dans un bucket S3.

Nous utilisons ici Scaleway comme fournisseur cloud à la place d'AWS. Le SDK AWS peut être utlisé avec les fournisseurs alternatifs.

.. note::

    Il faut créer un bucket sur https://www.scaleway.com/fr/ et une clé API. Il faut bien noter la clé privée donnée car celle-ci est unique et ne sera pas recommuniquée.

**Références :**

- Quickstart Scaleway : https://www.scaleway.com/en/docs/storage/object/quickstart/
- Tuto 1 : https://wanago.io/2020/08/03/api-nestjs-uploading-public-files-to-amazon-s3/

Configuration
-------------

Créer un fichier ``.env`` à la racine du projet et ajouter le code ci dessous.
Il permet d'ajouter la configuration vers le bucket : la région, la clé d'accès, la clé secrète, le nom du bucket et son lien d'accès.

.. code-block::
    :linenos:
    :caption: .env

    AWS_REGION=fr-par
    AWS_ACCESS_KEY_ID=*****
    AWS_SECRET_ACCESS_KEY=*****
    AWS_PUBLIC_BUCKET_NAME=dicoreen-dev
    AWS_ENDPOINT=https://s3.fr-par.scw.cloud

Installer le paquet config permettant d'accèder au ``ConfigService``. Ce service charge les variables contenues dans les fichiers ``.env`` et les rend accessible à travers le service ``ConfigService``.

.. code-block::

    yarn add @nestjs/config

Ajouter le module dans ``app.module.ts`` :

.. code-block::
    :emphasize-lines: 4
    :linenos:
    :caption: app.module.ts

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

Ajouter la configuration du sdk dans ``main.ts`` :

.. code-block::
    :emphasize-lines: 9-15
    :linenos:
    :caption: main.ts

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

Service et module de gestion des fichiers
-----------------------------------------

Créer un service et un module permettant de gérer les fichiers :

.. code-block::

    nest g service file
    nest g module file

Ajouter le code ci-dessous dans ``file.service.ts`` pour envoyer le fichier vers le bucket ou le supprimer :

.. code-block::
    :linenos:
    :caption: file.service.ts

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

        async removeFile(key: string) {
            const s3 = new S3({ endpoint: this.configService.get('AWS_ENDPOINT'), });
            //Supprimer le fichier sur le bucket
            await s3.deleteObject({Bucket: this.configService.get('AWS_PUBLIC_BUCKET_NAME'), Key: key}).promise();
        }
    }

.. note::
    Nous utilisons un UUID (Universally Unique IDentifier) comme clé pour les fichiers dans le bucket, c.a.d un identifiant unique généré aléatoirement lors de l'envoi du fichier. Il est nécéssaire d'installer une bibliothèque supplémentaire pour les générer : ``yarn add uuid @types/uuid``.

.. note::
     ``v4 as uuid`` correspond au renommement la variable.

Dans ``file.module.ts`` ajouter les imports, exports et providers qui conviennent pour utiliser le service de gestion de fichier :

.. code-block::
    :linenos:
    :caption: file.module.ts

    // import ...

    @Module({
    imports: [ConfigModule],
    providers: [FileService],
    exports: [FileService],
    })
    export class FileModule {}

Implementation pour l'entité *Theme*
------------------------------------

.. note::
    
    Il faut installer le paquet ``@types/multer`` pour pouvoir manipuler les fichiers envoyés en HTTP.

Dans ``theme.module.ts`` ajouter l'import de ``FileModule`` :

.. code-block::
    :emphasize-lines: 7
    :linenos:
    :caption: theme.module.ts

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

Dans ``theme.service.ts`` ajouter la méthode qui permet envoyer le fichier sur S3 et ajouter son chemin dans le theme.

.. code-block::
    :emphasize-lines: 21-26
    :linenos:
    :caption: theme.service.ts

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
        * Envoyer le fichier sur S3
        * Modifier le theme
        * Sauvegarder la nouvelle version du theme
        * @param theme 
        * @param imageBuffer 
        * @returns 
        */
        async addImage(theme: Theme, imageBuffer: Buffer) {
            const image = await this.fileService.uploadFile(imageBuffer);
            theme.image = image;
            await this.themeRepository.save(theme);
            return 'ok';
        }
    }

Enfin, ajouter la requête post d'ajout d'image dans ``theme.controller.ts`` :

.. code-block::
    :emphasize-lines: 24-37
    :linenos:
    :caption: theme.controller.ts

    import { Controller, HttpException, HttpStatus, Param, Post, UploadedFile, UseInterceptors } from '@nestjs/common';
    import { FileInterceptor } from '@nestjs/platform-express';
    // ...

    @Crud({
        model: {
            type: Theme,
        },
        query: {
            join: {
                words: {
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

Suppression et remplacement des images
--------------------------------------

Créer un fichier ``theme.subscriber.ts`` pour ajouter des fonctions annexes lors de l'executions de certaines fonctions CRUD.
Ici nous avons créé des fonctions pour la suppression d'un thème et la modification.

.. code-block::
    :linenos:
    :caption: theme.subscriber.ts

    // import ...

    @Injectable()
    export class ThemeSubscriber implements EntitySubscriberInterface<Theme> {
        constructor(@InjectConnection() readonly connection: Connection, private readonly fileService: FileService) {
            connection.subscribers.push(this);
        }

        listenTo() {
            return Theme;
        }

        afterRemove(event: RemoveEvent<Theme>) {
            if(event.databaseEntity.image != undefined) {
                this.fileService.removeFile(event.entity.image);
            }
        }

        afterUpdate(event: UpdateEvent<Theme>) {
            if (event.updatedColumns.find(element => element.propertyName == "image") != undefined && event.databaseEntity.image != undefined) {
            this.fileService.removeFile(event.databaseEntity.image);
            }
        }
    }

Ajouter ``theme.subscriber.ts`` au providers du fichier ``theme.module.ts`` :

.. code-block::
    :emphasize-lines: 8
    :linenos:
    :caption: theme.module.ts

    // import ...

    @Module({
    imports: [
        TypeOrmModule.forFeature([Theme]),
        FileModule,
    ],
    providers: [ThemeService, ThemeSubscriber],
    controllers: [ThemeController]
    })
    export class ThemeModule {}


Il faut utiliser ces mêmes principes pour les images de l'entité Word.

Tests
-----

Lien : https://github.com/nestjs/nest/tree/master/sample/29-file-upload

Le test à faire sur un cmd depuis un dossier contenant le fichier à envoyer :

.. code-block::

    curl http://localhost:3000/themes/6/image -F "file=@./pj_1.PNG" -F "name=test"
