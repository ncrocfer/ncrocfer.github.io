---
title: "Séparer un entier tous les n chiffres"
date: 2015-09-14T23:19:17+02:00
aliases: [/2015/09/14/separer-un-entier-tous-les-n-chiffres/]
---

Sous Python, le découpage d'une chaîne de caractères en plusieurs parties s'effectue grâce à la méthode **split()**. Mais parfois cette dernière ne suffit pas : le découpage d'un entier par bloc de 3 caractères par exemple.

Dans ce cas il faudra en effet partir de la fin de l'entier : *2000000* deviendra *2 000 000*, *1234* deviendra *1 234*, etc.

<!--more-->

Ce type d'affichage est régulièrement utilisé sur certains sites pour les statistiques, comme ici chez [StackOverflow](http://stackoverflow.com/10m) :

{{% img-responsive "/images/split_int.png" %}}

Voici la méthode que j'utilise pour cela :

```python
def split_int(number, separator=' ', count=3):
    return separator.join(
        [str(number)[::-1][i:i+count] for i in range(0, len(str(number)), count)]
    )[::-1]
```

On peut l'utiliser pour reproduire l'affichage de StackOverflow :

```python
>>> split_int(10144157, ',')
10,144,157
```

Edit
====

J'ai reçu un [tweet](https://twitter.com/sam_et_max/status/644019728374235136) de **@SamEtMax** proposant une solution incluse dans la stdlib. Elle fonctionne grâce à la lib **locale** et sa méthode **format**, qui dispose de l'argument **grouping**.

L'avantage principale de cette méthode est qu'elle se base sur la localisation de l'utilisateur : en France nous séparons les milliers par un espace (1 024), les Anglais utilisent la virgule (1,024) par exemple.

La lib locale convertira l'entier directement sous le bon format :

```python
>>> import locale
>>> locale.setlocale(locale.LC_ALL, 'en_US.utf8')
'en_US.utf8'
>>> print(locale.format("%d", 1024, grouping=True))
1,024
>>> locale.setlocale(locale.LC_ALL, 'fr_FR.utf8')
'fr_FR.utf8'
>>> print(locale.format("%d", 1024, grouping=True))
1 024

```

Je ne connaissais pas et j'avoue que c'est beaucoup plus simple que ma fonction custom, je passerai par cette solution désormais :)

