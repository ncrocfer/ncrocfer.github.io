---
title: "Créer une commande personnalisée pour Git"
date: 2015-08-03 23:32:43
categories: [Howtos]
tags: [git]
author: ncrocfer
---

Git est un formidable outil de versionning fournissant un nombre incroyable de fonctionnalités. Mais il peut arriver un cas particulier pour lequel aucune commande git n'apporte de solution : c'est pourquoi **git** propose à l'utilisateur de créer ses propres commandes personnalisées.

[--MORE--]

Il m'arrive par exemple de vouloir connaître le nombre de commits sur un projet. Pour cela je fais :

.. sourcecode:: bash

    $ git log --pretty=oneline | wc -l
    132

Ca marche mais c'est un peu long à taper à chaque fois. On peut évidemment faire un alias dans notre fichier ~/.bashrc, mais on peut également créer une commande git qui permette de faire la même chose.

Pour cela on crée un fichier dans **/usr/bin**, par exemple **git-count**. Le nom des commandes personnalisées doivent être séparées par des tirets et n'ont pas d'extension. On y place le code souhaité :

.. sourcecode:: bash

    #!/bin/bash
    git log --pretty=oneline | wc -l

N'oubliez pas de rendre ce fichier éxécutable (**chmod +x**). La commande est maintenant accessible depuis un repository local :

.. sourcecode:: bash

    $ git count
    132
