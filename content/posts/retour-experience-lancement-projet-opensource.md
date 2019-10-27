---
title: "Retour d'experience sur le lancement d'un projet opensource"
date: 2019-10-27T23:44:30+01:00
---

Ceux qui me suivent sur [Twitter](https://twitter.com/ncrocfer) savent que j'ai créé il y a quelque temps une plateforme dédiée à l'alerting de vulnérabilités : [Saucs.com](https://www.saucs.com).

Déjà 3 ans (le premier commit date du 03/11/2016) que le projet vit et que de nombreux utilisateurs s'en servent pour effectuer leur veille au quotidien. Malheureusement la vie personnelle et professionnelle fait qu'il est très difficile de maintenir seul ce genre de projet, c'est pourquoi il a été décidé en 2019 d'opensourcer Saucs. 

Cet article explique les difficultés d'un tel projet.

<!--more-->

Pourquoi opensourcer Saucs
==========================

Pour info la plateforme s'appelait à l'origine SecAdvisory, mais nous avons décidé de la renommer lorsque [Laurent](https://twitter.com/LaurentDurnez) a pris part au projet en tant que sysadmin. Il a depuis fait un boulot de fou, et si le site tient si bien la charge c'est vraiment grâce à lui et à l'architecture qu'il a su mettre en place.

A la base Saucs est né d'un besoin personnel : je souhaitais être tenu au courant des vulnérabilités d'un produit mais aucune plateforme ne répondait à mes attentes. L'idée a donc été de parser les CVE mises à disposition par le NVD, d'en extraire les produits associés (grâce aux CPE) puis de proposer un système d'alertes basé dessus.

En soit le principe est extrèmement simple, mais de nombreux utilisateurs ont été séduits par Saucs, peut-être dû au fait que le NVD ne fournit pas cette fonctionnalité sur son propre site. Les chiffres actuels sont plutôt sympas : au moment de l'écriture de cet article nous comptons exactement 3,351 utilisateurs et 1,184,274 rapports envoyés.

Malheureusement nous ne sommes pas assez et nous n'avons pas le temps de tout gérer. Alors que nous avons des tonnes d'idées en tête, leur concrétisation est devenu très compliquée par simple manque de temps. Les fonctionnalités ne sortent plus à un rythme convenable, et c'est pourquoi nous avons donc décidé début 2019 de libérer le code source et de transformer Saucs en solution opensource.

A priori l'idée est excellente : un code maintenu par la communauté, plus de fonctionnalités, d'autres outils basés dessus, la possibilité d'installation on-premise, etc.  

Mais c'est bien plus facile à dire qu'à faire, et je vais vous expliquer dans cet article pourquoi. En outre les utilisateurs réclamant à tout bout de champs où j'en suis comprendront peut-être un peu mieux pourquoi cela met autant de temps :)

Les difficultés rencontrées
===========================

La principale difficulté que j'ai rencontré a été de me *replonger* dans mon propre code source ! Cela faisait plusieurs mois que je n'y avais pas touché (voire plus d'un an), et la reprise a été difficile.

Il a ensuite fallu décider ce que je devais garder et ce que je devais modifier. La liste des choses à changer était finalement assez courte : le thème (indispensable car acheté en ligne et sous licence), l'upgrade des dépendances, un peu de refacto par-ci par-là, et c'était à priori tout. Ce qui pouvait me paraître simple m'a finalement pris plusieurs soirées et week-ends sur mon temps libre (pour rappel ce projet n'est en aucun cas lié à mon job).

Ce travail de refacto a malheureusement fait apparaître un nouveau problème : le legacy accumulé en 3 ans. En effet l'écosystème entourant les CVE avait sacrément évolué, et les flux proposés par le NVD aussi. Les fichiers XML sur lesquels Saucs se basait sont désormais dépréciés par un flux JSON. Encore une fois, ce qui peut paraître simple ("il suffit de changer le parser, voyons") ne l'est pas forcément quand ça n'a pas été prévu (dépendances fortes dans le code).

Anecdote amusante : j'ai entamé la migration du XML vers le JSON il y a quelques mois. A ce moment-là le NVD fournissait la version 1.0 de ce flux, et je me servais d'une clé nommée *affects* pour parser les vendeurs et leurs produits affectés. Cette clé était très pratique puisqu'elle fournissait directement la liste dans un format simple. Malheureusement j'ai eu la mauvaise surprise en revenant de vacances de constater que mon code levait une exception... la clé avait été supprimée dans la nouvelle version 1.1 de ce flux ([changelog](https://nvd.nist.gov/General/News/JSON-1-1-Vulnerability-Feed-Release)):

> *"In CVE_JSON_4.0_min.schema, the affects element has been removed from the required properties."*

Encore du travail non prévu. Je pourrai également citer l'exemple des CVSS : Saucs était à l'origine basé sur la version 2 alors que la version 3 est désormais proposée. Un changement dans le model Python (et donc dans la database directement) a dû être opéré, impliquant des migrations dans le schema actuel.

Le refacto du code m'a également permis de détecter ce que je pourrais appeler aujourd'hui *des erreurs de conception*. Erreurs qui me paraissaient normales et non gênantes il y a 3 ans. C'est là qu'on remarque que notre façon de coder évolue dans le temps :)

Pour finir il est également important de noter que le code ne pouvait pas être opensourcer tel quel car l'installation était pour ainsi dire *chaotique*. En effet l'import initial des CVE, CPE et CWE prenait plusieurs heures, il a donc fallut travailler à l'amélioration de cette partie. Pour info c'est désormais faisable en quelques minutes grâce à la commande `saucs import`.

Les erreurs à éviter
====================

Je dirai que la principale erreur que j'ai commise a été d'annoncer l'opensource de Saucs avant même d'avoir commencé à travailler dessus.

Je ne savais pas la charge de travail que cela allait représenter, et j'ai créé une attente parmi certains utilisateurs. Il faut savoir que je reçois régulièrement des messages sur Twitter de personnes, et parfois d'entreprises, qui me demandent où j'en suis. La plupart de ces messages sont bienveillants. Les utilisateurs souhaitent juste l'utiliser chez eux en local, ou alors certaines entreprises me contactent pour affiner leur propre roadmap. Néanmoins je reçois parfois des messages pouvant se résumer à : *"bah alors t'en es où de l'opensource ?? Ca traine !"*. Ceux-là me font sourire, et dans ces cas-là je leur rappelle tout simplement que je ne leur dois rien.

Dans tous les cas je vous conseille de bien à réfléchir à votre roadmap avant d'annoncer quoi que ce soit, l'attente créée peut être énorme et on vous demandera tôt ou tard des comptes.

Ma deuxième erreur a été de vouloir trop en faire. Je me suis lancé dans un refacto complet du code, ce qui a impliqué beaucoup de changements que je n'avais pas prévu. J'ai finalement décidé de stopper ces évolutions avant l'opensource et de les proposer après. Qui sait, peut-être que ce sera la communauté elle-même qui les créera. J'en serai ravi !

And next ?
==========

L'ajout de la version 3 des CVSS a rajouté un bug lors de l'envoie des mails, j'ai découvert ça ce week-end et je dois le fixer. Une fois cela fait nous allons écrire les scripts pour migrer les données de la base actuelle vers le nouveau schema. Seulement à ce moment là, lorsque la nouvelle version sera disponible en ligne sur [Saucs.com](https://www.saucs.com), le code source pourra être libéré sur Github.

J'ai hâte que ce moment arrive, d'une part car je sais que l'attente a été longue et que certains d'entres vous sont impatients, et d'autre part car des développeurs m'ont déjà proposé leur aide pour de futures fonctionnalités.

Nous avons vraiment envie que ce projet serve au plus grand nombre, et cela ne peut passer que par l'opensource. C'est la dernière ligne droite, ça devient bon !