---
title: "netstat sur Debian 8 Jessie"
date: 2015-11-24 23:18:55
categories: [Howtos]
tags: [netstat, docker]
author: ncrocfer
---

Lorsqu'on utilise docker pour développer, on ne sait pas toujours sur quel OS on va tomber. Tout dépend du choix du créateur de l'image : on tombera parfois sur du Debian, parfois sur du Ubuntu, parfois encore sur du CentOS.

[--MORE--]

En revanche je me suis rendu compte que la plupart (la totalité ?) des images officielles proposées par Docker étaient quant à elles basées sur Debian. Même si parfois elles diffèrent de version (exemple avec l'image `Redis <https://github.com/docker-library/redis/blob/8929846148513a1e35e4212003965758112f8b55/3.0/Dockerfile>`_ sous Wheezy et l'image `RabbitMQ <https://github.com/docker-library/rabbitmq/blob/60e5665854131f7fcb9ab0285d7d52ac932efb43/Dockerfile>`_ sous Jessie), on reste quand même sur du Debian. Et l'image officielle Python ne fait pas exception à cette règle.

Du coup pas de netstat de base pour tester si votre gunicorn écoute sur le bon port ou s'il est accessible sur localhost ou sur 0.0.0.0 (open bar).

Alors si comme moi vous avez bêtement tenté de rechercher le paquet netstat et que vous êtes tombé sur ça :

.. sourcecode:: bash

    ncrocfer@home:~$ docker run -it python:3.4 /bin/bash
    root@caa036eb3980:/# apt-get update
    ...
    ...
    root@caa036eb3980:/# apt-cache search netstat
    libparse-netstat-perl - module to parse the output of the "netstat" command
    netstat-nat - tool that display NAT connections
    root@caa036eb3980:/#

Eh bien sachez que l'outil se situe dans le paquet **net-tools** :

.. sourcecode:: bash

    root@caa036eb3980:/# apt-get install net-tools

Vous pouvez maintenant lancer vos services et vérifiez leurs connexions :

.. sourcecode:: bash

    root@caa036eb3980:/# pip install flask
    root@caa036eb3980:/# cat > app.py <<- EOM
    > from flask import Flask
    > app = Flask(__name__)
    > 
    > @app.route("/")
    > def hello():
    >     return "Hello World!"
    > 
    > if __name__ == "__main__":
    >     app.run()
    > EOM
    root@caa036eb3980:/# python app.py &
    [1] 104
    root@caa036eb3980:/# netstat -plnt
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      -               
    root@caa036eb3980:/#


