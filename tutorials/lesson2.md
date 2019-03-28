# Learning the Vue CLI: Lesson 2: The Architecture of a Vue Application

Let's explore what the Vue CLI has setup for us. Start by running:

```
npm run serve
```

This will run a webserver on localhost, port 8080, which you can visit in
your web browser:

![home page](/screenshots/vue-default-app-home.png)

## Vue Router

How did this page get rendered?

First, the URL is given to the Router in `src/router.js`:

```
import Vue from 'vue'
import Router from 'vue-router'
import Home from './views/Home.vue'

Vue.use(Router)

export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home
    },
    {
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (about.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import(/* webpackChunkName: "about" */ './views/About.vue')
    }
  ]
})
```

The Vue Router uses this configuration to choose which view to render from the `views` folder.

Most of this is easy to understand. The code defines a Router object that includes some configuration and then an array of `routes`. Each route has a path, a name, and a component. These components are stored in the `views` folder and are called `Home.vue` and `About.vue`.

I would recommend using the configuration shown for the `Home`, component, which is imported on the third line and then used in the first route.

The configuration shown for the `About` component is more advanced. I would recommend not using this for now. It uses [lazy loading](https://alligator.io/vuejs/lazy-loading-vue-cli-3-webpack/), an advanced feature that we won't cover now.

You can add an import statement:

```
import Home from './views/Home.vue'
import About from './views/About.vue'
```

and modify the path for the `About` component:

```
    path: '/about',
      name: 'about',
      component: About
```

This is simpler.

## Views

Once the Router decides which view to render, it accesses the view you imported from the `views` folder. Here is what the `About.vue` view looks like:

```
<template>
  <div class="about">
    <h1>This is an about page</h1>
  </div>
</template>
```

A `.vue` file is a single file component. It contains three sections:

- template: the HTML for the page
- script: the Vue and JavaScript for the page
- style: the CSS for the page

Not all components have all three sections. As you can see, this one contains only some HTML for the about page.

This is what the `Home.vue` view looks like:

```
<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'home',
  components: {
    HelloWorld
  }
}
</script>
```

This again has a `template` section for the HTML. It references an image in the `assets` folder.

In the `script` section, this view imports a component from `src/components` called `HelloWorld.vue`. This lets us access the component with the `HelloWorld` element in the template. We pass data to the component using attributes, such as the `msg` attribute.

The `default` section of the `script` contains the name of the view and the components it uses.

Notice, there is no CSS here.

## Components

A page can use as many components as it needs, and a component can likewise use its own components. Think of this structure as a tree, with the view as the parent and its components as its children.

Here is what the `src/components/HelloWorld.vue` component contains:

```
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      For a guide and recipes on how to configure / customize this project,<br>
      check out the
      <a href="https://cli.vuejs.org" target="_blank" rel="noopener">vue-cli documentation</a>.
    </p>
    ...
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
```

Note, I have shortened the template component so it is easier to view here.

In the `template` section, we can access the `msg` property that was given
to us as an attribute in the `template`
of `Home.vue`.

In the `script` section, we define the name of the component and list the properties
in a `props` object. The names of properties listed here have to match the
names given as attributes by a parent
component. So `msg` here is the same as `msg` in the template and `msg` used in the template of `Home.vue`.

In the `style` section, we can place any CSS styles. If the style uses `scoped` then the CSS is limited to only this component or view. If the `scoped` keyword is missing, then the styles are
globally applied. I recommend using `scoped` in all components and putting
global styles in `App.vue`.

## Menu

The navigation menu at the top of this site comes from `src/App.vue`:

```
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
```

In the template section, we use `router-link` to create links, so that these are resolved to components by the Router. We use `router-view` to tell the router to put the rendered component in that location.
So `router-view` is replaced with the
contents of the template for `Home.vue`
or `About.vue`, depending on which page
is active.

In the style section, we define global
CSS.

## Configuration

Configuration for the app is done in `src/main.js`:

```
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

Vue.config.productionTip = false

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')
```

Here we tell our Vue instance to use the router (Vue Router), store (Vuex), to render the app using `App.vue`, and to control the DOM in the div with the `#app` id.

The `mount` function tells Vue to find the `#app` div in `index.html` and replace it with the template defined in `App.vue`.

## index.html

You will find the entry to the application in `public/index.html`:

```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
  <title>learning-vue-cli-solution</title>
</head>

<body>
  <noscript>
    <strong>We're sorry but learning-vue-cli-solution doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
  </noscript>
  <div id="app"></div>
  <!-- built files will be auto injected -->
</body>

</html>
```

This is where you place the main index page for the application. Since Vue
controls everything in the `#app` div,
then we technically have only one web
page in the app. This style of writing
web applications is called a _single page application_. The Vue Router is replacing the DOM inside of the `#app` div each time it renders a view or component.

You could load Google fonts or Bootstrap here.
