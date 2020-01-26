---
title: "Celery : l'importance du pool d'exécution"
date: 2020-01-12T13:25:39+01:00
---

En tant que dev nous sommes régulièrement amenés à devoir exécuter des tâches en arrière-plan. La plupart du temps il s'agira de répondre à une action de l'utilisateur (call d'API, click sur un bouton...), parfois nous aurons besoin de lancer une tâche de manière périodique.

Dans les 2 cas l'objectif est de ne pas bloquer le processus principal. En effet je vous laisse imaginer les pertes de performance si, au moment d'un POST sur votre API, votre application se chargeait d'exécuter elle-même une tâche lourde et très longue. L'application rendrait la main à l'utilisateur plusieurs secondes, voire plusieurs minutes, après le call.

{{% img-center "/images/celery.png" %}}

Pour répondre à ce besoin les dev Python ont pris l'habitude d'utiliser la librairie **Celery**. Cet article n'est pas une introduction à cet outil, pour cela je vous renvoie vers le [quickstart](https://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html) officiel, vous comprendrez rapidement à quoi il sert et surtout comment l'utiliser.

Nous allons parler ici d'une feature qui est parfois délaissée lorsqu'on utilise Celery : les **pools d'exécution**. Derrière cette notion se cache toute la magie de Celery : comment et par quoi mes tâches vont-elles s'exécuter. Et vous allez voir que configurer correctement cette option peut amener des gains de performances incroyables !

La théorie
==========

Lorsque nous lançons un worker Celery, nous lançons en fait un processus chargé de coordonner le lancement des tâches empilées dans le broker. En réalité leur exécution à proprement parler ne sera pas effectuée par ce processus mais par d'autres processus enfants ou par des threads qu'il aura lui-même spawné.

Et c'est justement la façon dont est configuré le pool d'éxécution qui détermine comment sont lancés ces processus ou threads capables d'exécuter les tâches.

Celery fournit par défaut 4 pools d'exécution : **prefork**, **solo**, **gevent** et **eventlet**. La configuration s'effectue via l'option `-P`.

Mode prefork
------------

Par défault le pool sélectionné par Celery est le prefork. Nous pouvons facilement le voir lorsque nous lançons un worker :

```
$ celery -A tasks worker
celery@LAPTOP v4.4.0 (cliffs)

Darwin-17.7.0-x86_64-i386-64bit 2020-01-26 14:31:06

[config]
.> app:         tasks:0x10566ca20
.> transport:   redis://localhost:6379/0
.> results:     disabled://
.> concurrency: 4 (prefork)
.> task events: OFF (enable -E to monitor tasks in this worker)

[queues]
.> celery           exchange=celery(direct) key=celery
```

Ici la ligne `concurrency: 4 (prefork)` nous indique que nous utilisons le pool `prefork` avec une concurrency de **4**. Ce mode d'exécution se base sur le package [multiprocessing](https://docs.python.org/fr/3/library/multiprocessing.html) : Celery va donc spawner autant de processus que le nombre de coeurs de votre machine.

Nous pouvons vérifier que les processus ont bien été spawnés par Celery (le process principal et les 4 forks):

```
$ ps -A | grep -i celery
91822 ttys000    0:01.59 /Users/ncrocfer/Dev/celery_pool/venv/bin/python3 /Users/ncrocfer/Dev/celery_pool/venv/bin/celery -A tasks worker
91828 ttys000    0:00.02 /Users/ncrocfer/Dev/celery_pool/venv/bin/python3 /Users/ncrocfer/Dev/celery_pool/venv/bin/celery -A tasks worker
91829 ttys000    0:00.02 /Users/ncrocfer/Dev/celery_pool/venv/bin/python3 /Users/ncrocfer/Dev/celery_pool/venv/bin/celery -A tasks worker
91830 ttys000    0:00.02 /Users/ncrocfer/Dev/celery_pool/venv/bin/python3 /Users/ncrocfer/Dev/celery_pool/venv/bin/celery -A tasks worker
91831 ttys000    0:00.02 /Users/ncrocfer/Dev/celery_pool/venv/bin/python3 /Users/ncrocfer/Dev/celery_pool/venv/bin/celery -A tasks worker
```



Ce mode est particulièrement adapté aux applications nécessitant du calcul par le CPU (ces applications sont dîtes **CPU bound**). Plus vous aurez de coeurs puissants et plus votre application pourra traiter de tâches en parallèle.

A noter que la concurrency est paramétrable via l'option `-c` : 

```
$ celery -A tasks worker -c 2
...
.> concurrency: 2 (prefork)
...
```

Mode solo
---------

Ce mode est un peu particulier car le processus principal se chargera lui-même d'exécuter les tâches qu'il consommera :

```
$ celery -A tasks worker -P solo
...
.> concurrency: 4 (solo)
...
```

La concurrency reste à 4 mais ne vous y fiez pas, le mode **solo** ne s'en encombre pas et n'exécutera qu'une seule tâche à la fois.

Là encore nous pouvons le vérifier via les processus en cours d'exécution sur notre machine :

```
$ ps -A | grep -i celery
92356 ttys000    0:01.13 /Users/ncrocfer/Dev/celery_pool/venv/bin/python3 /Users/ncrocfer/Dev/celery_pool/venv/bin/celery -A tasks worker -P solo
```

Hormis pour lancer des tests, ce mode pourra être utile dans le cas d'un usage en environnement *containers*. En effet les comptes sont simples dans ce cas : *N* containers égal *N* tasks en parallèle.

Mode gevent & eventlet
----------------------

Là on arrive dans la partie intéressante :) A l'inverse du mode `prefork` utile pour gérer des tâches **CPU bound**, ces deux modes vont nous permettre de gérer facilement les tâches dîtes **IO bound**.

De telles tâches n'effectuent pas de calculs lourds, et ne nécessitent donc pas un grand nombre de CPU puissants. En revanche elles peuvent être amenées à effectuer beaucoup de requêtes vers d'autres systèmes : une API externe, une base de données, un site web, etc. Ici notre CPU n'a pas grand chose à faire puisque nos tâches vont principalement attendre le retour de leurs requêtes.

Je vous invite à vous renseigner sur la librairie [Asyncio](https://docs.python.org/3/library/asyncio.html) qui expliquera tout ça mieux que moi.

Dans l'exemple ci-dessous notre worker pourra gérer 100 tâches en parallèle via le mode `gevent` :

```
$ celery -A tasks worker -P gevent -c 100
...
.> concurrency: 100 (gevent)
...
```

La pratique
===========

Trève de théorie, illustrons maintenant l'utilisation des pool d'exécution via un exemple.

Nous souhaitons lancer plusieurs tâches en parallèle effectuant une requête vers une API externe. Cette API renvoie un nombre au hasard, et une fonction de callback est appelée afin d'effectuer la somme de ces nombres. Certes cet exemple est basique mais il va nous permettre de mettre en évidence la puissance des pools d'exécution dans Celery.

L'API tourne en locale, j'ai utilisé [Sanic](https://sanic.readthedocs.io/en/latest/) avec un timeout de 2 secondes *simulant* la latence d'une API externe :

```python
$ cat app.py
import random
import asyncio

from sanic import Sanic
from sanic.response import json


app = Sanic()


@app.get("/")
async def get_time(request):
    await asyncio.sleep(2)
    return json({"number": random.randint(1, 10)})


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

Nos tâches sont elles-aussi très simples :

```python
$ cat tasks.py
import requests
from celery import Celery, chord


app = Celery('tasks', broker='redis://localhost:6379/0', backend="redis://localhost:6379/1")


@app.task
def get_number():
    resp = requests.get("http://localhost:8000")
    return resp.json()["number"]

@app.task
def sum_numbers(numbers):
    return sum(numbers)


sign = chord(
    [get_number.si() for _ in range(5)],
    sum_numbers.s()
)
```

J'utilise un chord afin de lancer plusieurs requêtes en parallèle. La fonction `sum_numbers` est ensuite lancée afin de calculer la somme totale des nombres.

Le lancement peut s'effectuer en une ligne :

```
$ python -c "from tasks import sign; sign.delay()"
```

Testons avec le pool d'exécution par défaut (prefork) :

```
$ celery -A tasks worker --loglevel=INFO
...
[2020-01-26 19:30:48,852: INFO/MainProcess] Received task: tasks.get_number[5f2f8167-c667-4e83-b591-453e57b3a958]
[2020-01-26 19:30:48,855: INFO/MainProcess] Received task: tasks.get_number[a333586b-c396-4b91-917f-3fbdfa6d30e8]
[2020-01-26 19:30:48,860: INFO/MainProcess] Received task: tasks.get_number[88fa7cc0-2622-4053-8824-2f2d47c25a04]
[2020-01-26 19:30:48,876: INFO/MainProcess] Received task: tasks.get_number[fa35004e-51b8-4f48-ad84-e02640ed9621]
[2020-01-26 19:30:48,880: INFO/MainProcess] Received task: tasks.get_number[5ecb73c8-9ba5-4ebb-aaae-78e14908405a]
[2020-01-26 19:30:50,915: INFO/ForkPoolWorker-2] Task tasks.get_number[5f2f8167-c667-4e83-b591-453e57b3a958] succeeded in 2.057117607037071s: 1
[2020-01-26 19:30:50,915: INFO/ForkPoolWorker-3] Task tasks.get_number[a333586b-c396-4b91-917f-3fbdfa6d30e8] succeeded in 2.057576177001465s: 2
[2020-01-26 19:30:50,920: INFO/ForkPoolWorker-1] Task tasks.get_number[88fa7cc0-2622-4053-8824-2f2d47c25a04] succeeded in 2.0558868430089206s: 2
[2020-01-26 19:30:50,933: INFO/ForkPoolWorker-4] Task tasks.get_number[fa35004e-51b8-4f48-ad84-e02640ed9621] succeeded in 2.055466585967224s: 1
[2020-01-26 19:30:52,972: INFO/ForkPoolWorker-2] Task tasks.get_number[5ecb73c8-9ba5-4ebb-aaae-78e14908405a] succeeded in 2.0525691980146803s: 10
[2020-01-26 19:30:52,973: INFO/MainProcess] Received task: tasks.sum_numbers[605e4af4-229c-42a3-9be0-ce5daff79a0f]
[2020-01-26 19:30:52,975: INFO/ForkPoolWorker-2] Task tasks.sum_numbers[605e4af4-229c-42a3-9be0-ce5daff79a0f] succeeded in 0.0006177640170790255s: 16
```

Les tâches sont toutes prises en compte en une seule fois par notre worker (rien à voir avec la notion de pool d'exécution, il s'agit ici du **prefetch**, en gros le nombre de tâches qu'un worker peut se *réserver*).

En revanche seules 4 tâches sont effectuées en même temps, il faudra attendre 2 secondes supplémentaires pour que la 5ème le soit également. Le workflow complet aura donc pris **4 secondes**.

Testons maintenant avec le pool d'exécution **gevent** :

```
$ celery -A tasks worker --loglevel=INFO -P gevent -c 100
...
[2020-01-26 19:35:12,151: INFO/MainProcess] Received task: tasks.get_number[04192b64-b71e-4e1c-8eb8-f86487c6b013]
[2020-01-26 19:35:12,163: INFO/MainProcess] Received task: tasks.get_number[12d227b5-f0b1-4fd8-8b44-25e8d8fbc028]
[2020-01-26 19:35:12,173: INFO/MainProcess] Received task: tasks.get_number[db7838f5-3543-414c-9816-7df62b13b65d]
[2020-01-26 19:35:12,182: INFO/MainProcess] Received task: tasks.get_number[52f163d4-7d75-493a-a3ac-65122691b078]
[2020-01-26 19:35:12,193: INFO/MainProcess] Received task: tasks.get_number[14537419-8710-4b32-bb13-6c884ff22dca]
[2020-01-26 19:35:14,202: INFO/MainProcess] Task tasks.get_number[04192b64-b71e-4e1c-8eb8-f86487c6b013] succeeded in 2.048632029967848s: 6
[2020-01-26 19:35:14,207: INFO/MainProcess] Task tasks.get_number[12d227b5-f0b1-4fd8-8b44-25e8d8fbc028] succeeded in 2.042411718983203s: 2
[2020-01-26 19:35:14,208: INFO/MainProcess] Task tasks.get_number[14537419-8710-4b32-bb13-6c884ff22dca] succeeded in 2.0140742670046166s: 10
[2020-01-26 19:35:14,209: INFO/MainProcess] Task tasks.get_number[db7838f5-3543-414c-9816-7df62b13b65d] succeeded in 2.03493124502711s: 6
[2020-01-26 19:35:14,225: INFO/MainProcess] Task tasks.get_number[52f163d4-7d75-493a-a3ac-65122691b078] succeeded in 2.0403319080360234s: 9
[2020-01-26 19:35:14,225: INFO/MainProcess] Received task: tasks.sum_numbers[043e87a0-4963-468a-9f17-7ffe93127fcd]
[2020-01-26 19:35:14,227: INFO/MainProcess] Task tasks.sum_numbers[043e87a0-4963-468a-9f17-7ffe93127fcd] succeeded in 0.0006761000258848071s: 33
```

Ici les 5 tâches sont bien effectuées en même temps, le workflow aura donc mis **2 secondes**. Et encore je n'ai lancé que 5 tâches en parallèle, n'hésitez pas à augmenter ce nombre afin de constater par vous-même les résultats. On comprend bien la puissance de Gevent face à des tâches dédiées à de l'IO Bound.



Conclusion
==========

Les pool d'exécution ne sont pas beaucoup mis en avant dans la documentation Celery, hors comme vous le voyez il s'agit d'une configuration très simple à mettre en oeuvre afin de bénéficier de meilleurs performances.

N'hésitez surtout pas à utiliser les queues pour effectuer le routing de vos tâches IO bound et CPU bound. Les tâches gourmandes en CPU pourront être redirigées vers une queue **cpu** et les tâches gourmandes en IO bound vers une queue **io**. 

Il suffira alors de lancer 2 workers avec les bons arguments :

```
$ celery -A tasks worker -P gevent -c 100 -Q io
$ celery -A tasks worker -P prefork -c 4 -Q cpu
```

Vous trouverez plus d'info sur les queues [ici](https://docs.celeryproject.org/en/latest/userguide/routing.html).

En espérant que cet article vous ait plus, n'hésitez pas à me contacter sur [Twitter](https://twitter.com/ncrocfer) si vous avez des questions ou des remarques :)