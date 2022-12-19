# Documentation

## Lab 6

### Jarom Ti'a

### IT&C 210B

Include an appropriate title, your name, the class, and semester

<div style="page-break-after: always;"></div>

## Table of Contents: 

- Overview
- Design
- Detailed Documentation
- Tips and warnings
- References

# Procedures

## **Overview** 

In this lab you will create a RESTful API to handle all of your requests for you. You will also create a front-end website for those REST API's so that other users can interact with a todo list. The front-end portion of the web application will be publically available but the API's will not be.

## Technologies

- Javascript
- NodeJS
- Vue.js
- Mongoose.js
- NPM (Node Package Manager)
- Google Oauth2
- JSON
- CI/CD Pipelines
- XHR Request
- Progressive Web Apps

These technologies are used so that the user can be able to read, add, delete, and update tasks.

## Design: 

Users will type in a new to do item and click submit. This will call the create new task route, which will have NodeJS add a new task to the MongoDB database and the current Users will then see the new task displayed on the front-web application

Users will click the checkbox button. This will call the update task route, which will have NodeJS change the task's "done" id to the opposite of what it was to the MongoDB database and the current Users will then see a checkbox check or unchecked on the task that they clicked

Users will click the delete button. This will call the delete task route, which will have NodeJS delete the current task to the MongoDB database and the current Users will then see the task disappear as it is deleted

## Detailed Documentation: 

Lab-6A The web API's
- Models file
    - Task.js - The model for the mongo database
- Routes file
    - auth.js - The authentication end points that allow the user to log in or log out using google Oauth2
    - tasks.js - The CRUD functionality using different endpoints
    - user.js - The user endpoint which stores the user variables and information
- index.js - Allows front end applications to connect to the web api's
- mongoose.js - Creates a connection with mongoose
- mssqldb.js - Connects to the Azure Database
- passport.js - Stores the google authentication variables

Lab-6B The front-end part of the web API's
- src file
    - components file
        - AppBar.vue - Shows the navbar for the user which allows the user to logout
        - NewTaskForm.vue - The component for the individual tasks which will have a text field, date, and submit button
        - TaskList.vue - The component for the individual tasks. Contains the checkbox, task text, date, and delete button
    - router file
        - index.js - Whenever the user goes to a different page the index.js will reroute them to the specific endpoints using the backend web API's
    - util file
        - index.js - Stores the cookies and formats the date
    - views file
        - About.vue - Has information of what the project is about
        - #### Home.Vue - The home page when the user logs in. This will display the task list through it's CRUD method

          - Read task - Fetches the /tasks ends point and returns the response in json format

          - Create task - Fetches the /tasks end point and sends a POST request with the following body
          ```
          {
          method: `POST`,
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            Text: form.Text,
            Date: formatDate(form.Date),
          }),
          credentials: `include`,
          }
          ```
          It then checks the response to make sure it was successful or not and then if it was successful it will display the updated task list.

          - Update task - Fetches the /tasks/task_id endpoint sending a PUT request with the following body
          ```
          {
          method: `PUT`,
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ Done: !task.Done }),
          credentials: `include`,
          }
          ```
          It then checks the response to make sure it was successful or not and then if it was successful it will display the updated task list.

          - Delete task - Fetches the /tasks/task_id endpoint sending a PUT request with the following body
          ```
          {
          method: `PUT`,
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ Done: !task.Done }),
          credentials: `include`,
          }
          ```
          It then checks the response to make sure it was successful or not and then if it was successful it will display the updated task list.

        - Login.vue - The first page that a user will see before logging in. This is also the page that the user will be redirected to once they log in

## Tips & Warnings: 

1. If errors occur the most common error derive from the AppBar.vue and the Home.vue since they are tightly connected. When you look at console.log errors it will sometimes show that the error is with the AppBar.vue but sometimes this is actually a problem with how the Home.vue is interacting with AppBar.vue.

2. When completing this lab make sure to double check your .gitignore to make sure it ignores the .env file when pushing it to the github repo. If you do not do this then you will have private information leaked onto a public repository that you do not want others to see.

3. If you are experiencing a blank web page after logging in it is because your CRUD methods within Home.vue are not working properly. You will have to make sure that read functions properly else you will not see anything appear. If the nav bar does not show up then it is your AppBar.vue that does not work

## References: 

1. Google Identity - OAuth 2.0 https://developers.google.com/identity/protocols/oauth2/scopes, Google Identity, accessed December 2021.

2. Vue.js - List Rendering https://vuejs.org/v2/guide/list.html, Vue.js, accessed December 2021

3. Vue.js - Getting started https://012.vuejs.org/guide/, Vue.js, accessed December 2021

## Documented Code: 

Lab-6A The web API's
- Models file
    - Task.js - The model for the mongo database
    ```
    /*

    This file is for creating a Mongoose Model/Schema

    */

    // Schema Definition
    var mongoose = require('mongoose')

    var Task = new mongoose.Schema({
    UserId:  String, // String is shorthand for {type: String}
    Text:   String,
    Done: Boolean, // Boolean is shorthand for {type: Boolean}
    Date: String
    });

module.exports = mongoose.model('Task', Task);
    ```
- Routes file
    - auth.js - The authentication end points that allow the user to log in or log out using google Oauth2
    ```
    const express = require(`express`);
    const passport = require("passport");
    const router = express.Router();


    router.get(
    "/google",
    passport.authenticate("google", {
        scope: [`https://www.googleapis.com/auth/userinfo.email` , `https://www.googleapis.com/auth/userinfo.profile`],
    })
    );

    router.get(
    "/google/callback",
    passport.authenticate("google", { failureRedirect: "/login" }),
    function (req, res) {
        try {
        req.session.save();
        res.redirect(process.env.CLIENT_ORIGIN);
        } catch (error) {
        console.error(error);
        res.redirect(`${process.env.API_ORIGIN}/api/v1/auth/logout`);
        res.status(500).send(`Something went wrong.`);
        }
    }
    );

    router.get(`/logout`, async (req, res) => {
    req.session.destroy();
    req.logout();
    res.redirect(process.env.CLIENT_ORIGIN);
    });

    module.exports = router;

    ```
    - tasks.js - The CRUD functionality using different endpoints
    ```
    const express = require(`express`);
    const router = express.Router();

    const Task = require(`../models/Task`);

    /**
    * GET: Returns one task with the task's id specified in the path
    */
    router.get(`/:id`, async (req, res) => {
    try {
        const task = await Task.findById(req.params.id);
        if (!task)
        res.status(404).send(`Task with ID ${req.params.id} does not exist.`);
        else res.status(200).send(task);
    } catch (error) {
        console.error(error);
        res.status(500).send(`Something went wrong.`);
    }
    });

    /**
    * POST: requests one task with body text and header data
    */
    router.post(`/`, async (req, res) => {
    try {
        if(!req.body.Text || !req.body.Date) return res.status(500).send("Insufferable data")
        const new_task = await new Task({
        UserId: req.user.Id,
        Text: req.body.Text,
        Done: false,
        Date: req.body.Date,
        });
        await new_task.save();
        res.status(200).send(new_task);
    } catch (error) {
        console.error(error);
        res.status(500).send(`Something went wrong.`);
    }
    });

    /**
    * GET: Returns all of the tasks from the current user
    */
    router.get(`/`, async (req, res) => {
    try {
        const task_list = await Task.find({ UserId: req.user.Id });
        if (!task_list) res.status(404).send(`Tasks do not exist`);
        else res.status(200).send(task_list);
    } catch (error) {
        console.error(error);
        res.status(500).send(`Something went wrong.`);
    }
    });

    /**
    * PUT: Requests the task_id boolean to be switched
    */
    router.put(`/:id`, async (req, res) => {
    try {
        let task = await Task.findById(req.params.id);
        if (!task) return res.status(404).send("Task does not exist");
        task = await Task.updateOne(
        { _id: req.params.id },
        { Done: req.body.Done }
        ); //req.params.id
        task = await Task.findById(req.params.id);

        res.status(200).send(task);
    } catch (error) {
        console.error(error);
        res.status(500).send(`Something went wrong.`);
    }
    });

    /**
    * DELETE: Requests the task with task_id to be deleted from the database
    */
    router.delete(`/:id`, async (req, res) => {
    try {
        let task = await Task.findById(req.params.id);
        if (!task) return res.status(404).send("Task does not exist");
        task = await Task.deleteOne({ _id: req.params.id });
        if (!task)
        res.status(404).send(`Task with ID ${req.params.id} did not delete.`);
        else res.status(200).send(task);
    } catch (error) {
        console.error(error);
        res.status(500).send(`Something went wrong.`);
    }
    });

    module.exports = router;

    ```
    - user.js - The user endpoint which stores the user variables and information
    ```
        /**
    * This file is an example of a endpoint.
    * GET: Returns the current user object after 
    * being authenticated. 
    */
    const router = require(`express`).Router()

    router.get(`/`, async (req, res) => {
        try {
            res.status(200).send(req.user)
        } catch (error) {
            console.error(error)
            res.status(500).send(`There was a problem getting the user.`)
        }
    })

    module.exports = router

    ```
- index.js - Allows front end applications to connect to the web api's
    ```
    const express = require(`express`);
    const cookieParser = require(`cookie-parser`);
    const logger = require(`morgan`);
    const session = require(`express-session`);
    const cors = require(`cors`);
    const { authenticate } = require(`./util`);
    const passport = require("passport");
    const store = require(`./passport`)(session);

    // Here, you should require() your mssqldb, mongoose, and passport setup files that you create

    require("./mssqldb");
    require("./mongoose");

    // Here, you should require() your routers so you can use() them below
    const userRouter = require(`./routes/user`);
    const taskRouter = require("./routes/tasks");
    const authRouter = require("./routes/auth");

    const app = express();

    // These lines are provided for you.
    app.use(
    cors({
        origin: process.env.CLIENT_ORIGIN,
        credentials: true,
    })
    ); // CORS will allow a front end specified in the .env to have access to restricted resources.
    app.use(logger(`dev`)); // This line is for having pretty logs for each request that your API receives.
    app.use(express.json()); // This line says that if a request has a body, that your api should assume it's going to be json, and to store it in req.body
    app.use(express.urlencoded({ extended: false })); // this line says that if there's any URL data, that it should not use extended mode.
    app.use(cookieParser()); // This line says that if there are any cookies, that your app should store them in req.cookies

    // Here is where you should use the `express-session` middleware

    const sess = {
    name: "it210_session",
    secret: process.env.SESSION_SECRET,
    cookie: { maxAge: 604800000 },
    resave: false,
    saveUninitialized: true,
    store,
    };

    if (app.get("env") === "production") {
    app.set("trust proxy", 1); // trust first proxy
    sess.cookie.secure = true; // serve secure cookies
    sess.cookie.sameSite = "none"; // keeps cookies from being blocked for this lab
    }

    app.use(session(sess)); // use the session with the attributes defined above
    app.use(passport.initialize()); // This line is to initialize the authentication
    app.use(passport.session()); // This line is for the session variables

    // Here is where you should assign your routers to specific routes. Make sure to authenticate() the routes that need authentication.
    app.use(`/api/v1/user`, authenticate, userRouter);
    // app.use('/api/v1/user', userRouter); // TODO: Get rid of this once done testing
    app.use("/api/v1/tasks", authenticate, taskRouter);
    app.use("/api/v1/auth", authRouter);

    module.exports = app;

    ```
- mongoose.js - Creates a connection with mongoose
    ```
    const mongoose = require('mongoose');

    //create a connection with the MongoDB
    mongoose.connect(process.env.ATLAS_CONNECTION_STRING);

    const db = mongoose.connection;
    //Log error if unable to connect
    db.on('error', console.error.bind(console, 'connection error: '));
    //Log success message if connected
    db.once('open', function() {
        console.log("Mongo Db connected")
    });
    module.export = db
    ```
- mssqldb.js - Connects to the Azure Database
    ```
     /*
    This is a file you probably should not edit.

    This code sets up a connection to the Azure Database 
    you should have created in lab 4B when you published 
    your application to Azure. 
    
    The database holds user information we will use for 
    authentication. 

    You will have to set up a similar connection to your
    MongoDB database in the mongoose.js file. 

    */

    const { Connection } = require(`tedious`)

    const mssqldb = new Connection({
        authentication: {
            options: {
                // These variables have to be defined in your .env file
                userName: process.env.AZURE_DB_ADMIN_USERNAME,
                password: process.env.AZURE_DB_ADMIN_PASSWORD,
            },
            type: "default"
        },
        server: process.env.AZURE_SERVER_NAME,
        options: {
            database: process.env.AZURE_DB_NAME,
            encrypt: true
        }
    })
    // If connected to Azure, a message will be displayed in console
    mssqldb.connect( err => {
        if (err) console.error(err.message)
        else console.log(`User Database Connected`)
    })

    module.exports = mssqldb

    ```
- passport.js - Stores the google authentication variables
    ```
    /*

    This file is for initializing passport.
    Some of this file is provided for you.
    Read this file and try to understand what
    is happening.

    */

    // Require passport dependency
    var passport = require("passport");
    // Here you will require() anything else you need
    var GoogleStrategy = require("passport-google-oauth").OAuth2Strategy;

    const { getUserFromAzure } = require("./util");

    // This defines what will be in the session cookie
    passport.serializeUser(function (user, done) {
    done(null, user);
    });
    // Find the user from the session and use result in callback function
    passport.deserializeUser(async (user, done) => {
    try {
        done(null, user);
    } catch (error) {
        console.error(error);
        done(error.message);
    }
    });

    // Here you will set up a connection to Google using variables from your .env file
    passport.use(
    new GoogleStrategy(
        {
        clientID: process.env.GOOGLE_CLIENT_ID,
        clientSecret: process.env.GOOGLE_CLIENT_SECRET,
        callbackURL: `${process.env.API_ORIGIN}${process.env.GOOGLE_CALLBACK_PATH}`,
        },
        async function (token, tokenSecret, profile, done) {
        var user = await getUserFromAzure(profile.emails[0].value);
        var userProfile = profile.photos[0].value
        var displayName = profile.displayName
        user.name = displayName
        user.photo = userProfile
        return done(null, user);
        }
    )
    );

    // Initilize Session storage in MongoDB
    const initStore = (session) => {
    const MongoDbStore = require(`connect-mongodb-session`)(session);
    const store = new MongoDbStore(
        {
        uri: process.env.ATLAS_CONNECTION_STRING,
        collection: `Sessions`,
        },
        (err) => {
        if (err) console.error(err);
        else console.log(`Session Store Initialized`);
        }
    );
    store.on(`error`, console.error);
    return store;
    };

    module.exports = initStore;

    ```

Lab-6B The front-end part of the web API's
- src file
    - components file
        - AppBar.vue - Shows the navbar for the user which allows the user to logout
            ```
            <template>
                <div id="app-bar">
                    <v-app-bar app color="primary" dark>
                    <v-toolbar-title>  App</v-toolbar-title>

                    <v-spacer></v-spacer>

                    <!--  : Use a v-if to only show this button when the user is logged in -->
                    <v-btn icon v-on:click.stop="drawer = !drawer" v-if="loggedIn">
                        <!-- auth.isAuthenticated checks whether user is logged in or not -->
                        <v-icon>mdi-menu</v-icon>
                    </v-btn>
                    </v-app-bar>

                    <v-navigation-drawer v-model="drawer" absolute temporary right>
                    <v-list-item>
                        <!--   (extra-credit): If you choose to do the extra credit, comment out the following line -->
                        <!-- <v-list-item-title>{{ user.UserName }}</v-list-item-title> -->

                        <!--   (extra-credit): This is where you would put the image you get from Google's OAuth 2 scope -->
                        <!-- If the image's URL is a string on the user object, pass it into the :src attribute -->
                        <v-list-item-avatar>
                        <v-img :src="user.photo"></v-img>
                        </v-list-item-avatar>

                        <!--   (extra-credit): This is where you would put the user's email from Google -->
                        <!-- Remember, use mustache syntax {{}} to pass the value of a variable into this HTML -->
                        <v-list-item-content>
                        <v-list-item-title>{{ user.UserName }}</v-list-item-title>
                        </v-list-item-content>
                    </v-list-item>

                    <v-divider></v-divider>

                    <v-list dense>
                        <!--  : Add an :href attribute to this tag so that clicking this link will log a user out -->
                        <v-list-item link :href="`${localhost}/api/v1/auth/logout`">
                        <!-- HINT: When there's a colon before an attribute, that's a short-hand for v-bind: which means -->
                        <!-- that in the quotation marks of that attribute, you can put JavaScript! That means you can -->
                        <!-- use a backtick string to concatenate the api origin env variable to the logout path! -->

                        <v-list-item-icon>
                            <v-icon>mdi-logout-variant</v-icon>
                        </v-list-item-icon>

                        <v-list-item-content>
                            <v-list-item-title>Log Out</v-list-item-title>
                        </v-list-item-content>
                        </v-list-item>
                    </v-list>
                    </v-navigation-drawer>
                </div>
                </template>

                <script>
                import { authenticated } from "../util";
                export default {
                name: "AppBar",
                props: {
                    //  : Pass in the user object to this component as a prop
                    user: {
                    Type: Object,
                    default: () => {
                        "user";
                    },
                    },
                    // Make sure to give it a default value that has the UserName defined as an empty string
                },
                data: () => ({
                    drawer: false,
                    localhost: process.env.VUE_APP_API_ORIGIN,
                    //  : Get the api's url origin from process.env and set it to a variable here
                }),
                asyncComputed: {
                    loggedIn: {
                    get: async () => await authenticated(),
                    default: false,
                    },
                },
                };
                </script>
            ```
        - NewTaskForm.vue - The component for the individual tasks which will have a text field, date, and submit button
        ```
        <template>
            <div id="new-task-form">
                <!--when the user clicks this button it submits the form-->
                <!--  : Add a submit event handler attribute to handle the form submission -->
                <!-- HINT: Remember to pass in the form data to the method so it can pass it to your API -->
                <!-- HINT: Also, you can add .prevent to the end of the attribute's key to keep the page from reloading -->
                <v-form @submit.prevent="createTask(form)">
                <v-container>
                    <v-row>
                    <v-col cols="12" xs="12" sm="6">
                        <!--  : Bind this input to the Text property value of the form prop -->
                        <v-text-field
                        label="Add a task..."
                        prepend-icon="edit"
                        v-model="form.Text"
                        required
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12" xs="12" sm="4">
                        <v-menu
                        v-model="menu"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="290px"
                        >
                        <template v-slot:activator="{ on }">
                            <!--  : Bind this input to the Date property value of the form prop -->
                            <v-text-field
                            label="When should the task be done?"
                            prepend-icon="event"
                            v-model="form.Date"
                            v-on="on"
                            ></v-text-field>
                        </template>

                        <!--  : Also bind this input to the Date property value of the form prop -->
                        <v-date-picker
                            @input="menu = false"
                            v-model="form.Date"
                        ></v-date-picker>
                        </v-menu>
                    </v-col>
                    <v-col cols="12" xs="4" offset-xs="4" sm="2">
                        <v-btn class="mt-3" type="submit" text color="success" block>
                        <v-icon left>mdi-plus</v-icon>
                        Add Task
                        </v-btn>
                    </v-col>
                    </v-row>
                </v-container>
                </v-form>
            </div>
            </template>

            <script>
            import { getCurrentDate, formatDate } from "../util";
            export default {
            name: "NewTaskForm",
            props: {
                //  : Figure out any props that you'll need from this component's parent and add it/them here
                // Remember to add the prop as an attribute to this component's html tag
                form: { Type: Object },
                createTask: { Type: Function },
            },
            data: () => ({
                menu: false,
            }),
            methods: {},
            };
            </script>

            <style scoped>
            form {
            padding: 0 1rem;
            }
            </style>
        ```
        - TaskList.vue - The component for the individual tasks. Contains the checkbox, task text, date, and delete button
        ```
        <template>
            <div id="task-list">
                <v-list-item-group>
                <!--  : Add a v-for attribute on this tag -->
                <!-- Remember to use the :key attribute to define a key for each task! -->
                <v-list-item v-for="task in tasks" :key="task._id">
                    <template>
                    <v-list-item-action>
                        <!--  : Add a click event handler attribute here that updates an task -->
                        <v-btn icon @click="updateTask(task)">
                        <!--  : Add a v-if and v-else check here based on the current task's Done property -->
                        <v-icon v-if="task.Done">check_box</v-icon>
                        <v-icon v-else>check_box_outline_blank</v-icon>
                        </v-btn>
                    </v-list-item-action>
                    <v-list-item-content>
                        <v-list-item-title>
                        <!--  : Show the value of the task's Text property here -->
                        {{ task.Text }}
                        </v-list-item-title>
                        <v-list-item-subtitle>
                        <!--  : Show the value of the task's Date property here -->
                        {{ task.Date }}
                        </v-list-item-subtitle>
                    </v-list-item-content>
                    <v-list-item-action>
                        <!--  : Add a click event handler attribute here that deletes this task -->
                        <v-btn icon>
                        <v-icon color="red" @click="deleteTask(task)">delete</v-icon>
                        </v-btn>
                    </v-list-item-action>
                    </template>
                </v-list-item>
                </v-list-item-group>
            </div>
            </template>

            <script>
            export default {
            name: "TaskList",
            props: {
                //  : Figure out what you need to bring in from props, then and then add those variables here
                // Make sure to include these props on the component's tag when using it in another file
                // HINT: For the tasks prop, set the default value to:
                // default: () => []
                updateTask: { Type: Function },
                deleteTask: { Type: Function },
                form: { Type: Object },
                tasks: { default: () => [] },
            },
            };
            </script>

        ```
    - router file
        - index.js - Whenever the user goes to a different page the index.js will reroute them to the specific endpoints using the backend web API's
            ```
            import Vue from "vue";
            import VueRouter from "vue-router";
            import Home from "../views/Home.vue";
            import {authenticated} from "../util";
            import Login from "../views/Login.vue";

            Vue.use(VueRouter);

            const checkAuth = async (to, from, next) => {
            try {
                if (await authenticated()) next();
                else
                next({
                    path: "/login",
                    replace: true,
                });
            } catch (error) {
                console.error(error.message);
                next({
                path: "/login",
                replace: true,
                });
            }
            };

            const routes = [
            {
                path: "/",
                name: "Home",
                component: Home,
                beforeEnter: checkAuth,
                props: true,
            },
            {
                path: "/about",
                name: "About",
                // route level code-splitting
                // this generates a separate chunk (about.[hash].js) for this route
                // which is lazy-loaded when the route is visited.
                component: () =>
                import(/* webpackChunkName: "about" */ "../views/About.vue"),
            },
            {
                path: "/login",
                name: "Login",
                component: Login,
            },
            ];

            const router = new VueRouter({
            mode: "history",
            base: process.env.BASE_URL,
            routes,
            });

            export default router;
            ```
    - views file
        - #### Home.Vue - The home page when the user logs in. This will display the task list through it's CRUD method
        ```
        <template>
            <div class="home">
                <v-card class="mx-auto" max-width="900" v-if="fetched">
                <!--  : Add your NewTaskForm component -->
                <!-- Make sure to pass in any necessary props -->
                <NewTaskForm :createTask="createTask" :form="form" />

                <v-divider></v-divider>
                <v-list subheader two-line flat>
                    <v-subheader>
                    <!--  : use the `user` prop to display the user's username here -->{{
                        user.UserName
                    }}'s Tasks:
                    </v-subheader>

                    <!--  : Add your TaskList component -->
                    <!-- Make sure to pass in any necessary props -->
                    <TaskList
                    :tasks="tasks"
                    :deleteTask="deleteTask"
                    :updateTask="updateTask"
                    />
                </v-list>
                </v-card>
            </div>
            </template>

            <script>
            import { getCurrentDate, formatDate } from "../util";
            //  : Import the components you want to use from their files
            import NewTaskForm from "../components/NewTaskForm.vue";
            import TaskList from "../components/TaskList.vue";

            export default {
            name: "Home",
            components: {
                //  : Use the Vue Documentation to find out how to use this property
                NewTaskForm,
                TaskList,
            },
            props: {
                //  : Add the user object as a prop that's passed in from the App.vue component
                user: {
                Type: Object,
                default: { UserName: "" },
                },
            },
            data: () => ({
                fetched: false, // This keeps us from getting an error when the page loads, but there's no data
                tasks: [], // This will hold the list of tasks you get from your API
                form: {
                Text: "",
                Date: getCurrentDate(),
                //  : Add 2 properties to this form data object to track the Text and the Date
                // HINT: Capitalize the "Text" and "Date" properties to make it easier to pass the data to your API
                // HINT: You can use the `getCurrentDate()` method to get today's date in the proper format for the datepicker
                },
            }),
            mounted() {
                //  : Call the method that gets the tasks from your API
                this.readTasks();
            },
            methods: {
                createTask(form) {
                //  : Use fetch() to send a POST request to your API that includes the data from this.form
                console.log("create");
                fetch(
                    // The first parameter is a string that contains the full URL to your endpoint
                    `${process.env.VUE_APP_API_ORIGIN}/api/v1/tasks`,
                    // The second parameter is an object with options. You can include request
                    // headers here, options for credentials, which method, which mode, etc.
                    {
                    method: `POST`,
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({
                        Text: form.Text,
                        Date: formatDate(form.Date),
                    }),
                    credentials: `include`,
                    }
                    // Note: The default for method is GET, so you don't need to include the
                    // method on any GET requests.
                ).then((response) => {
                    // Here we're just checking if the response was successful or not before
                    // trying to do anything about it.
                    if (response.ok) {
                    this.form.Text = "";
                    this.form.Date = getCurrentDate();
                    // If it is successful, we want to update the task list.
                    this.readTasks();
                    }
                });
                //  : Remember to get the updated task list when it's done
                //  : Remember to reset the values in this.form to their initial values when it's done
                },
                readTasks() {
                //  : Use fetch() to send a GET request to your API and update this.fetched and this.tasks with the data that's returned
                console.log("read");

                fetch(
                    // The first parameter is a string that contains the full URL to your endpoint
                    `${process.env.VUE_APP_API_ORIGIN}/api/v1/tasks`,
                    // The second parameter is an object with options. You can include request
                    // headers here, options for credentials, which method, which mode, etc.
                    {
                    credentials: `include`,
                    }
                    // Note: The default for method is GET, so you don't need to include the
                    // method on any GET requests.
                )
                    .then((response) => {
                    // Here we're just checking if the response was successful or not before
                    // trying to do anything about it.
                    if (response.ok) {
                        // If it is successful, we want to update the task list.
                        return response.json();
                    }
                    })
                    .then((response) => {
                    this.fetched = true;
                    this.tasks = response;
                    });
                },
                updateTask(task) {
                //  : Use fetch() to send a PUT request to your API to update an task to be Done/not Done.
                console.log("update");

                fetch(
                    // The first parameter is a string that contains the full URL to your endpoint
                    `${process.env.VUE_APP_API_ORIGIN}/api/v1/tasks/${task._id}`,
                    // The second parameter is an object with options. You can include request
                    // headers here, options for credentials, which method, which mode, etc.
                    {
                    method: `PUT`,
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ Done: !task.Done }),
                    credentials: `include`,
                    }
                    // Note: The default for method is GET, so you don't need to include the
                    // method on any GET requests.
                ).then((response) => {
                    // Here we're just checking if the response was successful or not before
                    // trying to do anything about it.
                    if (response.ok) {
                    // If it is successful, we want to update the task list.
                    this.readTasks();
                    }
                });
                //  : Remember that the task's ID should be included in the path of the request, i.e. http://yourserverurl/api/v1/tasks/2r984hfiwufw948feoi
                //  : Remember to get the updated task list when it's done
                },
                // This method is given to you. Use it to see how to make fetch() requests.
                deleteTask(task) {
                fetch(
                    // The first parameter is a string that contains the full URL to your endpoint
                    `${process.env.VUE_APP_API_ORIGIN}/api/v1/tasks/${task._id}`,
                    // The second parameter is an object with options. You can include request
                    // headers here, options for credentials, which method, which mode, etc.
                    {
                    method: `DELETE`,
                    credentials: `include`,
                    }
                    // Note: The default for method is GET, so you don't need to include the
                    // method on any GET requests.
                ).then((response) => {
                    // Here we're just checking if the response was successful or not before
                    // trying to do anything about it.
                    if (response.ok) {
                    // If it is successful, we want to update the task list.
                    this.readTasks();
                    }
                });
                },
            },
            };
            </script>

            <style scoped>
            .form {
            padding: 0 1rem;
            }
            </style>
        ```