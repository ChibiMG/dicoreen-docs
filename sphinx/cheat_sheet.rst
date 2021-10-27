Cheat sheet
===========

Cheat sheet 1 : https://thomas-cokelaer.info/tutorials/sphinx/rest_syntax.html#code-block-directive

Cheat sheet 2 : https://docutils.sourceforge.io/docs/ref/rst/directives.html

Notes
-----

Ajout d'une note :

.. code-block::

    ..note::

        //Texte de la note

Il existe d'autre type de note tel que : "attention", "danger", "error", "important", "note", "tip", "warning"...

Parties de code
---------------

Ajout d'une partie de code :

.. code-block::

    .. code-block::
        //Personnalisation du code

        //code

Personnalisation du code
^^^^^^^^^^^^^^^^^^^^^^^^

Ajout d'un surlignage du code entre x lignes : ``:emphasize-lines: 4-6``.

Ajout des numéros de ligne du code : ``:linenos:``.

Ajout du nom du fichier du code : ``:caption: nest-cli.json``
