---
title: "Autoreload des modules sous iPython"
date: 2015-11-30T21:01:09+02:00
aliases: [/2015/11/30/autoreload-des-modules-sous-ipython/]
---

Si vous utilisez iPython comme interpréteur Python (et si ce n'est pas le cas vous loupez quelque chose), il vous est certainement déjà arrivé de bosser dessus dans un terminal, dans un autre de modifier l'un des modules importés, puis de relancer la fonction dans iPython. Sauf que ce dernier n'a pas rechargé le code et que le résultat est le même qu'avant.

<!--more-->

Prenez par exemple le code suivant :

```python
def calcul(a, b):
    print(a+b)
```

Bon ok c'est bidon, mais c'est pour montrer le principe. Dans un autre terminal, démarrez iPython puis testez-le :

```python
In [1]: from calcul import calcul
In [2]: calcul(2, 3)
5
```

Si vous modifiez la fonction dans le second terminal, par exemple :

```python
def calcul(a, b):
    print(a*b)
```

La mise à jour ne sera pas répercutée dans iPython, il faudrait le fermer puis le relancer. L'activation de l'**autoreload** permet d'éviter cela :

```ipython
In [3]: calcul(2, 3)
5

In [4]: %load_ext autoreload

In [5]: %autoreload 2

# ré-enregistrez le module calcul.py

In [6]: calcul(2, 3)
6
```

Comme vous le voyez les modifications effectuées après l'activation de l'extension sont automatiquement pris en compte. Si vous souhaitez que l'extension soit activée à chaque session, il faut créer un nouveau profile :

```bash
ncrocfer@home:~/ ipython profile create
[ProfileCreate] Generating default config file: '/home/ncrocfer/.ipython/profile_default/ipython_config.py'
```

Puis ajoutez à la fin les lignes suivantes :

```bash
c.InteractiveShellApp.exec_lines = []
c.InteractiveShellApp.exec_lines.append('%load_ext autoreload')
c.InteractiveShellApp.exec_lines.append('%autoreload 2')
```

L'autoreload des modules importés s'effectuera désormais automatiquement !
