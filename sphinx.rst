Documentation avec Sphinx
=========================

Sphinx utilise le langage RST.
Cheat Sheet : https://thomas-cokelaer.info/tutorials/sphinx/rest_syntax.html#code-block-directive

Installation
------------

Prérequis : avoir installé Python et vérifier sa présence dans les variables d'environnement.

.. code-block::

    pip install -U sphinx

Création du projet
------------------

Créer le dossier ou sera mis en place le projet et y accèder en ligne de commande.

.. code-block::

    sphinx-quickstart

Génération
----------

.. code-block::

    make html
