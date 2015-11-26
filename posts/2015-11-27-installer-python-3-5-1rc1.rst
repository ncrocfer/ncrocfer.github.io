---
title: "Installer Python 3.5.1rc1"
date: 2015-11-27 00:10:32
categories: [Howtos]
tags: [python]
author: ncrocfer
---

La version 3.5.1rc1 de Python est sortie cette semaine, incluant  `de nombreuses corrections <https://docs.python.org/3.5/whatsnew/changelog.html#python-3-5-1-release-candidate-1>`_.

Voici comment l'installer si vous souhaitez la tester. Sur mon laptop j'ai un répertoire dans lequel je place les sources des différentes versions de Python (**~/dev/python/src**), et un autre dans lequel je compile les binaires (**~/dev/python/bin**).

[--MORE--]

On commence par télécharger les sources :

.. sourcecode:: bash

    ncrocfer@home:~$ cd ~/dev/python/src
    ncrocfer@home:~/dev/python/src$ wget https://www.python.org/ftp/python/3.5.1/Python-3.5.1rc1.tgz
    ncrocfer@home:~/dev/python/src$ tar xzvf Python-3.5.1rc1.tgz
    ncrocfer@home:~/dev/python/src$ cd Python-3.5.1rc1/

On compile ensuite les sources, en choisissant le répertoire de destination souhaité :

.. sourcecode:: bash

    ncrocfer@home:~/dev/python/src/Python-3.5.1rc1$ ./configure --prefix="/home/ncrocfer/dev/python/bin/3.5.1rc1/"
    ncrocfer@home:~/dev/python/src/Python-3.5.1rc1$ make
    ncrocfer@home:~/dev/python/src/Python-3.5.1rc1$ make install

Vous pouvez désormais tester Python 3.5.1rc1 :

.. sourcecode:: bash

    ncrocfer@home:~$ cd ~/dev/python/bin/3.5.1rc1/bin/
    ncrocfer@home:~/dev/python/bin/3.5.1rc1/bin$ ./python3
    Python 3.5.1rc1 (default, Nov 27 2015, 00:32:21) 
    [GCC 4.8.4] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>>

A noter que cette méthode fonctionne évidemment pour les autres versions de Python. Utile si vous devez tester votre code sur une version spécifique de Python non packagée dans votre système de paquet.
