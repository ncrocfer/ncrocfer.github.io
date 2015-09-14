---
title: "Séparer un entier tous les n chiffres"
date: 2015-09-14 23:19:17
categories: [Howtos]
tags: [python]
author: ncrocfer
---

Sous Python, le découpage d'une chaîne de caractères en plusieurs parties s'effectue grâce à la méthode **split()**. Mais parfois cette dernière ne suffit pas : le découpage d'un entier par bloc de 3 caractères par exemple.

Dans ce cas il faudra en effet partir de la fin de l'entier : *2000000* deviendra *2 000 000*, *1234* deviendra *1 234*, etc.

[--MORE--]

Ce type d'affichage est régulièrement utilisé sur certains sites pour les statistiques, comme ici chez `StackOverflow <http://stackoverflow.com/10m>`_ :

.. image:: /images/split_int.png
  :align: center

Voici la méthode que j'utilise pour cela :

.. sourcecode:: python

    def split_int(number, separator=' ', count=3):
        return separator.join(
            [str(number)[::-1][i:i+count] for i in range(0, len(str(number)), count)]
        )[::-1]

On peut l'utiliser pour reproduire l'affichage de StackOverflow :

.. sourcecode:: python

    >>> split_int(10144157, ',')
    10,144,157
