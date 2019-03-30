# Learning Vue CLI: Lesson 3: Using Vuex to Manage State

The Vuex library helps you to manage the state of a large Vue application. You can ignore this for smaller applications.

Consider an application with many views and components. Many of them may need to store state for the application. If different components each store a list of tickets, for example, how do you keep them all in sync? If many components are calling a REST API on the server, how do you avoid having all of this logic scattered throughout the application?

This is where Vuex can help Âou with state management.

## Overview of Vuex

To understand Vuex, it is helpful to see how its architecture works:

![vuex architecture](/screenshots/vuex.png)

A Vue component calls an action on the store, which uses a mutation to change the state, which is then rendered in a component.

For example, a component might call the `getItems` action to fetch items from the back end, use the `setItems` mutation to store these as state, and then a component can use `$store.state.items` to render them

The Vue CLI has created Vuex state for us in `src/store.js`:

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {

  },
  mutations: {

  },
  actions: {

  }
})
```

You can see that it contains sections for each of these things.

## State for Tickets

The tickets are a good place to use state in our application. Edit `src/state.js` so it contains the following:

```
import Vue from 'vue';
import Vuex from 'vuex';
import axios from 'axios';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    tickets: [],
  },
  mutations: {
    setTickets(state, tickets) {
      state.tickets = tickets;
    },
  },
  actions: {
    async getTickets(context) {
      try {
        let response = await axios.get("/api/tickets");
        context.commit('setTickets', response.data);
      } catch (error) {
        console.log(error);
      }
    }
  }
});
```

Notice that we need to import `axios` here.

The `state` section in Vuex is similar to the `data` section in a component.

Mutations modify the state. Each mutation takes `state` as its first parameter and then an optional second parameter.

Actions are similar to methods in a component. To modify state from within an action, use `context.commit`, specifying the name of the mutation and then passing its parameters.

## Modifying the Home View

In `src/views/Home.vue`, we can modify the `script` section to contain the following:

```
<script>
export default {
  name: 'home',
  computed: {
    tickets() {
      return this.$store.state.tickets;
    }
  },
  created() {
    this.$store.dispatch("getTickets");
  }
}
</script>
```

Notice that we don't have to import `axios` here. We also do not keep any data, because the tickets have moved to the store.

Instead, we use a computed property to return the state of the tickets from the store. The created method dispatches the `getTickets` action defined in the store.

## Using Vuex to Create Tickets

Let's add another action to the store, for creating tickets:

```
    async addTicket(context, data) {
      try {
        await axios.post("/api/tickets", data);
      } catch (error) {
        console.log(error);
      }
    },
```

Now we can modify the `addTicket` method in `src/views/Create.vue` to use this:

```
addTicket() {
  this.$store.dispatch("addTicket", {
    name: this.addedName,
    problem: this.addedProblem
  });
  this.addedName = "";
  this.addedProblem = "";
  this.creating = false;
}
```

You should also remove the `axios` import since we are not using it here.

## What was the point?

Your application should work the same as before. The benefits of Vuex come from managing state for large applications, and coordinating state changes between multiple components in your application. For example, it can be helpful to store a record for the currently logged in user in Vuex so that every component has access to it.
