# Learning Vue CLI: Lesson 3: Building an App

Let's build the simple version of the help ticket application.

## Back end

First, install the packages we'll need for the server:

```
npm install express mongoose
```

Then, create a directory `server` in the top level of the project. Create a file in `server/server.js`:

```
const express = require('express');
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: false
}));

const mongoose = require('mongoose');

// connect to the database
mongoose.connect('mongodb://localhost:27017/tickets', {
  useNewUrlParser: true
});

const tickets = require("./tickets.js");
app.use("/api/tickets", tickets);

app.listen(3000, () => console.log('Server listening on port 3000!'));
```

Then, create a file in `server/tickets.js` containing:x

```
const mongoose = require('mongoose');
const express = require("express");
const router = express.Router();

//
// Tickets
//

const ticketSchema = new mongoose.Schema({
  name: String,
  problem: String,
});

const Ticket = mongoose.model('Ticket', ticketSchema);

router.get('/', async (req, res) => {
  try {
    let tickets = await Ticket.find();
    return res.send(tickets);
  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});

router.post('/', async (req, res) => {
  const ticket = new Ticket({
    name: req.body.name,
    problem: req.body.problem
  });
  try {
    await ticket.save();
    return res.send(ticket);
  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});

router.delete('/:id', async (req, res) => {
  try {
    await Ticket.deleteOne({
      _id: req.params.id
    });
    return res.sendStatus(200);
  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});

module.exports = router;
```

## Configuration

When you use `npm run serve` with Vue CLI, it starts the webpack development server. This acts as the main web server for your application, which is the same role that nginx plays when you deploy the application.

This means we need to configure the webpack development server to send any requests for your REST API (anything starting with `/api`), to the node server we'll be running.

To do this, create a file called `vue.config.js` in the top level of your project directory. In this file, put the following:

```
module.exports = {
  devServer: {
    proxy: {
      '^/api': {
        target: 'http://localhost:3000',
      },
    }
}
```

## Front End

For the rest of this lesson, we will be modifying the front end. While we do this, run the back end with:

```
node server.js
```

Run the front end with:

```
node run serve
```

This will let you visit the website at `localhost:8080`. You will notice that the website automatically reloads every time you edit a file that modifies the front end.

## Title and Imports

We want to change the title of our application and import the `axios` library. To do this, edit `public/index.html` and change the title:

```
<title>Tickets</title>
```

In addition, add the following script tag at the end of the body:

```
  <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.18.0/axios.min.js" integrity="sha256-mpnrJ5DpEZZkwkE1ZgkEQQJW/46CSEh/STrZKOB/qoM=" crossorigin="anonymous"></script>
</body>
```

## Header

The header for the website is common to all web pages. So we'll modify the `template` section in `src/App.vue` to include a header. We'll also add a div to wrap the pages displayed by the router.

```
<template>
<div id="app">
  <div class="header">
    <h1>Ticket System</h1>
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>
  </div>
  <div class="content">
    <router-view />
  </div>
</div>
</template>
```

We also want to change the styles of the page, so we'll modify the `style` section of `src/App.vue` to have the following:

```
<style>
body {
  margin: 0px;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.header {
  display: flex;
  justify-content: space-between;
  background: #571845;
  padding: 10px 100px;
  color: white;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #fff;
}

#nav a.router-link-exact-active {
  color: #FFC300;
}

.content {
  padding: 10px 100px;
}
</style>
```

If you check the browser, you should see the header:

![header](/screenshots/header.png)

## Viewing Tickets

We want to show a list of all of the tickets on the home page.

First, modify the `template` section of `src/views/Home.vue`:

```
<template>
<div>
  <h1>Current Tickets</h1>
  <div v-for="ticket in tickets" v-bind:key="ticket._id">
    <hr>
    <div class="ticket">
      <div class="problem">
        <p>{{ticket.problem}}</p>
        <p><i>-- {{ticket.name}}</i></p>
      </div>
    </div>
  </div>
</div>
</template>
```

Likewise, modify the `script` section of `src/views/Home.vue`:

```
<script>
export default {
  name: 'home',
  data() {
    return {
      tickets: {}
    }
  },
  created() {
    this.getTickets();
  },
  methods: {
    async getTickets() {
      try {
        let response = await axios.get("/api/tickets");
        this.tickets = response.data;
      } catch (error) {
        console.log(error);
      }
    },
  }
}
</script>
```

Notice that the `script` section contains the Vue code you are used to writing. It is mostly identical except:

- Instead of creating the Vue instance here (it is created in `main.js`), we use an export statement

- The data must be a function with a return statement that returns a JavaScript object containing the data.

Since we have not provided a way to create tickets yet, we can test it with:

```
curl -X POST -d '{"name": "Daniel", "problem": "not another ticket system?!"}' -H "Content-Type: application/json" localhost:3000/api/tickets
```

If you reload the web page for the app, you should see at least this newly created ticket, as well as any others left over in your database from previous activities.

![view tickets](/screenshots/view-tickets.png)

## Creating Tickets

To create tickets, we will add a new page in the application. First, modify the navigation in `src/App.vue` to include a new link:

```
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/create">New Ticket</router-link> |
      <router-link to="/about">About</router-link>
    </div>
```

Notice that this link references a view at `/create`. You can create this component by creating a new file in `views/Create.vue`. Put this in the `template` section:

```
<template>
<div>
  <h1>Create a Ticket</h1>
  <form v-if="creating" @submit.prevent="addTicket">
    <input v-model="addedName" placeholder="Name"></p>
    <textarea v-model="addedProblem"></textarea>
    <br />
    <button type="submit">Submit</button>
  </form>
  <div v-else>
    <p>Thank you for submitting a ticket! We will respond shortly.</p>
    <p><a @click="toggleForm" href="#">Submit another ticket</a></p>
  </div>
</div>
</template>
```

This contains the form we have used previously. We have made one small change, which is to add a `v-if` directive to the form and use `v-else` to show a thank you message.

In the `script` section of this same file, add:

```
<script>
export default {
  name: 'create',
  data() {
    return {
      creating: true,
      addedName: '',
      addedProblem: '',
    }
  },
  methods: {
    toggleForm() {
      this.creating = !this.creating;
    },
    async addTicket() {
      try {
        let response = await axios.post("/api/tickets", {
          name: this.addedName,
          problem: this.addedProblem
        });
        this.addedName = "";
        this.addedProblem = "";
        this.creating = false;
        this.getTickets();
      } catch (error) {
        console.log(error);
      }
    },
  }
}
</script>
```

This code submits the form using the API for the server and toggles the `creating` variable to show either the form or the thank you message.

Finally, add some CSS that is specific to this view:

```
<style scoped>
input {
  font-size: 1.2em;
  height: 30px;
  width: 200px;
}

textarea {
  font-size: 1.6em;
  width: 100%;
  max-width: 500px;
  height: 100px;
}

button {
  margin-top: 20px;
  font-size: 1.2em;
}
</style>
```

You should now have a page that lets you submit tickets.

![submit tickets](/screenshots/submit-ticket.png)

![thank you](/screenshots/thank-you.png)
