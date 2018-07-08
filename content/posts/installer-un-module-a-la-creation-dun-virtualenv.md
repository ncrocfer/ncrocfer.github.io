---
title: "Installer un module à la création d'un virtualenv"
date: 2015-12-23T23:22:23+02:00
aliases: [/2015/12/23/installer-un-module-a-la-creation-dun-virtualenv/]
---

L'utilisation des environnements virtuels (avec **Virtualenv** et **Virtualenvwrapper**) est inévitable si vous développez régulièrement en Python. D'une part leur utilisation permet de laisser votre hôte propre, et d'autre part différentes versions d'un même module peuvent être testées de manière très simple.

<!--more-->

En revanche aucun module n'est disponible par défaut dans un nouvel environnement récemment créé.

Pour cela 2 solutions existent : la première est d'importer les modules disponibles dans l'environnement global (votre host). En gros un **pip freeze** dans votre host et dans votre virtualenv renverra le même résultat :

```bash
$ mkvirtualenv foo --system-site-packages
```

L'idée est d'installer et de mettre à jour sur votre host les modules que vous utilisez le plus souvent, et d'installer dans le virtualenv ceux qui sont plus spécifiques à chaque projet.

La seconde solution, si vous ne souhaitez vraiment rien installer sur le host, et d'installer automatiquement les modules à la création du virtualenv. C'est dans le fichier **~/.virtualenvs/postmkvirtualenv** que les commandes se font :

```bash
#!/bin/bash
# This hook is sourced after a new virtualenv is activated.

pip install requests
pip install httpie
```

La création de l'environnement est un peu plus longue mais les modules sont directement disponibles dans leur dernière version :

```bash
$ mkvirtualenv foo
New python executable in foo/bin/python
Installing setuptools, pip...done.
Downloading/unpacking requests
  Downloading requests-2.9.1-py2.py3-none-any.whl (501kB): 501kB downloaded
Installing collected packages: requests
Successfully installed requests
Cleaning up...
Downloading/unpacking httpie
  Downloading httpie-0.9.2-py2.py3-none-any.whl (66kB): 66kB downloaded
Requirement already satisfied (use --upgrade to upgrade): requests>=2.3.0 in /home/ncrocfer/.virtualenvs/foo/lib/python2.7/site-packages (from httpie)
Downloading/unpacking Pygments>=1.5 (from httpie)
  Downloading Pygments-2.0.2-py2-none-any.whl (672kB): 672kB downloaded
Installing collected packages: httpie, Pygments
Successfully installed httpie Pygments
Cleaning up...
(foo) $ pip freeze
Pygments==2.0.2
argparse==1.2.1
httpie==0.9.2
requests==2.9.1
wsgiref==0.1.2
(foo) $
```
