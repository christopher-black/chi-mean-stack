### MEAN stack from scratch
Exercise building everything up from scratch.

**In Terminal**

1. Fork the repository
2. `cd chi-mean-stack`
3. `npm init`
4. `npm install express body-parser angular mongoose bootstrap grunt grunt-contrib-copy grunt-contrib-uglify grunt-contrib-watch --save`

**In Atom**

5. Folder structure

> NOTE: `node_modules/` & `package.json` are auto generated

```
chi-mean/
├── client/
│   ├── scripts/
│   │   └── client.js
│   ├── styles/
│   │   └── style.css
│   └── views/
│       └── index.html
├── node_modules/
│   └── ...
├── server/
│    ├── modules/
│    ├── routes/
│    └── app.js
├── .gitignore
├── Gruntfile.js
└── package.json
```

> Commit our changes

6. Create our `Gruntfile.js`

```JavaScript
module.exports = function(grunt) {
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    uglify: {
      build: {
        src: 'client/scripts/client.js',
        dest: 'server/public/scripts/client.min.js'
      }
    },
    copy: {
      html: {
        expand: true,
        cwd: 'client/views/', // Current working directory
        src: ['index.html'], // List of files to copy
        dest: 'server/public/views/' // Destination
      },
      css: {
        expand: true,
        cwd: 'client/css/', // Current working directory
        src: ['style.css'], // List of files to copy
        dest: 'server/public/views/' // Destination
      },
      angular: {
        expand: true,
        cwd: 'node_modules/angular/', // Current working directory
        src: ['angular.*'], // List of files to copy
        dest: 'server/public/vendors/angular/' // Destination
      },
      bootstrap: {
        expand: true,
        cwd: 'node_modules/bootstrap/dist/', // Current working directory
        src: ['css/*.css','fonts/*.*'], // List of files to copy
        dest: 'server/public/vendors/bootstrap/' // Destination
      }
    },
    watch: {
      files: ['client/**/*.*'], // All files in the client folder
      tasks: ['uglify', 'copy']
    }
  });
  // LOAD PLUGIN: Bring the plugin into the project
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-copy');
  grunt.loadNpmTasks('grunt-contrib-watch')

  // REGISTER TASK: Actually use the plugin
  grunt.registerTask('default', ['uglify','copy','watch']);
};

```

> Commit our changes

7. Build our server in `app.js`

```JavaScript
var express = require('express');
var app = express();
var path = require('path');

app.set('port', (process.env.PORT || 5000));

app.use(express.static('/server/public'));

app.get('/', function(req,res) {
  res.sendFile(path.resolve('server/public/views/index.html'));
});

app.listen(app.get('port'), function() {
  console.log('Localhost running on port', app.get('port'));
});
```

> Commit our changes

8. Create `modules/db.js` to store our database connection code.

**db.js**

```JavaScript
var mongoose = require('mongoose');
var mongoURI = 'mongodb://localhost:27017/chimessages';
var MongoDB = mongoose.connect(mongoURI).connection;

MongoDB.on('error', function(error){
  console.log('Connection error', error);
});

MongoDB.once('open', function(){
  console.log('Connected to Mongo.');
});

module.exports = MongoDB;
```

Finally, add `var db = require('./modules/db.js');` to your `app.js` file. At this point when we run `npm start`, we should see `Connected to Mongo.` If you see an error, make sure that you are running `mongod` in a separate terminal window.

9. Create `routes/message.js` to handle our routing.

```JavaScript
var express = require('express');
var router = express.Router();
var mongoose = require('mongoose');

var MessageSchema = mongoose.Schema({
  name: String,
  message: String
});

var Message = mongoose.model('message', MessageSchema, 'messages');

router.get('/', function(req, res){

});

router.post('/', function(req, res){

});

router.put('/', function(req, res){

});

router.delete('/', function(req, res){

});

module.exports = router;
```

> Commit our changes

10. Update **app.js** to include the following lines.

```JavaScript
var bodyParser = require('bodyParser');
var message = require('./routes/message.js');

// Angular converts your data to a JSON object
app.use(bodyParser.json()); // Required for Angular
app.use(bodyParser.urlencoded({extended: true}));
```

We now have the majority of our server side code complete. Let's switch gears and jump over to the client.

### MEAN Stack - client side

**client.js**

```JavaScript
var myApp = angular.module('myApp', []);

myApp.controller('BaseController', ['$scope', '$http', function($scope, $http){
    $scope.firstmessage = {
      name : "Scott",
      message : "Just go ahead and type in your first message."
    };

    $http.post('/message', $scope.firstmessage).then(function(response){
      console.log(response.data);
    });
}]);
```

**message.js**

Update our routes.

```JavaScript
router.get('/', function(req,res){
    Message.find({}, function(err, allMessages){
      if(err) {
        console.log('Error finding messages', err);
      }
      res.send(allMessages);
    });
});

router.post('/', function(req,res){
    var message = new Message({
        name: req.body.name,
        message: req.body.message
    });

    message.save(function(err, savedMessage){
      if(err){
        console.log("Mongo Error: ", err);
        res.sendStatus(500);
      }

      res.send(savedMessage);
    });
});
```

> Commit our changes

Let's refactor our **client.js**

```JavaScript
var myApp = angular.module('myApp', []);

myApp.controller('BaseController', ['$scope', '$http', function($scope, $http){
  // Declare our array
  $scope.messageList = [];

  $scope.getMessages = function() {
    $http.get('/message').then(function(response){
      //console.log(response);
      $scope.messageList = response.data;
    });
  }

  $scope.postMessage = function() {
    $http.post('/message', {/* replace with form variables */}).then(function(response){
      // Angular puts the array in a variable named data
      console.log(response.data);
    });
  }

  // This will run on page load
  $scope.getMessages();
}]);
```

Add `ng-repeat` in our HTML. More resources on [ng-repeat](https://blog.rjmetrics.com/2015/09/02/8-features-of-ng-repeat/).

**index.html**

Update your main html tag to include `ng-app="myApp"` and add the following content within your body tag.

```HTML
<div ng-controller="BaseController">
  <!-- Bind our inputs to the newMessage object -->
  <input type="text" ng-model="newMessage.name">
  <input type="text" ng-model="newMessage.name">
  <!-- Pass the newMessage object into the postMessage function -->
  <button type="button" ng-click="postMessage(newMessage)">Submit Message</button>
  <!-- messageList matches the variable we put in $scope
       message is a name we create (could be taco) -->
  <div ng-repeat="message as messageList">
    <!-- message is an object with a property name -->
    <p>{{message.name}}</p>
  </div>
</div>
```

Now add the following to your `client.js` scope for your BaseController:

```JavaScript
$scope.newMessage = {
  name: '',
  message: ''
}

$scope.postMessage = function(message) {
  console.log(message);
  // Add our newMessage here
  $http.post('/message', message).then(function(response){
    // Angular puts the array in a variable named data
    console.log(response.data);
    // We've add a message, retrieve the whole list again
    getMessages();
  });
}
```

> Commit our changes

Final project can be found [here](https://github.com/scottbromander/chi_mean).
