![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Express | File Upload

## Learning Goals

After this learning unit, you will be able to:

- Understand the importance of file upload in web applications
- Create a file upload process in your web applications
- Explain why we need to implement HTML multipart
- Create your Express application using Mongoose and implementing file uploading

## Introduction

Uploading files to the internet is essential in web development.

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload.gif =400x)


NodeJS or Express don't support file uploading easily, there is a lot of boiler plate code needed to receive and store the file. Nonetheless, there are packages to help us!

In this learning unit we will see how to create a simple Express application using a Mongo database that will upload pictures and render them.


The project we are going to create will be something similar to instagram. Users will be able to:

- Upload pictures and give them a name
- See a list of all uploaded pictures with their names

Ready?

## Express File Upload Project

### Create Project

Let's begin by creating a new npm project. You will need to install **express-generator** in your system. If you haven't done it before, use `npm install express-generator -g`.

Create a project using `ejs` for views, `sass` and name your project `lab-express-file-upload`. Install your dependencies and start the server.

```bash
$ express -v ejs -c sass --git lab-express-file-upload
$ npm install
$ npm start
```

### Remove Users Routes

The express generator created a filesystem for you with some key files. You might notice you have a new folder `routes` and inside it, you'll find a `user.js` file. As we won't be implementing Users for now, you can delete this file and also remove the `require('./routes/user.js')` from `app.js`.

### Set up index route

We will implement the whole solution with just one route: `/routes/index.js`

```javascript
/* GET home page. */
router.get('/', function(req, res, next) {
    res.render('index')
});
```

Let's begin by create a simple HTML index file with a very simple form where the user will upload the pictures:

```htmlmixed
<section>
  <h2>Upload your photo</h2>
  <form action="/upload" method="post" enctype="multipart/form-data">
    <input type="text" name="name">
    <input type="file" name="file">
    <input type="submit" value="Save">
  </form>
</section>
```

Let's take a look at the code above. We send the post to the `/upload` action and assign a type and name to every input. This will be necessary to access them when creating and saving Pictures in our database.

The code adds a very simple form, but one of the most important parts is the `enctype` property. Let's take a few minutes to explain this.

### The multipart attribute

When we are working with HTML, by default you can send plain text over the requests to our server. What happens if we want to send an image? An image is not plain text, so we have to do something extra to be able to receive this information.

The `enctype="multipart/form-data"` allows the form to be able to upload files to the server. We will need to install a middleware, multer, in order to send this kind of data to the server.

### Install and require multer

We will also need to install [multer](https://www.npmjs.com/package/multer):

![Multer picture](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_4414d553b9a172503c5595c85adda3bd.png)

```
$ npm install --save  multer
```

Multer is a middleware for handling **only** multipart/form-data and it is used for uploading files.

Multer adds a file or files object to the request object, so it makes easier to work with multipart bodies. The body object contains the values of the text fields of the form, the file or files object contains the files uploaded via the form.

As we will use multer when posting the data in the `index` view, require the package in the `index.js` file under the `routes` path.

```javascript
var multer  = require('multer');
```

And now let's configure our route:

```javascript
const Picture = require('../models/pictures');
```

Until now, our application renders the index view. The user could insert the picture but it doesn't exists yet in our application. Let's create the model to change this.

### Model Setup

#### Install and connect to mongoose

To be able to connect your application to a database and create a model for Pictures to save them, we need to install mongoose:

```bash
$ npm install --save mongoose
```
Now add this line to your app.js to connect the database to your application. The database and collection in MongoDB will be created when we insert the first picture. But this will come later.

```javascript
mongoose.connect('mongodb://localhost/lab-express-file-upload');
```

#### Create your Picture model

Create a `pictures.js` file under a new `models` folder that will hold the `Picture` object code:

```bash
$ mkdir models
$ cd models
$ touch pictures.js
```

In the `models/pictures.js` file, require mongoose and create a new Schema with three parameters: name, pic_path and pic_name. Also, add two [timestamps](http://whatis.techtarget.com/definition/timestamp) to save the moment when the file was created or updated. Then, export your model:

```javascript
const mongoose = require("mongoose");
const Schema   = mongoose.Schema;

const pictureSchema = new Schema({
  name: String,
  pic_path: String,
  pic_name: String
}, {
  timestamps: { createdAt: "created_at", updatedAt: "updated_at" }
});

var Picture = mongoose.model("Picture", pictureSchema);
module.exports = Picture;
```

### Create the uploads folder

We need to store our files in an specific folder, so create the path inside your project folder:

```
$ mkdir public/uploads
```

### The Upload

To upload a new picture, we will use [multer](https://www.npmjs.com/package/multer) and set the destination path.

In the post, we are creating a new Picture object with the attributes from the `req` object -this will insert a new element in the database while multer takes care of uploading the file to the proper path.

When the picture is inserted, the app will redirect the user to the index route.


```javascript
// Route to upload from project base path
var upload = multer({ dest: './public/uploads/' });

router
.post('/upload', upload.single('file'), function(req, res){

  pic = new Picture({
    name: req.body.name,
    pic_path: `/uploads/${req.file.filename}`,
    pic_name: req.file.originalname
  });

  pic.save((err) => {
      res.redirect('/');
  });
});

```

A brief explanation to understand the info we are inserting in the DDBB:

:::info
## Accesing the data

### The Request object

The req object represents the HTTP request. It contains the data organized in properties: query string, parameters, body, HTTP headers, and so on. By convention, the object is always referred to as `req` (and the HTTP response is `res`) but its actual name is determined by the parameters to the callback function in which you’re working.

#### Some properties of the `req` object

###### `req.body`

Contains key-value pairs of data submitted in the request body. Middlewares as `body-parser` and `multer` populate it.

###### `req.params`

This property is an object containing properties mapped to the named route “parameters”. For example, if you have the route /user/:name, then the “name” property is available as req.params.name. This object defaults to {}.

###### `req.query`

This property is an object containing a property for each query string parameter in the route. If there is no query string, it is the empty object, {}.

###### `req.file`

Multer adds the `body` object to the `req`, but also, it adds a `file` object with the multipart data in it.

:bulb: Take some time and use your debugging tools to inspect these elements.
:::


## Testing and debugging

You can test the uploading of files by opening the Network tab in the Google Chrome DevTools.


![Network Tab](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_cab4a941423af22d7b15a864681fd2c7.png)

In the picture above, the last method shows the upload of a text file. You get to know where it was sent, the status of the request, type of document and even the time it took to upload it.

Use the Network tab to verify everything works properly. You will be able to see your uploaded files if you open the browser and from your project root folder, go to `public/uploads`



## Show Pictures

### Edit the get method to index

To show the pictures we will need to edit the code we created before. Let's begin by edit our index route

```javascript
/* GET home page. */
router.get('/', function(req, res, next) {
  Picture.find((err, pictures) => {
    res.render('index', {pictures})
  })
});
```

In the code above we inserted a find method to retrieve all the pictures. Then, the application will render the index sending the object `pictures` as a param. This object will be used by `index.ejs`. Ready to see how this is done!?

### Edit the index view

In the `index.ejs` we will embed JavaScript to go over the `pictures` object and render its properties one by one:

```htmlmixed
<section>
  <% pictures.forEach(function(picture) { %>
    <h3><%= picture.name %></h3>
    <img src="<%= picture.pic_path %>" alt="<%= picture.pic_name %>" />
  <% })%>
</section>
```

## Summary

In this learning unit we have learnt how to implement a file uploading in our express application. Now, you understand the importance of the enctype attribute in order to let multer handle the file. You also know how to use the Network tab of the Google Chrome DevTools to understand what's happening on the web

## Extra Resources

* [HTTP Methods](http://www.restapitutorial.com/lessons/httpmethods.html) -Definitions for different http methods
* [Upload to S3](https://devcenter.heroku.com/articles/s3-upload-node) - Some help info about uploading files to Amazon S3
* [Multipart form-data encoding](http://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean)
* [Multer library](https://github.com/expressjs/multer) - Handling multipart form data within express
