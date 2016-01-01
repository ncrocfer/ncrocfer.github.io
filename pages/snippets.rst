---
title: Snippets
menu: True
---

Cette page recence les snippets que je trouve intéressants et dont je peux être amené à me servir régulièrement. Ceux qui sont mis dans un alias peuvent être placés dans le **~/.bashrc**.

Les snippets seront parfois liés à un article, parfois non.

------------

**Python : supprimer les fichiers .pyc, .pyo et le dossier pycache**

.. sourcecode:: bash

    $ alias rmpyc='find . | grep -E "(pycache|.pyc|.pyo$)" | xargs rm -rf'

**Docker : démarrer un shell dans un container actif**

.. sourcecode:: bash

    $ cat ~/.bashrc
    ...
    dockerbash() {
        docker exec -it $1 /bin/bash
    }
    alias docker-bash=dockerbash

    $ docker-bash bc543781f4f5

**Docker : supprimer tous les containers**

.. sourcecode:: bash

    $ alias docker-rm-all='docker rm --force `docker ps -qa`'


