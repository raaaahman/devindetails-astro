---
layout: ../../layouts/BlogPostLayout.astro
title: "Créer une extension pour Vite"
pubDate: 2025-03-14
description: "Création d'une extension de concaténation de fichiers pour Vite."
author: Sylvain Schellenberger
image:
  url: "/images/vite-plugin.svg"
  alt: "Logo de la GitHub Game Off 2024 sur fond bleu nuit."
tags: ["vite", "javascript", "node.js"]
---

## Introduction

Si vous avez fait du développement front-end dans les cinq dernières années, vous avez probablement entendu parlé de Vite. Conçu par [Evan You](https://evanyou.me/), le créateur de Vue.js, Vite a connu un rapide succès et est utilisé avec un bon nombre de bibliothèques JavaScript différentes.

Cela est rendu possible par son système d'**extensions**. Les extensions peuvent intercepter et modifier différentes étapes du cycle de vie de Vite, comme la résolution des modules, la transformation des fichiers, ou la génération du bundle final.

Par exemple, pour transformer tous les fichiers contenant du JSX en fichiers JavaScript pouvant être lus par le navigateur, il existe l'extension [Vite Plugin React](https://github.com/vitejs/vite-plugin-react), créée et maintenue par l'quipe de développement de Vite. Après l'abandon en Février dernier de l'outil de développement Create React App[^1], Vite devient un des outils de choix pour développer des _applications monopages_[^2] avec React.

### Le besoin

J'avais précédemment écrit sur la fiction interactive[^3] que nous avions réalisé avec un ami lors d'une game jam l'année dernière. En reprennant le travail sur ce prototype, j'ai trouvé qu'il serait utile d'avoir un petit script qui me permette d'effectuer une transformation et une concaténation des fichiers de contenus (pour rappel, le jeu est écrit selon la syntaxe [YarnSpinner](https://www.yarnspinner.dev/)) pendant la _compilation_ du programme plutôt que pendant son exécution.

C'est similaire ce que fait l'extension Vite Plugin React, sauf que cette extension transforme uniquement des **fichiers sources**, c'est-à-dire des fichiers qui sont importés dans le code (via le mot clé `import` pour les modules EcmaScript[^4]). Or, pour faciliter la collaboration avec le _narrative designer_[^5] de l'équipe, j'ai préféré gérer les fichiers de contenus comme des **resources statiques**, qui sont téléchargés par l'application lors de l'exécution.

Il m'a donc fallu utiliser un mélange de l'API des extensions de Vite et de modules node.js pour faire des opérations sur ces fichiers.

## Extensions pour Rollup

Oui, vous avez bien lu, [Rollup.js](https://rollupjs.org/) est l'outil qui permet de compiler les fichiers en un _bundle_ plus adapté à la production (transpilation, code minifié, moins de requêtes HTTP, etc.). Vite est en réalité une surcouche qui utilise une pré-configuration de Rollup, mais pour la production uniquement. Nous verrons que pour le serveur de développement, Vite implémente sa propre logique.

### Création d'une extension pour Rollup

Que l'on s'intéresse à l'ajout de fonctionnalité pour la production ou le développement, l'enregistrement de l'extension dans Vite est le même:

```javascript file:vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react"; // L'extension officielle pour React.
import { vitePluginYarn } from "./dev/vitePluginYarn": // L'extension créée localement pour mon projet.

export default definConfig({
  plugins: [
    react(),
    vitePluginYarn()
  ]
})
```

On voit que mon extension est ajoutée à la liste des extensions du projet à partir de la valeur retournée par **une fonction**. C'est la manière conventionnelle de procéder, car elle permet de passer des options pour l'extension en paramètre.

```javascript file:dev/vitePluginYarn.js
const defaultOptions = {
  name: "vite-plugin-yarn",
};

export const vitePluginYarn = (options) => {
  // On surcharge les options par défaut avec les éventuelles options passées à la fonction d'initialisation.
  const pluginOptions = Object.assign(defaultOptions, options || {});

  return {
    // Le nom de l'extension est utile pour toutes les fonctionnalités de débogage par exemple.
    name: pluginOptions.name,
  };
};
```

L'extension est retournée sous forme d'objet qui doit satisfaire l'API des extensions Rollup[^7]. Bon, à ce stade là, mon extension ne fait pas grand chose, il va me falloir l'interfacer avec le système de compilation de Rollup.

### Anatomie d'une extension pour Rollup

- Les hooks
- Utilisation du PluginContext

## Serveur de développement Vite

- Différence avec la production
- Créer un middleware pour le serveur de développement

### Astuces

- Utiliser des closures plutôt que des propriétés d'objet

[^1]: Sunsetting Create React App (anglais), [react.dev/blog](https://react.dev/blog/2025/02/14/sunsetting-create-react-app)
[^2]: Application monopage (SPA), Glossaire MDN, [developer.mozilla.org](https://developer.mozilla.org/fr/docs/Glossary/SPA)
[^3]: Github Game Off 2024 - Retour d'expérience, [devindetails.fr](https://devindetails.fr/posts/github-gameoff-2024-postmortem)
[^4]: Les Modules JavaScript, Guide JavaScript, [developer.mozilla.org](https://developer.mozilla.org/fr/docs/Web/JavaScript/Guide/Modules)
[^5]: Narrative designer, [wikipedia.org](https://fr.wikipedia.org/wiki/Narrative_designer)
[^6]: Plugin API, [vite.dev](https://vite.dev/guide/api-plugin.html)
[^7]: Plugins Overview, [rollupjs.org](https://rollupjs.org/plugin-development/#plugins-overview)
