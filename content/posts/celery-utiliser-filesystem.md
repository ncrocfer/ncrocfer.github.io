---
title: "Celery : utiliser le filesystem comme broker et result backend"
date: 2020-02-09T13:04:09+01:00
---

Lorsqu'on utilise Celery en production il est d'usage d'utiliser un **broker** robuste comme RabbitMQ. Dans le cas où nos tâches utilisent un canvas tel que `chord` il faudra également penser à configurer la connection vers un **result backend** afin de stocker le résultat des tâches. On pourra par exemple utiliser Redis.

Mais avant la mise en production il y a l'étape du développement des tâches. Et même si l'installation d'un Redis ou d'un RabbitMQ est assez simple, j'ai pu remarqué par le passé que ces dépendances fortes pouvaient parfois être un frein à l'adoption de Celery.

Hors il faut savoir que Celery est basé sur la librairie [Kombu](https://github.com/celery/kombu), utilisée pour transmettre des messages d'un producer à un consumer. Et la bonne nouvelle est que Kombu [supporte le filesystem](https://kombu.readthedocs.io/en/latest/_modules/kombu/transport/filesystem.html) comme mode de transport !

Ce qui signifie qu'en configurant Celery on peut facilement s'abstraire de ces dépendances :

```python
from celery import Celery

app = Celery('tasks')
app.conf.update({
    # Configuration du broker
    "broker_url": "filesystem://",
    "broker_transport_options": {
        "data_folder_in": "/tmp/out",
        "data_folder_out": "/tmp/out",
    },

    # Configuration du result backend
    "result_backend": "file:///tmp/result",
})
```

Comme vous le voyez c'est très simple : 

- on indique via le `broker_url` que nous souhaitons utiliser le système de fichiers,
- puis on transmet à Kombu le dossier dans lequel les messages seront stockés.

Evidemment les clés `data_folder_in` et `data_folder_out` doivent être les mêmes, sinon vos producers écriront leurs messages dans un dossier et vos consumers tenteront de les lire ailleurs.

Et comme Celery [supporte par défaut](https://docs.celeryproject.org/en/latest/internals/reference/celery.backends.filesystem.html) le filesystem comme backend, sa configuration peut se faire via la clé `result_backend`.

Il est important que les dossiers existent avant de lancer Celery, voici donc un petit snippet très simple qui les créera pour vous sous un dossier `.celery` du répertoire courant :

```python
# tasks.py
import os
from celery import Celery

OUT_DIR = os.path.join(os.getcwd(), ".celery/out")
RESULT_DIR = os.path.join(os.getcwd(), ".celery/result")

# Create folders if they don't exist
for dir in [OUT_DIR, RESULT_DIR]:
    if not os.path.exists(dir):
        os.makedirs(dir)


app = Celery('tasks')
app.conf.update({
    "broker_url": "filesystem://",
    "broker_transport_options": {
        "data_folder_in": OUT_DIR,
        "data_folder_out": OUT_DIR,
    },
    "result_backend": f"file://{RESULT_DIR}",
})

@app.task
def tsum(numbers):
    return sum(numbers)
```

On peut tester en lançant un simple chord :

```
In [1]: from tasks import tsum

In [2]: from celery import chord

In [3]: chord(
   ...:     [tsum.si([1, 2]), tsum.si([3, 4])],
   ...:     tsum.s()
   ...: ).delay()
Out[3]: <AsyncResult: 11422611-c389-4020-9358-ca485b14d65d>
```

Puis en visualisant les résultats du worker :

```
$ celery@LAPTOP v4.4.0 (cliffs)

Darwin-17.7.0-x86_64-i386-64bit 2020-02-09 14:11:36

[config]
.> app:         tasks:0x10605a8d0
.> transport:   filesystem://localhost//
.> results:     file:///Users/ncrocfer/Dev/.celery/result
.> concurrency: 4 (prefork)
.> task events: OFF (enable -E to monitor tasks in this worker)

[queues]
.> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.tsum

[2020-02-09 14:11:36,645: INFO/MainProcess] Connected to filesystem://localhost//
[2020-02-09 14:11:36,677: INFO/MainProcess] celery@LAPTOP ready.
[2020-02-09 14:11:36,682: INFO/MainProcess] Received task: celery.chord_unlock[71b7898d-b3b4-4689-9f4b-ebbe2e96cb1d]  ETA:[2020-02-09 13:10:35.235858+00:00]
[2020-02-09 14:11:36,687: INFO/MainProcess] Received task: tasks.tsum[39a3728b-ee9a-44f8-b661-97e20e74c72e]
[2020-02-09 14:11:36,694: INFO/MainProcess] Received task: tasks.tsum[e79e3917-1c80-4366-adcc-23b601f38836]
[2020-02-09 14:11:36,700: INFO/ForkPoolWorker-1] Task tasks.tsum[39a3728b-ee9a-44f8-b661-97e20e74c72e] succeeded in 0.005183988017961383s: 3
[2020-02-09 14:11:36,705: INFO/ForkPoolWorker-2] Task tasks.tsum[e79e3917-1c80-4366-adcc-23b601f38836] succeeded in 0.005367834935896099s: 7
[2020-02-09 14:11:37,737: INFO/ForkPoolWorker-3] Task celery.chord_unlock[71b7898d-b3b4-4689-9f4b-ebbe2e96cb1d] succeeded in 0.05950633704196662s: None
[2020-02-09 14:11:38,711: INFO/MainProcess] Received task: tasks.tsum[11422611-c389-4020-9358-ca485b14d65d]
[2020-02-09 14:11:38,719: INFO/ForkPoolWorker-4] Task tasks.tsum[11422611-c389-4020-9358-ca485b14d65d] succeeded in 0.0034185299882665277s: 10
```

Evidemment cette configuration n'est à utiliser qu'en mode *développement*, je vous conseille fortement de passer sous un autre broker lorsque vous passerez vos tâches en prod :)