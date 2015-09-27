---
title: "Formater du JSON dans le terminal"
date: 2015-09-27 18:16:18
categories: [Howtos]
tags: [python, json]
author: ncrocfer
---

Il m'arrive de devoir lire un fichier JSON dans le terminal. Dans bien des cas malheureusement ce fichier n'est pas formaté et la lecture est quasiment impossible.

[--MORE--]

Par exemple avec ce fichier **customers.json** :

.. sourcecode:: bash

    $ more customers.json
    [ { "_id": "56081479e15e4ab842a896e8", "index": 0, "guid": "5a6b6434-e2c0-4c03-8103-a7009132a3c6", "isActive": false, "balance": "$3,324.04", "picture": "http://placehold.it/32x32", "age": 30, "eyeColor": "brown", "name": "Joan Maldonado", "gender": "female", "company": "PLASMOS", "email": "joanmaldonado@plasmos.com", "phone": "+1 (882) 446-3244", "address": "904 Will Place, Vincent, New Hampshire, 791", "about": "Enim velit reprehenderit aliquip nulla sint. Commodo et magna pariatur Lorem. Ut esse ut aliquip nulla et labore aute veniam qui ex et. Do proident elit occaecat id enim id ad fugiat nostrud. Amet non minim cupidatat sint voluptate ullamco. Consequat commodo sunt elit mollit consequat labore nisi excepteur sint ex consequat.\r\n", "registered": "2014-07-22T10:21:02 -02:00", "latitude": 20.049304, "longitude": -116.525111, "tags": [ "in", "non", "esse", "Lorem", "dolor", "id", "officia" ], "friends": [ { "id": 0, "name": "Gardner Reese" }, { "id": 1, "name": "Iva Flores" }, { "id": 2, "name": "Walters Young" } ], "greeting": "Hello, Joan Maldonado! You have 3 unread messages.", "favoriteFruit": "apple" }, { "_id": "56081479db98561a62a7679d", "index": 1, "guid": "6109f36b-c17b-4a7d-9c3d-82b5afed5b08", "isActive": false, "balance": "$2,768.70", "picture": "http://placehold.it/32x32", "age": 26, "eyeColor": "green", "name": "Tisha Patel", "gender": "female", "company": "KIOSK", "email": "tishapatel@kiosk.com", "phone": "+1 (860) 531-3279", "address": "500 Caton Place, Rockingham, Kansas, 7867", "about": "Magna do ullamco commodo Lorem ullamco eu sit enim dolor. Enim est eiusmod irure commodo ad duis veniam officia dolor incididunt veniam in proident. Magna eiusmod cillum sit nisi ullamco nostrud velit laboris sint enim ad voluptate nulla quis. Voluptate consectetur excepteur eiusmod sunt. Labore officia ea laborum quis tempor magna laboris nostrud aute adipisicing non.\r\n", "registered": "2015-08-23T12:53:27 -02:00", "latitude": -40.008583, "longitude": 94.809866, "tags": [ "dolore", "et", "laborum", "occaecat", "aute", "laboris", "officia" ], "friends": [ { "id": 0, "name": "Phyllis Mccullough" }, { "id": 1, "name": "Mercedes Bullock" }, { "id": 2, "name": "Virginia Witt" } ], "greeting": "Hello, Tisha Patel! You have 8 unread messages.", "favoriteFruit": "apple" }, { "_id": "5608147988cc9bbeebd14516", "index": 2, "guid": "f0bb0639-5d2a-47e1-883d-21fb53fb3d4c", "isActive": true, "balance": "$2,235.78", "picture": "http://placehold.it/32x32", "age": 23, "eyeColor": "brown", "name": "Elvia Gaines", "gender": "female", "company": "GYNK", "email": "elviagaines@gynk.com", "phone": "+1 (991) 513-2225", "address": "612 Canton Court, Grapeview, Michigan, 1411", "about": "Ipsum incididunt ad occaecat occaecat ipsum adipisicing sunt veniam aliquip commodo. Ad mollit elit anim nulla dolor aute elit qui tempor cupidatat anim nulla incididunt. Lorem ex non et aute laborum. Et esse qui consequat quis labore. Irure commodo incididunt Lorem magna ullamco. Reprehenderit consequat tempor cupidatat eiusmod magna in laboris adipisicing.\r\n", "registered": "2015-08-13T05:05:02 -02:00", "latitude": -72.222724, "longitude": 56.951741, "tags": [ "ullamco", "cillum", "excepteur", "proident", "do", "do", "voluptate" ], "friends": [ { "id": 0, "name": "Jarvis Lambert" }, { "id": 1, "name": "Carla Ferguson" }, { "id": 2, "name": "Cline Combs" } ], "greeting": "Hello, Elvia Gaines! You have 7 unread messages.", "favoriteFruit": "strawberry" }, { "_id": "56081479a320a31282066ffb", "index": 3, "guid": "11d34616-a325-4326-a4dd-260f26950ca8", "isActive": true, "balance": "$2,083.11", "picture": "http://placehold.it/32x32", "age": 24, "eyeColor": "blue", "name": "Sallie Barr", "gender": "female", "company": "TRANSLINK", "email": "salliebarr@translink.com", "phone": "+1 (911) 454-2623", "address": "714 Doughty Street, Grahamtown, North Dakota, 4512", "about": "Non incididunt do anim culpa adipisicing nostrud ullamco ea tempor. Enim duis quis pariatur excepteur ut voluptate et amet ut sint voluptate. Consectetur id irure mollit adipisicing veniam eiusmod veniam occaecat. Dolor consequat nulla dolor id velit eu irure. Cillum velit ad sunt fugiat officia cupidatat fugiat et consectetur amet tempor. Duis fugiat ipsum tempor ipsum fugiat nisi pariatur dolor cupidatat adipisicing ea nostrud velit. Eu nulla laborum fugiat ipsum occaecat non est quis do voluptate minim et officia.\r\n", "registered": "2014-08-10T03:51:55 -02:00", "latitude": -74.14838, "longitude": 24.023487, "tags": [ "elit", "proident", "aliquip", "culpa", "dolor", "cupidatat", "do" ], "friends": [ { "id": 0, "name": "Aida Scott" }, { "id": 1, "name": "Cash Gillespie" }, { "id": 2, "name": "Bryan Hoover" } ], "greeting": "Hello, Sallie Barr! You have 4 unread messages.", "favoriteFruit": "apple" }, { "_id": "56081479a03f0f3f3f359e7b", "index": 4, "guid": "d9bfb305-7c94-43ec-974a-12addf238468", "isActive": false, "balance": "$3,170.81", "picture": "http://placehold.it/32x32", "age": 39, "eyeColor": "brown", "name": "Frances Martinez", "gender": "female", "company": "CODACT", "email": "francesmartinez@codact.com", "phone": "+1 (887) 504-2517", "address": "609 Balfour Place, Emory, Colorado, 6617", "about": "Incididunt consequat deserunt reprehenderit eu laborum aliquip duis non fugiat mollit fugiat. Velit do eiusmod fugiat consequat aliquip fugiat anim mollit aliquip cupidatat cupidatat Lorem reprehenderit est. Dolore voluptate culpa labore minim. Commodo sunt commodo sint in exercitation aliquip aute aute sunt deserunt ea occaecat est irure. Minim aliqua proident est excepteur. Proident minim deserunt dolore adipisicing aliquip ipsum enim est consectetur amet fugiat excepteur ex. Tempor non officia exercitation ipsum ipsum est reprehenderit reprehenderit pariatur nulla fugiat reprehenderit consequat mollit.\r\n", "registered": "2014-07-18T08:39:15 -02:00", "latitude": -11.888415, "longitude": 150.058245, "tags": [ "enim", "culpa", "deserunt", "est", "elit", "veniam", "voluptate" ], "friends": [ { "id": 0, "name": "Mooney Kelly" }, { "id": 1, "name": "Maddox Mills" }, { "id": 2, "name": "Olsen Perkins" } ], "greeting": "Hello, Frances Martinez! You have 1 unread messages.", "favoriteFruit": "strawberry" }, { "_id": "56081479d6f7624b753fbd24", "index": 5, "guid": "f6f40203-43b5-46b4-90ef-4e03ac0410d9", "isActive": true, "balance": "$2,787.96", "picture": "http://placehold.it/32x32", "age": 34, "eyeColor": "green", "name": "Houston Kerr", "gender": "male", "company": "INCUBUS", "email": "houstonkerr@incubus.com", "phone": "+1 (912) 474-3368", "address": "802 Preston Court, Cataract, Kentucky, 5816", "about": "Eu amet nulla veniam nostrud magna id. Veniam elit qui exercitation esse esse excepteur reprehenderit ullamco et. Dolore officia Lorem pariatur tempor dolore Lorem quis enim ad aliqua. Sit sunt culpa aute consequat fugiat. Magna minim duis ea cillum quis duis proident tempor deserunt velit cupidatat nulla. Culpa eiusmod consectetur eiusmod labore.\r\n", "registered": "2014-10-12T09:08:48 -02:00", "latitude": 33.730743, "longitude": -17.213511, "tags": [ "sunt", "dolor", "sit", "sit", "eu", "ad", "ex" ], "friends": [ { "id": 0, "name": "Marguerite Skinner" }, { "id": 1, "name": "Webster Sykes" }, { "id": 2, "name": "Wendy Campos" } ], "greeting": "Hello, Houston Kerr! You have 5 unread messages.", "favoriteFruit": "strawberry" } ]

Pour pallier à ce problème une solution est d'utiliser le module JSON de Python :

.. sourcecode:: bash

    $ python -m json.tool customers.json
    [
        {
            "_id": "56081479e15e4ab842a896e8",
            "about": "Enim velit reprehenderit aliquip nulla sint. Commodo et magna pariatur Lorem. Ut esse ut aliquip nulla et labore aute veniam qui ex et. Do proident el
    it occaecat id enim id ad fugiat nostrud. Amet non minim cupidatat sint voluptate ullamco. Consequat commodo sunt elit mollit consequat labore nisi excepteur sint ex co
    nsequat.\r\n",
            "address": "904 Will Place, Vincent, New Hampshire, 791",
            "age": 30,
            "balance": "$3,324.04",
            "company": "PLASMOS",
            "email": "joanmaldonado@plasmos.com",
            "eyeColor": "brown",
            "favoriteFruit": "apple",
            ...
        }
    ]


En revanche la coloration syntaxique n'est pas prise en compte (bon ici le résultat est coloré, mais ce n'est pas le cas dans le terminal). Pour cela on peut utiliser **jq**, qui est un parser complet de JSON intégrant une tonne d'options. Nous allons simplement l'utiliser pour formater et colorer notre fichier :

.. sourcecode:: bash

    $ jq "." customers.json
    [
      {
        "favoriteFruit": "apple",
        "greeting": "Hello, Joan Maldonado! You have 3 unread messages.",
        "friends": [
          {
            "name": "Gardner Reese",
            "id": 0
          },
          {
            "name": "Iva Flores",
            "id": 1
          },
          {
            "name": "Walters Young",
            "id": 2
          }
        ],
        ...
      }
    ]

Le "." est l'expression de base de **jq** qui permet de renvoyer le résultat sans le modifier.

Bien sûr je ne montre ici que l'utilisation basique de l'outil. Comme je le disais avant il existe énormément d'options et je vous invite à lire la `doc <https://stedolan.github.io/jq/manual/>`_ pour en savoir plus.
