<template>
  <div id="app">
    <h1>Back Pocket</h1>
    <h4>A cookbook</h4>
    <button @click="randomize">Randomize</button>
    <a class="recipe-link" v-for="recipe in recipes" :key="recipe" @click="getRecipe(recipe)">{{ getDisplayName(recipe) }}</a>
    <div v-if="recipeHtml" v-html="recipeHtml"></div>
    <p v-else>This recipe book is not intended to show you <em>how</em> to cook, so much as it is to give you ideas for <em>what</em> to cook. I find myself frequently just forgetting about favorite recipes, and this basically exists to remind me of them and how to make them. Feel free to use the randomize button to shake things up! I've curated this around my own tastes, but if you aren't a fan of the recipes here, the code for this site is open source! - Koby</p>
  </div>
</template>

<script>
import { marked } from 'marked';

export default {
  name: 'App',
  data() {
    return {
      recipes: [],
      recipeHtml: ''
    }
  },
  methods: {
    async getRecipe(recipe) {
      const res = await fetch(recipe);
      const markdown = await res.text();
      this.recipeHtml = marked.parse(markdown, {
        breaks: true
      });
    },
    randomize() {
      this.recipes = this.recipes
        .map(r => ({ r, order: Math.random()}))
        .sort((a, b) => a.order - b.order)
        .map(({r}) => r);
    },
    async loadRecipes() {
      // This is a janky way to do this, but it means that I don't need a backend... soooo
      const recipeListHtml = await (await fetch('/recipes/')).text();
      const el = document.createElement('html');
      el.innerHTML = recipeListHtml;
      console.log(recipeListHtml);

      const recipes = [...el.getElementsByTagName('a')]
        .filter(aTag => aTag.pathname?.includes('.md'))
        .map(aTag => location.hostname === 'localhost'
          ? aTag.pathname
          : '/recipes/' + aTag.pathname);
      this.recipes = recipes;
    },
    getDisplayName(recipe) {
      const recipePathParts = recipe.split('/');
      return decodeURIComponent(
        recipePathParts[recipePathParts.length - 1].split('.')[0].replaceAll('_', ' ')
      );
    }
  },
  async created() {
    await this.loadRecipes();
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  align-items: center;
  display: flex;
  flex-direction: column;
  text-align: center;
}

.recipe-link {
  display: block;
  cursor: pointer;
  color: blue;
  padding: 0.25em;
}

button {
  display: block;
  padding: 0.5em;
  margin: 0.5em;
}

h1 {
  margin-bottom: 0em;
}
</style>
