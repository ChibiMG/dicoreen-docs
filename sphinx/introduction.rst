Introduction
============

Sphinx utilise le langage RST.

Installation
------------

.. note::

    Python doit être installé et présent dans les variables d'environnement (PATH).

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

.. note::

    En cas de problème le projet peut etre nettoyé avec la commande ``make clean``.
    