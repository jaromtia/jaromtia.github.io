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

- NodeJS
- Google Oauth2
- Javascript
- JSON
- NPM (Node Package Manager)
- Vue.js
- Mongoose.js
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

3. 

Include at least 3 tips and/or warnings that highlight common errors or misunderstandings. These are similar to tips from regular lab write-ups that you create. Remember, don’t use first person.

## References: 

1. Google Identity - OAuth 2.0 https://developers.google.com/identity/protocols/oauth2/scopes, Google Identity, accessed December 2021.

2. Vue.js - List Rendering https://vuejs.org/v2/guide/list.html, Vue.js, accessed December 2021

3. 

3. Wireshark Foundation, “Wireshark: the world’s foremost protocol analyzer”
http://www.wireshark.org/, Wireshark Foundation, accessed Sept 2011.
4. Stack Overflow, “How do I convert a string into an integer in javascript”
http://stackoverflow.com/questions/1133770/how-do-i-convert-a-string-into-an-integer-in-javascript,
Stack Overflow, accessed Sept 2011.

## Documented Code: 

Include as an appendix your documented code. This should include comments that help explain variables, functions, etc. in context. They may include the same text as used in the Detailed
Documentation section, but in this case they will be inside of your code using the language commenting
features. Hopefully you have been doing this throughout the project, so it won’t be a lot of extra work.
It’s a key habit to get into for any coding.
