---
layout: ../../layouts/BlogPostLayout.astro
title: "GitHub Game Off 2024 - Retour d'expérience"
pubDate: 2024-19-13
description: "J'ai créé une fiction interactive avec React et YarnBound pour la GitHub Game Off 2024."
author: Sylvain Schellenberger
image:
  url: "/images/hackback-githubgameoff2024-cover.png"
  alt: "Logo de la GitHub Game Off 2024 sur fond bleu nuit."
tags: ["react", "javascript", "game developement"]
---

Le mois dernier, j'ai participé à la game jam[^1] [GitHub Game Off 2024](https://itch.io/jam/game-off-2024) lors de laquelle j'ai créé une fiction interactive sur le thème du hacking et à l'aide de React.js.

Vous pouvez y [jouer sur itch.io](https://raaaahman.itch.io/hackback), ou lire le [code source sur GitHub](https://github.com/raaaahman/atthack).

Comment  fonctionne une fiction interactive, et était-ce une bonne idée d'utiliser React pour en construire une? Je vous raconte. 

## Une fiction interactive dans une application web

La GitHub Game Off est une game jam qui se déroule durant tout le mois de Novembre. Le thème imposé lors de cette game jam tenait en un seul mot: secrets.

Pour m'aider à mener à bien ce projet, j'ai fait appel à [Endi Pons](https://www.linkedin.com/search/results/all/?fetchDeterministicClustersOnly=true&heroEntityKey=urn%3Ali%3Afsd_profile%3AACoAAA2YPEYBSDK6x0_Xx_dhbXK4cM9DddH5EAk&keywords=endi%20pons&origin=RICH_QUERY_SUGGESTION&position=0&searchId=845b3fd9-e6ac-45f4-8249-4babd9e9670d&sid=IyG&spellCorrectionEnabled=false) , qui est un UX designer avec une appétence pour la **gamification**[^3] et qui a bien voulu endosser la casquette de _narrative designer_.

!["Octalysis" du concept du jeu Hackback, réalisée par Endi Pons.](/images/hackback-octalysis.png)

Pour répondre au thème de la game jam, nous avons choisi d'écrire une fiction parlant du *hacking* (piratage), et plus particulièrement du **social engineering**, un ensemble de techniques utilisées par les pirates pour dérober des informations à leurs victimes. Si ces méthodes sont souvent exercées à des fins criminelles, nous avions toutefois choisi un contexte où elles aideront le joueur à déjouer une machination à base de logiciels espions et de réseaux clandestins.

## Utiliser React pour créer un jeu vidéo

Faire une **interface utilisateur** à l'aide de React, Tailwind et DaisUI n'a rien d'exceptionnel pour un développeur front-end. La relier à un moteur de fiction interactive est la partie qui m'a posé le plus de défis lors de cette jam.

Pour cette partie, j'ai choisi d'utiliser la bibliothèque [YarnBound](https://github.com/mnbroatch/yarn-bound/), qui permet d'interpréter la syntaxe d'écriture de dialogues [Yarn](https://www.yarnspinner.dev/), à ne pas confondre avec le gestionnaire de dépendances JavaScript du même nom. Cette bibliothèque inclut elle-même une autre bibliothèque [bondage.js](https://github.com/mnbroatch/bondage.js) qui permet de transformer les fichiers de texte brut contenu les scripts en Yarn, de les décomposer en objets JavaScript (les _nodes_) qui seront ensuite parcouru par un générateur[^4] et redécoupés en objets plus précis (les _results_), représentant chaque ligne du fichier texte, eux stockées dans une instance de la classe[^5] `YarnBound`.

![YarnBound permet de décomposer les fichiers Yarn en objets JavaScript qui seront ensuite utilisé dans l'application.](/images/yarn-to-app.png)

Cette conception **orientée objet** est assez difficilement conciliable avec React. A moins d'utiliser une autre fonctionnalité de JavaScript: les Proxies[^6]; ce que j'ai fait à travers la bibliothèque [Valtio](https://valtio.dev/) pour me simplifier la tâche.
## Défis d'implémentation

Au vu du plan que je m'étais fixé, j'imaginais la tâche assez simple. Oui, comme chaque projet "simple" en programmation, celui-ci aussi a vécu des retournements que je n'avais pas envisagés.

La bibliothèque YarnBound n'étant que très peu utilisée, je n'ai pas pu m'appuyer sur des tutoriels ni exemples, et j'ai du fouiller dans son code à maintes reprises. Elle est d'ailleurs prévu d'une certaine manière: le script du dialogue est chargé et converti lors de l'instanciation de la classe, puis parcouru au fur et à mesure que l'on appelle sa méthode `advance()` (ce qui se produit lorsque l'utilisateur clique dans notre application).

Sauf que notre application n'a pas qu'un seul fichier Yarn à lire et interpréter, mais plusieurs qui s'enchaînent sans interruptions claire. J'ai donc du concevoir une solution de **asynchrone** pour télécharger les fichiers au fur et à mesure que le joueur avance dans les dialogues.

![Architecture de l'application de fiction interactive.](/images/interactive-fiction-architecture.png)

Cette solution asynchrone aurait pu être facilitée par Valtio et le composant [Suspense](https://fr.react.dev/reference/react/Suspense) de React. Sauf que Valtio avait retiré la gestion des Promises[^8] de ses fonctionnalités, en prévision de la version 19 de React et de son hook [use](https://react.dev/reference/react/use) qui doit faciliter justement la gestion de cet asynchronisme. Ne connaissant pas encore cette nouvelle API, j'ai décidé de rester en terrain connu et d'utiliser des hooks mieux connus de React.

Malheureusement, ce code plein d'asynchronismes a généré un début de **dette technique** qui m'a ralenti par la suite, lorsque j'ai voulu ajouter des fonctionnalités supplémentaires, comme la sauvegarde des données du jeu.

## Défis de gestion de projet

L'autre difficulté a été de planifier les fonctionnalité et le contenu en parallèle, l'un dépendant beaucoup plus de l'autre dans le cas d'un jeu vidéo que d'une application web classique. J'avais par exemple conçu une première version de l'application qui prévoyait que chaque personnage fictif aie sa propre instance de YarnBound qui gère tous les dialogues relatif à ce personnage.

Or, en combinant en intégrant les dialogues écrits par mon camarade, il s'est avéré plus pratique de ne gérer qu'une seule instance de YarnBound pour gérer le jeu comme un seul même dialogue qui progresse à travers les différent personnages de manière séquentielle.

![Graphe des dialogues d'une des missions du jeu.](/images/hackback-mission3-nodegraph.png)

Egalement, c'est en testant les dialogues quasi-complets, un peu avant la date de publication, que je me suis rendu compte du manque de praticité pour tester le contenu: il fallait en effet constamment remettre les dialogues au début pour pouvoir vérifier que les modifications apportées au contenu corrigeaient bien le problème.

Créer une **boucle de rétroaction** courte est un atout essentiel dans un projet informatique, elle l'est d'autant plus lorsque les délais de complétion du projet sont si serrés. 
## Les avantages du web pour créer et diffuser des jeux

Construire les différentes pages et leurs composants n'a évidemment pas été trop difficile étant donné que c'est ce que je fais le plus en tant que développeur front-end. J'avais d'ailleurs gardé cette tâche pour la toute fin, me concentrant surtout sur les **règles métiers**. 

Grâce au **responsive web design**, nous avons également une version adaptée pour les smartphones, ce qui n'est pas le cas pour tous les autres participants de la GitHub Game Off 2024. Je pense d'ailleurs que la fiction interactive se prête plutôt bien au mobile en général, et le fait d'être un application web permet en plus de fonctionner quel que soit le système d'exploitation du téléphone (iOS ou Android).

![Le jeu est prévu pour être joué sur ordinateur comme sur smartphone](/images/hackback-responsive(2).png)

Le fait que le jeu ne demande pas de téléchargement d'un fichier exécutable est également un avantage pour permettre aux joueurs curieux de rapidement essayer le jeu.
## Conclusion

J'aime beaucoup l'exercice des game jams, qui me permettent à la fois de pratiquer mes compétences techniques, mais aussi de tester ma capacité à **organiser et prioriser** les tâches. Même si souvent cela fini en "crunch"[^2].

Participer à celle-ci avec Endi a ajouté une dimension de collaboration enrichissante. Je n'avais plus seulement les contraintes du thème et du temps, mais je devais aussi m'assurer que les développement que je produisais pouvaient servir à l'histoire et les interactions qu'il était en train de créer.

J'ai eu d'ailleurs beaucoup de plaisir à découvrir l'histoire, même si nous discutions des possibilités scénaristiques avant qu'il ne l'écrive. C'est un processus très interactif qui a permis, je pense, à s'assurer que la majorité des fonctionnalités soient utilisées et construites pour être conformes à l'utilisation qui en étaient prévues.

Ce projet n'est actuellement qu'au stade de prototype. Et nous avons des idées du côté du scénario comme du côté des fonctionnalités qu'il nous plairait d'implémenter. J'espère pouvoir y apporter des corrections et des ajouts dans les mois à venir (mais peut-être pas ce mois de décembre).

[^1]: Une game jam est un défi de programmation de jeux vidéos durant une durée déterminée, souvent sur un thème imposé. Un peu comme un hackathon.
[^2]: Le crunch est un terme bien connu des développeurs de jeux vidéos, et parfois des joueurs, qui désigne une période de travail intensive pour déployer le jeu dans les délais annoncés, incluant beaucoup d'heures supplémentaires, souvent impayées.
[^3]: La gamification, ou _ludification_, est une discipline qui tend à introduire des mécaniques et des problématiques inspirés des jeux, notamment des jeux vidéos, dans d'autres domaines, tel que des sites web ou des formations.
[^4]: Les générateurs / fonction génératrices sont une fonctionnalité de JavaScript. Voir [Generator, Référence Javascript, developers.mozilla.org](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Global_Objects/Generator).
[^5]: YarnBound utilise la syntaxe des classes JavaScript. Voir [Classes, Référence JavaScript, developers.mozilla.org](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Classes)
[^6]: Un Proxy est un objet JavaScript qui en encapsule un autre, permettant au code extérieur d'accéder aux propriétés et méthodes de l'objet encapsulé, tout en permettant d'intercepter les appels à ces dites propriétés et méthodes. Voir [Proxy, Référence JavaScript, developers.mozilla.org](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
[^7]: Les sous-modules sont une fonctionnalité de Git qui permet d'inclure certains dépôts comme dépendances d'autres dépôts. Voir [7.11 Utilitaires Git - Sous-Modules, Documentation, git-scm.com](https://git-scm.com/book/fr/v2/Utilitaires-Git-Sous-modules)
[^8]: Les Promises (promesses) sont des objets JavaScript qui permettent de traiter des valeurs de façon asynchrone. Voir [Promise, Référence JavaScript, developers.mozilla.network](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Global_Objects/Promise)