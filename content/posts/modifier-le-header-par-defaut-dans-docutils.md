---
title: "Modifier le header par défaut dans docutils"
date: 2015-08-05T00:11:35+02:00
aliases: [/2015/08/05/modifier-le-header-par-defaut-dans-docutils/]
---

Je travaille actuellement sur un [générateur statique de blog](https://github.com/ncrocfer/radric) qui génère les pages HTML à partir de la syntaxe **reStructuredText**.

J'utilise pour cela **docutils**, et par défaut les headers (h1, h2, h3...) sont générés par ordre croissant. Si il détecte un h1, alors un header de niveau moindre deviendra h2, ensuite h3, etc.

<!--more-->

Malheureusement dans mon thème, le titre de chaque post n'est pas généré par docutils, mais apparaît directement dans un template en tant que h1. Ce qui fait que chaque titre généré par docutils par la suite sera également un h1.

Pour modifier cela, il faut placer le paramètre **initial_header_level** à la valeur souhaitée :

```python
parts = publish_parts(file.plaintext, writer_name='html',
                      settings_overrides={'initial_header_level':2,})
return parts['html_body']
```

De cette façon les premiers headers seront directement générés en h2 (il faut mettre la valeur 3 pour h3, 4 pour h4, ...).

