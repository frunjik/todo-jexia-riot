# Jexia-Riot-Todo Tutorial
=======

This tutorial shows how easy it is to create a web based Todo application using the Jexia and the [riot.js](http://riotjs.com/) and [jQuery](http://jquery.com/) libraries.

Jexia decreases your application development for creating great web applications or mobile apps. In this tutorial Jexia is used as backend service to store and retrieve todo items. All interaction with Jexia is done using RESTful API's that are generated for you automatically.

Riot.js is a JavaScript library that lets you write your JavaScript code as separate components and helps you to write code that has a clean separation of concerns and is reusable.

It is assumed that you have a basic knowledge of JavaScript and HTML. To complete this tuturial follow the steps below, or go directly to the code on github (https://github.com/frunjik/todo-jexia-riot/blob/master/index.html).


Since only Client side javascript is needed for this tutorial no local setup is needed! All that is needed is access to Jexia in order to create the backend of your Todo application.


The tutorial shows two JavaScript objects:
1. The Todo component manages the list of Todo items
2. The Jexia component manages the communication with Jexia


The riot Todo component fires events for when creating, deleting or updating an item.

The Jexia component has methods for each of the DataSet CRUD calls:

 - To create a new item, send a POST request
 - To read an existing item, send a GET request
 - To update an existing item, send a PUT request
 - To delete an existing item, send a DELETE request




## Step 1

Login to Jexia with your username and password.

Create a DataApp with the name: ToDoApp

Create the DataSet with name: todo

All created DataSet have a root key when created. Working with the default root key is not recommended.So to work with the DataSet for our application we create a key. In order to create a key:

* Choose 'Manage your Keys' from the menu.
* Create a key named 'TodoKey' for this DataApp and DataSet.
* Give your key a good description. E.g. ' Test key for todo'.
Make sure the key has `read / write`  permissions (the default).

The created DataSet get default methods (RESTful) for each of the DataSet CRUD calls. So for our Todo application we make use of the following REST calls:

 - To create a new item, send a POST request
 - To read an existing item, send a GET request
 - To update an existing item, send a PUT request
 - To delete an existing item, send a DELETE request


## Step 2

Create a file index.html containing:

```html
<!doctype html>
<html>
    <head>
        <title>Jexia - riot - todo</title>
    </head>
    <body>
		 <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.0.0-alpha1/jquery.min.js"></script>
         <script src="https://cdnjs.cloudflare.com/ajax/libs/riot/2.3.1/riot+compiler.min.js"></script>
    </body>
</html>
```

At the end of the body we include the links to the JavaScript files we need. In our case jQuery and riot.


## Step 3

Now we create the basic riot todo component. This component takes care of displaying the list of todo items and has methods for adding, updating and removing items from the list:

```html
<script type="riot/tag">
            <todo>
                <div class="row">
                     <div class="col-12"><h3>{opts.title}</h3></div>
                </div>
                <div class="row">
                     <div class="col-12"><hr/></div>
                </div>
                <div class="row" each={items}>
                    <div class="col-1">
                        <label class={ completed: done }>
                            <input type="checkbox" checked={done} onclick={toggle}>
                        </label>
                    </div>
                    <div class="col-10">{title}</div>
                    <div class="col-1" align="right">
                        <button onclick="{remove}">X</button>
                    </div>
                </div>
                <form onsubmit={create}>
                    <div class="row">
                        <div class="col-6">
                            <input name="input" onkeyup={edit}>
                        </div>
                        <div class="col-6" align="right">
                            <button disabled={!text}>+</button>
                        </div>
                    </div>
                </form>
                this.items = opts.items;
                remove(e) {
                    this.trigger('remove', e.item);
                }
                edit(e) {
                    this.text = e.target.value;
                }
                create(e) {
                    if (this.text) {
                        this.trigger('create', {title: this.text});
                        this.text = this.input.value = '';
                    }
                    return false;
                }
                toggle(e) {
                    var item = e.item;
                    item.done = !item.done;
                    this.trigger('modify', item);
                    return true;
                }
            </todo>
        </script>
```

## Step 4
In order to use the RESTful api's created on our DataSet we create a new riot component. So we create a component named Jexia to access our DataSet. We do that in the following simple way:

```javascript
<script type="riot/tag">
            <jexia>
                this.url = opts.url;
                this.token = opts.token;
                get() {
                    this.ajax(this.url, 'get', 'got');
                }
                create(data) {
                    this.ajax(this.url, 'post', 'created', data);
                }
                modify(id, data) {
                    this.ajax(this.url + id, 'put', 'modified', data);
                }
                remove(id) {
                    this.ajax(this.url + id, 'delete', 'removed');
                }
                ajax(url, method, trigger, data) {
                    var self = this;
                    $.ajax({
                        url: url,
                        type: method,
                        data: data,
                        headers: {
                            'Authorization': 'Bearer ' + this.token
                        }
                    }).
                    done(function(data) {
                        $.each(data, function(index, item) {
                            if(item && item.done) {
                                item.done = (item.done === "true");
                            }
                        });
                        self.trigger(trigger, data);
                    });
                }
            </jexia>
        </script>
```

By using this component for the communication with Jexia where are data is stored we must make sure that the required access token is included in the header of every HTTPS request. Without this access token in the header of every REST call we get an error message that we are not allowed to access our data. The next step will show you how we get this access token.

Our riot Todo component fires events for when creating, deleting or updating an item.




## Step 5

To access our DataSet we need an access token. We get this when the application starts (when the page is loaded). We attach to jQuery's ready event to get things started:

```javascript
var appId = 'JEXIA-APP-ID',         // TODO: replace with your Jexia AppId
    dataApp = 'http://' + appId + '.app.jexia.com/',
    dataSet = dataApp + 'todo/',
    todo,
    jexia;
$(document).ready(function() {
    $.ajax({
        url: dataApp,
        type: 'post',
        data: {
            key: 'JEXIA-KEY',       // TODO: replace this with your key
            secret: 'JEXIA-SECRET'  // TODO: replace this text your secret
        }
    }).
    done(function(data) {
        boot(dataSet, data.token);
    });
});
```

Replace the `JEXIA-KEY` and `JEXIA-SECRET` tags in the code with the data you received in Step 1. You can find the key and secret on the Api Key Management page. By default the secret is hidden, press the show link to see it. The access token received by posting the key and secret, has a limited lifespan. The refreshing of the token is not part of this tutorial. If the token is expired a new token is acquired by refreshing the page.


## Step 6

The last piece that is needed is to knit all the components together. The code at bottom of the page, boots the application after receiving the access token. The starting of the application consists of mounting (creating) the riot components, and assigning the event handlers.

```javascript
<script type="text/javascript">
    function boot(url, token) {
        riot.compile(function() {
            todo = riot.mount('todo', {
            title: 'What do you jexia today?',
                items: []
            })[0];
            todo.on('create', function(data) {
                jexia.create(data);
            });
            todo.on('modify', function(data) {
                jexia.modify(data.id, data);
            });
            todo.on('remove', function(data) {
                jexia.remove(data.id);
            });

            jexia = riot.mount('jexia', {url: url, token: token})[0];
            jexia.on('got', function(data) {
                todo.items = data;
                todo.update();
            });
            jexia.on('created', function(data) {
                jexia.get();
            });
            jexia.on('removed', function(data) {
                jexia.get();
            });
            jexia.on('modified', function(data) {
                jexia.get();
            });
            jexia.get();
        });
    }
    var appId = 'JEXIA_APP_ID',                             // TODO: replace with your Todo APP_ID
        dataApp = 'http://' + appId + '.app.jexia.com/',
        dataSet = dataApp + 'todo/',
        todo,
        jexia;
    $(document).ready(function() {
        $.ajax({
            url: dataApp,
            type: 'post',
            data: {
                key: 'JEXIA_KEY',                           // TODO: replace with your Jexia key for your created DataSet
                secret: 'JEXIA_SECRET'                      // TODO: replace with the secret belonging to your created DataSet
            }
        }).
        done(function(data) {
            boot(dataSet, data.token);
        });
    });
</script>
```

## Step 7

To give he page a nicer look we include some minimal css stylesheets at the top of the page:

```html
<head>
	<title>Jexia - riot - todo</title>
    <link rel="stylesheet" href="todo.css">
    <link rel="stylesheet" href="minimal.css">
</head>
```


## Summary

The nice aspect of this example is that you can load it directly from your local filesystem into the browser, without the need for an extra (middleware) backend. You can drop the files of this example in a folder on you local file-system, load the index.html into your browser and start using it.

In this tutorial we created a fully function Todo application in no time. This simple example does no error checking and can be improved using Jexia's Real-Time communication API so that all users of our Todo application directly see changes on the Todo list. In this you can create a group Todo application a few extra lines. Check our SDK's or other examples to find out how you can benefit from using Jexia.


You can find the complete code for this tutorial [here](https://github.com/frunjik/todo-jexia-riot/blob/master/index.html).

The text of this tutorial is available under the terms of CC BY-SA 4.0. All tutorial code is MT licenced.


>Written with [StackEdit](https://stackedit.io/).
