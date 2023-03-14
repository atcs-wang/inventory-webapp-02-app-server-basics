# Part 02: Web app server setup and basics

This tutorial follows after: [Part 01: Protyping our web pages](https://github.com/atcs-wang/inventory-webapp-01-static-prototypes)

> New technologies: [NodeJS](https://nodejs.org/en/), [Express](https://expressjs.com/)

MDN has a very good [starting tutorial for Express](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/development_environment
); it is more fleshed out in parts, but could be a good additional reference.

The second layer of a web app is the **web application server**, or just "web server" or "app server", for short. In this tutorial, we will create a simple web server that serves your static prototype pages. 

> In later tutorials, this server will be greatly improved and expanded in its capabilities, but it is essential to understand the basics first.

## (2.1) Setting up a NodeJS / Express project

> This is adapted from [expressjs.com's installation instructions](https://expressjs.com/en/starter/installing.html).

If you haven't before, you will need to install NodeJS on your machine. Download an installer for the latest Long Term Support (LTS) version from the [NodeJS website](https://nodejs.org/en/), and run it.

Once installed, you can confirm your version of Node and the Node Package Manager (npm) with these terminal commands:
```
> node -v
> npm -v
```

Create a folder for a new project, and open it in VSCode (or whatever editor you are using). 

> Although you could start with your existing project containing your prototypes, it's better to start clean and then transfer relevant files.

Open a terminal (which should be navigated to your project's root directory by default) and run the `npm init` command to create a  `package.json` file for your application. 

```
> npm init
```

This command prompts you for a number of things, such as the name and version of your application. For now, you can simply hit RETURN to accept the defaults for most of them, *except for this one*:
```
entry point: (index.js)
```

Instead, enter `app.js`. (You can do another filename or change this later if you want)

Once the prompts are finished, note the contents of the `package.json` file.

> **What is the `package.json` file for?**
> `package.json` functions a bit like a settings/configuration file for your whole project. Its primary function is keeping track of the package dependencies of your project, and is important going forward.

Now we can install the `express` package with `npm install`, like this:
```
> npm install express
```

It might take a moment to complete, but afterwards you should notice a few changes.

1. There is now a new subdirectory called `node_modules`, which contains a large number of other subdirectories, each of which correspond to an installed package. Although we only told `npm` to install `express`, it also installed all of `express`'s dependencies. (As of the time of writing this, installing `express` results in 50 packages total being installed). 

2. `package.json` has a new section called `dependencies`, which lists express and its (minimum) version. 
It might look something like this:
```json
"dependencies": {
    "express": "^4.17.2"
}
```
3. There is also a new file called `package-lock.json`. This contain a detailed list of every single installed packages and their exact versions.

Any time you install or update a package with `npm`, the files go into `node_modules`, and both `package.json` and `package-lock.json` are updated. The Node Package Manager (NPM) ensures that all your package dependencies are organized and tracked, and the versions are maintained nicely.

The biggest benefit of the Node Package Manager becomes clear when trying to share your project - say, as a git repository. Because all those installed packages in `node_modules` can be very big, we prefer not to track them or upload them to Github. 

Create a `.gitignore` file and add this line:

```
node_modules
```

Now, the packages themselves will not be tracked by git, and not uploaded to Github. *However, both `package.json` and `package-lock.json` are tracked.* So, if anyone clones your repository, they will *not* download a copy of all your package dependencies - but they *will* have the list of every dependency and their versions! 

Here's the fun part: you can (re-)install every dependency listed in `package.json` and `package-lock.json` with a single command:

```
npm install
```

This example's repository is the same - if you clone it, simply `npm install` to get dependencies set up before running.

> Strictly speaking, the `package-lock.json` is only necessary if exact package versioning is important. If only the `package.json` is available, `npm` may install any acceptable above-minimum version of direct dependencies, and will figure out the indirect dependencies.

## (2.2) A "Hello world" example server

> This is adapted from [expressjs.com's "Hello World" instructions](https://expressjs.com/en/starter/hello-world.html), which includes a similar example which is live-editable in-browser!

Create a file called `app.js`. This will be our main point of entry for the web application server. 

Add the following code:

```js
//set up the server
const express = require( "express" );
const app = express();
const port = 3000;

// define a route for the default home page
app.get( "/", ( req, res ) => {
    res.send( "<h1>Hello world!</h1>" );
} );

// start the server
app.listen( port, () => {
    console.log(`App server listening on ${ port }. (Go to http://localhost:${ port })` );
} );
```

Then, run it from the terminal with: 
```
> node app.js
```

Open your browser and go to `http://localhost:3000`. You should see the "Hello world!" message.

> You can type `Ctrl-C` in the terminal to stop the server. 

### (2.2.1) What's going on with "Hello World"?

> Before continuing, you should have a basic understanding of HTTP and its associated vocabulary: 
> 1. A **client** web browser visiting a website sends an **HTTP request** to the site's web **server**, which is hosted at a certain **domain** (like `www.google.com` or `www.youtube.com`). 
> 2. In addition to a **domain**, HTTP requests are also made to a particular **endpoint** of the server, which is a **path** (like `/search` or `/article`) and a specific request **method** (like `GET` or `POST`). 
>    - Most of the request info is encoded in the **URL** of the website page: for example, the URL** of `atcs-wang.github.io/assignments.html` shows the domain is `atcs-wang.github.io` and the path is `/assignments.html`. (The default method is `GET`.)
> 3. The web server should be constantly listening for requests from clients, then send back an appropriate **HTTP response** with a **status code** (like `200 OK`, or `404 NOT FOUND`) and a **body** with the requested data (like HTML pages).
>
> If you need a refresher on how this process and vocabulary, watch this short video: https://www.youtube.com/watch?v=guvsH5OFizE

Even though this is basically the simplest possible express web server you can write, there's quite a lot going on! Let's break down a few key parts.

- The first section sets up the server: line 1 imports the "express" module using `require()`, which is used to create a server (`app`). A `port` number is also defined. During development/testing, the port can be any number from 1024 to 65353; in deployed web servers, the standard port for HTTP is 80. However, 3000 is a typical development port for express servers, so we'll use that one for now. 

- The second section performs the "routing": configuring the server to handle certain HTTP request **endpoints** (aka **path** and **method**). The general pattern for defining a "route" looks like this:
    ```js
    app.METHOD(PATH, HANDLER)
    ```
  Here, `app.get('/', ... ` tells the app to listen for requests to the `/` **path** (aka the **root** path) with the `GET` **method**. (This is simply a request for the homepage.) The handler function looks like: 
  ```js
  (req, res) => {    
      res.send( "<h1>Hello world!</h1>" );
  }`
  ```
  This sends a response (via the object `res`) with a bit of HTML as the **body**. (The **status code** is automatically set to `200 OK`)
  
  > NOTE: We'll get into routing more deeply soon; if you want to dive in early, ExpressJS has a [more detailed primer](https://expressjs.com/en/guide/routing.html).

- Finally, the last section tell the server to start running; this makes the server "loop" repeatedly, listening for HTTP requests on the given `port` number. It prints a message to the console confirming that it has started.  (It doesn't stop until you manually stop it with `Ctrl-C` in the terminal.)
  > NOTE: Making the message string with backticks (\`) instead of typical single or double quotes lets us interpolate the value of `port` into the string with ``` `${port }` ```. These kinds of strings are called "Template literals", and are very convenient for constructing strings with variable values.

- When we run `node app.js`, the server is merely running on your local computer or network - which is not configured to receive requests from the wider internet. (Later, we'll get a cloud server with a **domain** for that). That's fine for now - browsers use the special pseudo-domain `localhost:PORT` to send HTTP requests to local servers running on certain ports.


> #### A quick side note about `express-generator`
>After the "Hello World" app, many `express` tutorials (including the [MDN one](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/development_environment) linked above) introduce a tool called `express-generator`, which sets up a 'skeleton' project and provides a lot of "boilerplate" code (that is, code that many projects start with). This can help jump start the building of a project, but can be bit overwhelming for a first-time learner.
>
>We won't use `express-generator` in this tutorial; rather, we'll build up our app's code and structure organically, understanding each new part as it becomes relevant and necessary. Eventually, our final structure will be very similar to `express-generator` produces, at which point you might better appreciate the tool for future projects.
>
> If you're curious enough to dig in now, Express has some [terse starting instructions for express-generator](https://expressjs.com/en/starter/generator.html), and MDN's has a [second part to their express tutorial](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/skeleton_website) that explains the output of `express-generator` nicely.

## (2.3) Making a simple app server for our prototypes 

Let's update the "Hello World!" server to actually serve our prototype pages. 

### (2.3.1) Adding and exploring more routes

The existing route handles the "/" path, which is the homepage.Let's start by adding two more routes:

```js
// define a route for the default home page
app.get( "/", ( req, res ) => {
    res.send( "<h1>Hello world!</h1>" );
} );

// define a route for the assignment list page
app.get( "/assignments", ( req, res ) => {
    res.send( "<h1>This is the assignments list page.</h1>" );
} );

// define a route for the assignment detail page
app.get( "/assignments/detail", ( req, res ) => {
    res.send( "<h1>This is the assignment detail page.</h1>" );
} );
```

If you haven't yet, stop your old server (`Ctrl-C` in the terminal) and run it again (`node app.js`). 

> If you forget to stop your old server, you'll get an error like `Error: listen EADDRINUSE: address already in use :::3000`. 

In your browser, go to `localhost:3000/assignments` and `localhost:3000/assignments/detail` and see the corresponding messages. 

Then, try a path that wasn't defined in the routing, like `locahost:3000/notworking`. You should see a message that says `Cannot GET /notworking`. Express automatically handles any requests to endpoints we didn't set up routes for by sending this simple message.

> If you use Chrome or Firefox, you can open your browser's Developer Tools and see the network activity. (right-click on your browser, choose **Inspect** and open the **Network** tab). This helps you see details about HTTP requests and responses, such as status codes. After opening, try hitting both valid and invalid routes again. The valid ones should have a status code of `200 OK` , and the invalid ones a `404 NOT FOUND`.

### (2.3.2) Sending files as HTTP response

Right now, of course, the server only sends snippets of HTML. We want to send our prototype HTML pages, which are the files `index.html`, `assignments.html`, and `detail.html`. 

It's finally time to add our prototype files into our new project. Place them into a new subdirectory (aka folder) called `views`.

At this point, your file structure should look something like this:
```
|-node_modules
| |-...
|-views
| |-assignments.html
| |-detail.html
| |-index.html
|-.gitignore
|-app.js
|-package-lock.json
|-package.json
```

Then, update the routing code to this:

```js
// define a route for the default home page
app.get( "/", ( req, res ) => {
    res.sendFile( __dirname + "/views/index.html" );
} );

// define a route for the assignment list page
app.get( "/assignments", ( req, res ) => {
    res.sendFile( __dirname + "/views/assignments.html" );
} );

// define a route for the assignment detail page
app.get( "/assignments/detail", ( req, res ) => {
    res.sendFile( __dirname + "/views/detail.html" );
} );
```

The `res.sendFile` method does exactly what you think it does: it sends a file, given by an absolute path, in the HTTP response. 

> To give the absolute path of the file, we use a special variable that exists in NodeJS called `__dirname`, which contains the path of the root directory of the program. Appended to that is the *relative* path of each file in the project: the containing `views` directory, then the file name. Keeping track of where everything is located is tricky - which is why organized file structure is important!

Restart the server, and try visiting your `localhost` pages in the browser again. Ta-da! You should see your prototype pages appear instead of the little fragments.

>  You might notice that any images are no longer appearing properly. Same goes for any external resources like stylesheets or scripts - they won't load with your page yet. **Don't Panic!**  We'll address this momentarily in section 2.4 below.

### (2.3.3) Fixing page links

An issue now is that the navbar, nav menus, and the link-buttons that navigate between the pages themselves don't work correctly. All hyperlinks now need to use the same routes (URL paths) we defined in the server.

First, let's fix the navbar and mobile nav menu on each page. Find these links (which show up twice in the `<header>` - once for the desktop nav, and once for the mobile nav):

```html
<a href="index.html">...
<a href="assignments.html">...
```
and change the `href` to:
```html
<a href="/">...
<a href="/assignments">...
```

There's also the `Get Started!` button in `index.html`. Find this part:
```html
<a class="btn" href="assignments.html">...
```
and change the `href` so it says
```html
<a class="btn" href="/assignments">...
```

Finally, there's the two "Info/Edit" buttons in `assignments.html`. Find both instances of:
```html
<a class="btn-small waves-effect waves-light" href="detail.html">
```
and change the `href` so it says
```html
<a class="btn-small waves-effect waves-light" href="/assignments/detail">
```

Restart your server, and navigate to your homepage. Test all your links to make sure they work now!
### (2.3.4) Adding logging
As we continue to build out and test our server, we'd like to "log" records of all HTTP requests made to the server. This can be helpful during development and debugging, and even during production for security and data integrity purposes.

One way to do this is to add a `console.log` statement in each of our routes, like this:

```js
// define a route for the default home page
app.get( "/", ( req, res ) => {
    console.log("GET /");
    res.sendFile( __dirname + "/views/index.html" );
} );

// define a route for the assignment list page
app.get( "/assignments", ( req, res ) => {
    console.log("GET /assignments");
    res.sendFile( __dirname + "/views/assignments.html" );
} );

// define a route for the assignment detail page
app.get( "/assignments/detail", ( req, res ) => {
    console.log("GET /assignments/detail");
    res.sendFile( __dirname + "/views/detail.html" );
} );
```

(Re)start the server again, and visit each page. Notice that the messages print out to the terminal as each request comes in.

So far, we've only used one of the two parameters in the route handler function: the `res` object, which represents the HTTP response that the handler is supposed to prepare for sending back. We've already used the `send` and `sendFile` methods of it. 

The other parameter `req` is an object that represents the HTTP request, and all the information that is sent with it. For example, the properties `url` and `method` get the request's path and method type, respectively. Even though that particular information can be derived from the route definition, the `req` object can be quite useful.

We can, then, replace each of the 3 different `console.log` statements with the same one:

```js
    console.log(`${req.method} ${req.url}`);
```

#### (2.3.4.1)  Logging with "middleware"
It seems a bit silly, now, that we have the same `console.log` statement in each of our routes. We don't want to code the same thing in every route if we can write it just once.

This brings us to a powerful concept/tool in the Express framework: "**middleware**". Middleware are functions that can process requests *before* being finally handled. We can specify that middleware ought to be applied to *all* incoming requests with the `app.use` method. 

Update the routing code to this:
```js
// define middleware that logs all incoming requests
app.use((req, res, next) => {
    console.log(`${req.method} ${req.url}`);
    next();
} );

// define a route for the default home page
app.get( "/", ( req, res ) => {
    res.sendFile( __dirname + "/views/index.html" );
} );

// define a route for the assignment list page
app.get( "/assignments", ( req, res ) => {
    res.sendFile( __dirname + "/views/assignments.html" );
} );

// define a route for the assignment detail page
app.get( "/assignments/detail", ( req, res ) => {
    res.sendFile( __dirname + "/views/detail.html" );
} );
```

We have moved our `console.log` statements into a single middleware function, registered with `app.use()`. Like the route handlers, middleware functions have the parameters `req` and `res`, but also `next`. `next` is a function, which is called to pass the control on from the middleware to the next thing. 

> **Notice**: the middleware is registered ***above*** the route handlers. *This is crucial*, as Express will apply the middleware and handlers *in the order defined*. 
> 
> If the middleware is moved below any of the route handlers, those route handlers will run first, handling the request and sending the response -  *without* ever running the middleware. Try moving the middleware after or between the route handlers and see what happens!

Now the middleware not only logs requests for our 3 routes, but it even logs attempted requests for routes we *don't* handle. Try running and testing the server now by navigating to each page, but also trying to access pages (unhandled routes) that don't exist (e.g. `localhost:3000/notarealroute`).

> This is actually a good thing - we want to have records of any and all requests made to our server - valid or invalid.

#### (2.3.4.2)  The `morgan` module

Defining our own middleware can be very useful, but a lot of very useful middleware can be imported from modules. Our bare-bones logger middleware works fine, but let's replace it with a much nicer one from the `morgan` module.

This module needs to be installed first:
```
npm install morgan
```

Somewhere in the first section of the `app.js` file, you should import the `morgan` module with:

```js
const logger = require("morgan");
```

Finally, replace our custom logging middleware with this:

```js
app.use(logger("dev"));
```

Try running and testing the server once again. You'll notice that, in addition to logging the request method and path, information about the *response* is also logged, such as the (color coded!) status code and the response time.

> Side note: You might notice that in addition to `200` and `404` status codes being logged, there are a lot of `304` status codes being sent, which means `NOT MODIFIED`. When the server notices the same client is sending the same request repeatedly, and it also knows the response content hasn't changed, it will send this status code instead of `200 OK` and not bother re-sending the content. This is because the client's browser is likely to have "cached" (temporarily saved) the last response. If the server is mistaken, the client can send a special request insisting that the content is re-sent. This generally saves time and computational/network resources.

## (2.4) Serving Static Files

> ExpressJS.com has a brief explanation of static file serving: 
[https://expressjs.com/en/starter/static-files.html](https://expressjs.com/en/starter/static-files.html)

HTML pages often reference other "static" files (such as external CSS, external JS, fonts, images, etc.) as required resources for the webpage. While rendering the page, the web browser will make additional HTTP requests to the server for those files.

Since such static files are often numerous (and typically referenced by their actual file name/path, rather than a custom URL path), we'd rather not set up custom routes for each file. 

Thankfully, it is easy to configure Express to serve all static files in a given folder via another middleware - the so-called "static" middleware, which is built into the express module:

1. Create a folder in the root of your project called `public`.

2. Add this line to `app.js` *below* the `app.use(logger("dev"))` line:
    ```js
    // define middleware that serves static resources in the public directory
    app.use(express.static(__dirname + '/public'));
    ```


Now when the app receives HTTP requests with URL paths that match relative filepaths within the `public` folder, the middleware will simply respond with the resource.  

What does that mean for us? Practically, it means we can now use images, external stylesheets, and external scripts!  
### (2.4.1) Using images

Images are perhaps the most common form of static files. We used one in our prototype's `index.html`, though it's been 'broken' here so far. Let's fix that!

1. First, create a new folder in `public` called `images`.
2. Then, add to them any images needed, such as `clutter-clipart.jpg`

3. Verify that the server is serving it by visiting `http://localhost:3000/images/clutter-clipart.jpg`. 
    > The logs in the Terminal should show the GET request for `images/clutter-clipart.jpg`.
    
3. Check `index.html`'s' image tag reads like this:
    ```html
        <img src="/images/clutter-clipart.jpg">
    ```
4. Verify the image appears on the homepage by visting `http://localhost:3000/`. 
    > You might notice that the request and response for both the page AND the image are logged in the Terminal.


### (2.4.2) Using external stylesheets (CSS) and external scripts (JS)

Now that we can also reference static files served by our own server, we can also utilize external spreadsheets and scripts.

We already have some internal stylesheets and scripts in our prototype which we will now convert to external stylesheets and scripts. 

> This might seem like a trivial bit of code to externalize, but later on we will be applying security practices that forbid internal spreadsheets and scripts. It's best to move things now.

> You may already have been using external stylesheets or scripts, in which case you can simply follow the steps below that pertain to where those scripts ought to be placed. 

#### (2.4.2.1) External Stylesheet (assignmentsStyle.css)
Two of the views - `assignments.html` and `detail.html` - currently have internal stylesheets that are identical:
```html
<style>
    form {
        border: solid 2px lightgrey;
        padding: 10px;
        border-radius: 5px;
    }

    form:focus-within {
        border-color: orange;
    }

    tr:hover {
        background-color: #F0F0F0;
    }
</style>

```
This CSS would be better off as a single external stylesheet. Do the following:

- inside of the `public` folder, create a new folder called `stylesheets`
- inside of *that* new folder create a new file called `assignmentsStyle.css`
- move the contents of the `<style>` tag into `assignmentsStyle.css`
- replace the style tags in both `assignments.html` and `detail.html` with:

    ```html
    <link rel="stylesheet" href="/stylesheets/assignmentsStyle.css">
    ```

> Make sure this link is placed ***below*** the other stylesheet links to the Materialize CSS; this makes sure that the cascade prioritizes our external CSS over the default Materialize styles.
>
> Also, don't forget the `/` at the beginning of the `href` value- this makes the path *absolute*, not relative to the current page's URL.

After restarting your server, you can verify that the server is serving `assignmentsStyle.css` by directly visiting `http://localhost:3000/stylesheets/assignmentsStyle.css` in your browser. The logs in the Terminal should show the GET request for the stylesheet.

#### (2.4.2.2) External Scripts (materializeAutoInit.js)

All of the views - `assignments.html` and `detail.html` - also have a small internal script at the bottom of the `<body>`:
```html
<script> M.AutoInit();</script>
```

This, too, is better off as an external script. Do the following

- inside of the `public` folder, create a new folder called `scripts`
- inside of *that* new folder create a new file called `materializeAutoInit.js`
- move the contents of the `<script>` tag into `materializeAutoInit.js` (literally, just `M.AutoInit();` )
- replace the script tags in all 3 html files with:

    ```html
    <script src="/scripts/materializeAutoInit.js"></script>
    ```

After restarting your server, you can verify that the server is serving `materializeAutoInit.js` by directly visiting `http://localhost:3000/scripts/materializeAutoInit.js` in your browser. The logs in the Terminal should show the GET request for the script.

### (2.4.3) Testing static resources 

If you haven't already, check that each page properly displays with images, the custom styles (that is, a border on the form and table highlighting on mouse hover) and working dynamic elements (including the hidden mobile nav menu and modals can be toggled). 

In each case, the logs in the Terminal will show GET requests both for that page, *and* for the external static resources that the page requires.

>If the static resources can be loaded on the browser, but the server logs don't display the GET requests for it, double check that the line in `app.js` setting up the static file middleware comes *after* the line setting up the logger middleware. If it comes *before*, the handling of static files will only be logged if NOT found. 
>
>Sometimes, you may actually prefer to NOT see server logs for every successful static resource request - in which case you could move the logging middleware after the static resource middleware, so it only logs non-static resource requests.

> You can also see all the referenced external files of an HTML page via the browser's Developer Tools - right-click Inspect, and click the Sources tab. The Network tab will show you the status of the actual HTTP requests for each of these resources too, including status code. This is a great way to debug if a resource doesn't appear to be loading.

### (2.4.4) OPTIONAL: Serve Materialize CSS/JS locally instead of from a CDN

Each of our views still reference external stylesheets and external scripts which contain Materialize's CSS and JS:

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">
...
<script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
```

These are (currently) being served not from our own server but from another domain, `cdnjs.cloudflare.com`, which is a publicly accessible Content Deliver Network (CDN). 

However, if you prefered to have these resources served from our own server instead, you can download the files from [Materialize's website](https://materializecss.com/getting-started.html). 

Place `materialize.min.css` in  `public/stylesheets`, and `materialize.min.js` in `public/javascripts` (you may need to create the subfolder `javascripts`).

Finally, change the link/script tag references to local paths instead:

```html
<link rel="stylesheet" href="stylesheets/materialize.min.css">
...
<script src="javascripts/materialize.min.js"></script>
```

> There are numerous benefits (better performance, increased reliability, cost savings, and resilience against cyber attacks.) associated with using a CDN instead. Hence, this is an optional step, and not represented in the code base. 



### (2.4.5) Browser tab icon - favicon.ico

If you "Hard Refresh" your browser (Ctrl-Shift-R on Windows, Cmd-Shift-R Mac), you'll notice another GET request made to your server - `GET /favicon.ico`. 

This is because the first time a browser visits a website, it requests an icon file (presumably called "**`favicon.ico`**") that can be used in the Tab. The browser will "cache" that icon for a while so it doesn't have to request it every time, but "Hard Refresh" forces the browser to clear its cache and ask again. So, if we add a `favicon.ico` file into the `public` folder (and possibly Hard Refresh the browser), it will be used as the browser's tab icon for all the pages of our webapp! 

You can easily turn any image, text, or emoji into a favicon at [favicon.io](https://favicon.io/). For this example, we used the "Card File Box" emoji.

> Optionally, if you prefer to use an icon file NOT called `favicon.ico` and/or store it elsewhere (e.g. a subfolder of `public`), you can add this to each page's head section to specify the location of the favicon. 
>```html
><link rel="shortcut icon" type="image/jpg" href="Favicon_Image_Location"/>
>```
>This technique also allows you to use different tab icons for different pages, if you so wish. 

## (2.5) Conclusion:
You've set up a NodeJS project, and you've implemented a simple Express web server, learning about the basics of HTTP requests/responses and defining routes.

The structure of your project should look something like this:
```
|-node_modules
| |-...
|-public
| |-images
| | |-clutter-clipart.jpg
| |-scripts
| | |-materializeAutoInit.js
| |-stylesheets
| | |-assignmentsStyle.css
| |-favicon.ico
|-views
| |-assignments.html
| |-detail.html
| |-index.html
|-.gitignore
|-app.js
|-package-lock.json
|-package.json
``` 

Of course, the server so far is not very different or better than using a static file server (like [VSCode's Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) or [Python's http.server module](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/set_up_a_local_testing_server#running_a_simple_local_http_server)). The pages themselves, of course, are also still just static prototypes.

Next, we'll discuss databases and how the web server can be use them to put real data into the pages.

Next up: [Part 03: Database Layer: Database connection and first table set-up](https://github.com/atcs-wang/inventory-webapp-03-db-connection-setup/)
