# Learning the Vue CLI: Lesson 1: Installing and Using the Vue CLI

The first step is to install the Vue CLI using npm:

```
nvm use stable
npm install -g @vue/cli
```

Notice we are using the `-g` flag here. This will install the Vue CLI globally,
so it is available to all your node projects. You will only need to install
the Vue CLI once, instead of installing it separately for each project.

We are also seeing the `@vue/cli` syntax for the first time. This allows npm
to separate packages into _scopes_, meaning Vue can have a package called `cli`,
and so can other organizations. We specify the scope `@vue` to say we want the
`cli` package from Vue and not from some other organization.

Now that the Vue CLI is installed, we can create a new project:

```
vue create learning-vue-cli
```

This is an interactive tool, so it will ask you a few questions:

- _Please pick a preset_: Use the arrow and enter keys to choose **Manually
  select features**.

- _Check the features needed for your project_: Use the arrow and space keys
  to choose **Router** and **Vuex**. Press enter when you are done.

- _Use history mode for router?_: **Y**

- _Pick a linter / formatter config_: Choose **ESLint with error prevention
  only**. If you don't already use a tool for automatically formatting your
  code, then you may try **ESLint + Prettier**, which will add this for you.

- _Pick additional lint features_: Choose **Lint on save**

- _Where do you prefer placing config for Babel, PostCSS, ESLint, etc?_:
  Choose **In dedicated config files**.

- _Save this as a preset for future projects?_: **Y**

- _Save preset as_: **Router + Vuex + ESLint**

It will then install a bunch of plugins for you.

After this completes, you will see these files and folders:

![folders](/screenshots/cli-folders.png)

- babel.config.js: configuration for babel, which compiles code for you

- postcss.config.js: configuration for
  tools that compile CSS

- public: static files for your application, include a basic `index.html`. You can store images in a folder here if you like.

- src: JavaScript files you will write and other assets

Let's take a look at the src folder:

![src folder](/screenshots/cli-src-folder.png)

- App.vue -- top-level Vue component for your app

- main.js -- configuration for your app

- router.js -- configuration for Vue Router

- store.js -- configuration for VueX

- assets -- images for your app

- components -- single file components for each part of your app

- views -- single file components that
  represent the different pages in your
  app

To understand how this works, your front
end will follow this architecture:

![front end architecture](/images/vue-architecture.png)

We'll go through each of these parts in more details in the next lesson.
