---
title: "Sérialiser une instance de classe en JSON sous Python"
date: 2015-08-19 00:30:35
categories: [Howtos]
tags: [python, json]
author: ncrocfer
---

Je me suis récemment confronté pour un projet perso à devoir sérialiser une instance de classe en JSON. J'aurais très bien pu utiliser un autre type de sérialisation, comme Pickle, mais je suis habitué au JSON et je préfère rester sur ce format pour l'ensemble de mes projets.

Pour la serialisation nous devons passer par la méthode **JSON.dumps()**. A priori rien de bien compliqué pour des types basiques (list, dict, str, bool...), mais les instances de classe ne sont pas serialisables.

[--MORE--]

Prenons une classe toute simple :

.. sourcecode:: python

    # -*- coding: utf-8 -*- 

    class MyClass(object):
        def __init__(self):
            self.var1 = "foo"
            self.var2 = "bar"

Si vous essayez de sérialiser une instance de celle-ci, une exception sera levée :

.. sourcecode:: python

    >>> from myclass import MyClass
    >>> import json
    >>> obj = MyClass()
    >>> json.dumps(obj)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/lib/python3.4/json/__init__.py", line 230, in dumps
        return _default_encoder.encode(obj)
      File "/usr/lib/python3.4/json/encoder.py", line 192, in encode
        chunks = self.iterencode(o, _one_shot=True)
      File "/usr/lib/python3.4/json/encoder.py", line 250, in iterencode
        return _iterencode(o, 0)
      File "/usr/lib/python3.4/json/encoder.py", line 173, in default
        raise TypeError(repr(o) + " is not JSON serializable")
    TypeError: <myclass.MyClass object at 0x7f37074c8e10> is not JSON serializable

Pour pallier à ce problème, la `documentation <https://docs.python.org/3/library/json.html>`_ suggère d'utiliser une sous-classe de **json.JSONEncoder** et de retourner le résultat attendu dans la méthode **default()**. Ce qui donne dans notre cas :

.. sourcecode:: python

    class MyEncoder(json.JSONEncoder):
        def default(self, obj):
            if isinstance(obj, MyClass):
                return vars(obj)
            else:
                return json.JSONEncoder.default(self, obj)

On spécifie ensuite cet *encoder* dans l'argument nommé **cls** :

.. sourcecode:: python

    >>> json.dumps(obj, cls=MyEncoder)
    '{"var1": "foo", "var2": "bar"}'

Ca fonctionne, mais il existe une façon bien plus simple d'arriver au même résultat sans devoir créer d'*encoder* spécifique. Il suffit simplement d'utiliser l'attribut **__dict__** de l'instance. Celui-ci retourne l'ensemble des attributs internes d'un objet sous forme de dictionnaire, et c'est exactement ce que nous souhaitons :

.. sourcecode:: python

    >>> json.dumps(obj.__dict__)
    '{"var1": "foo", "var2": "bar"}'

C'est beau Python...

