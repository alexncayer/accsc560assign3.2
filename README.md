
//Alex Cayer CSC 560 Assign 3.2 Attempt 4
//Purpose of the program: To create API from CSC 560 Assign 2 that can add, delete, update, 
//and 5 chosen functions.
//Note that I did utilize parts of the code from Sudeep Timalsina's tutorial, 
//"Create your First REST API with Node.js, Express and MongoDB"
//as a guide to help build each of the steps (Timalsina, 2020).
//But, did make sure to implement some code not talked about in tutorial as well.
//In GitHub, view in the "raw" mode under the README file to see correct code spacing.

//index4.js down below

//Import Express
let express = require('express');

//Allows application to start
let app = express();

//Chooses specific port to use
var port = process.env.PORT || 8080;

//Welcome message for testing purposes
app.get('/', (req, res) => res.send('Welcome to Express'));

//Helps start the application at port number
app.listen(port, function() {
    console.log("Running csc560a3p2 on Port" + port)
})

//Helping to look over the parts regarding application/json. 
//This was referenced to help adding new data as noted from user Rafael Sanchez in Timalsina's article 
//(Timalsina, 2020).
app.use(express.json())
app.use(express.urlencoded({ extended: true})) //"parsing application/x-www-form-urlencoded" (Timalsina, 2020).

//Import routes itself
let apiRoutes = require("./routes")
//Then utilizes the routes above for application
app.use('/api', apiRoutes)

//import body parser
let bodyParser = require('body-parser');
//imports the mongoose package
let mongoose = require('mongoose');

//sets up bodyparser to deal with "post requests" (Timalsina, 2020)
app.use(bodyParser.urlencoded ({
    extended: true
}));

//links up with mongoose. Utilized 127.0.0.1 due to not being able to connect to mongodb with localhost.
//Found this out when viewing Database Administrators' post called "Unable to use Mongodb in NodeJS", which
//suggests to use 127.0.0.1 over localhost (Database Administrators, 2022).
const dbPath = 'mongodb://127.0.0.1/csc560a3p2new4';
const options = {useNewUrlParser: true, useUnifiedTopology: true}
const mongo = mongoose.connect(dbPath, options);

mongo.then(() => {
    console.log('connected');
}, error => {
    console.log(error, 'error');
})

//curlingbioModel.js down below
//Helps make the model in the first place.

var mongoose = require('mongoose');

//schema itself down below
var curlingbioSchema = mongoose.Schema({
    name: {
        type: String,
        required: true
    },
    age: {
        type: Number,
        required: true
    },
    gender: {
        type: String,
        required: true
    },
    position: {
        type: String,
        required: true
    },
    teamname: {
        type: String,
        required: true
    },
    totalthrowsthatstayinplay: {
        type: Number,
        required: true
    },
    totaleliminatethrows: {
        type: Number,
        required: true
    },
    sweepturnspergame: {
        type: Number,
        required: true
    },
    teamwins: {
        type: Number,
        required: true
    },
    yearsofexperience: {
        type: Number,
        required: true
    }
});

//Sends out the Curlingbio Model
var Curlingbio = module.exports = mongoose.model('curlingbio', curlingbioSchema);

module.exports.get = function (callback, limit) {
    Curlingbio.find(callback).limit(limit);
}

//routes.js of csc560a3p2 version 4
//Helps test API itself

//sets up the express router
let router = require('express').Router();

//set default API response
router.get('/', function (req, res) {
    res.json({
        status: 'API Works',
        message: 'Welcome to Alex C CSC560A3P2 API'
    })
})

//Utilizes the Bio Controller to help with routing
var curlingbioController = require('./curlingbioController');

//Curlingbio routes
router.route('/curlingbio')
    .get(curlingbioController.index)
    .post(curlingbioController.add);

router.route('/curlingbio/:curlingbio_id')
    .get(curlingbioController.view)
    .patch(curlingbioController.update)
    .put(curlingbioController.update)
    .delete(curlingbioController.delete);


//Attempted routes down below that resulted in something like "Curlingbio.findByTeamname is not a function".
//Thinking it is a route error after talking with Dr. Litman on 2/15/2023, which had the same thought (Litman, 2023).
//router.route('/curlingbio/:curlingbio_teamname')
    //.get(curlingbioController.view);
//router.route('/curlingbio/:curlingbio_gender')
    //.get(curlingbioController.view);
//router.route('/curlingbio/:curlingbio_yearsofexperience')
    //.get(curlingbioController.view);
//router.route('/curlingbio/:curlingbio_age')
    //.get(curlingbioController.view);
//router.route('/curlingbio/:curlingbio_totalthrowsthatstayinplay')
    //.get(curlingbioController.view);

//Export API routes
module.exports = router;

//curlingbioController.js
//Purpose is to handle many different requests.

//Utilizes Bio Model or in this case, it is "imported" (Timalsina, 2020)
Curlingbio = require('./curlingbioModel');

//Index portion below
exports.index = function (req, res) {
    Curlingbio.get(function (err, curlingbio) {
        if (err)
            res.json({
                status: "error",
                message: err
            });
        res.json({
            status: "success",
            message: "Got Curlingbio Successfully!",
            data: curlingbio
        });
    });
};

//For creating new Curling bio on person
exports.add = function(req, res) {
    var curlingbio = new Curlingbio();
    curlingbio.name = req.body.name? req.body.name: curlingbio.name;
    curlingbio.age = req.body.age;
    curlingbio.gender = req.body.gender;
    curlingbio.position = req.body.position;
    curlingbio.teamname = req.body.teamname;
    curlingbio.totalthrowsthatstayinplay = req.body.totalthrowsthatstayinplay;
    curlingbio.totaleliminatethrows = req.body.totaleliminatethrows;
    curlingbio.sweepturnspergame = req.body.sweepturnspergame;
    curlingbio.teamwins = req.body.teamwins;
    curlingbio.yearsofexperience = req.body.yearsofexperience;

    //Saves the new bio and then checks to see if any error happened
    curlingbio.save(function (err) {
        if (err)
            res.json(err);
        res.json({
            message: "New Curlingbio on Player Added!",
            data: curlingbio
        });
    });
};

//View Bio regarding Id, which can help identify specific user.
//Example is Katie Brief, which has the lowest totalthrowsthatstayinplay of 3 (performed in query 1 of 
// CSC 560 Assignment 2 on the CurlingteamsUpdated database. This query was taken as a screenshot for proof
//for that assignment).
exports.view = function (req, res) {
    Curlingbio.findById(req.params.curlingbio_id, function (err, curlingbio) {
        if (err)
            res.send(err);
        res.json({
            message: 'Curlingbio Details on Player(s)',
            data: curlingbio
        });
    });
};

//Update Bio
exports.update = function (req, res) {
    Curlingbio.findById(req.params.curlingbio_id, function (err, curlingbio) {
        if (err)
            res.send(err);
        curlingbio.name = req.body.name ? req.body.name : curlingbio.name;
        curlingbio.age = req.body.age;
        curlingbio.gender = req.body.gender;
        curlingbio.position = req.body.position;
        curlingbio.teamname = req.body.teamname;
        curlingbio.totalthrowsthatstayinplay = req.body.totalthrowsthatstayinplay;
        curlingbio.totaleliminatethrows = req.body.totaleliminatethrows;
        curlingbio.sweepturnspergame = req.body.sweepturnspergame;
        curlingbio.teamwins = req.body.teamwins;
        curlingbio.yearsofexperience = req.body.yearsofexperience;

        //save Curlingbio player changes and checks if any errors occured.
        curlingbio.save(function (err) {
            if (err)
                res.json(err)
            res.json({
                message: "Curlingbio of Player Updated Successfully",
                data: curlingbio
            });
        });
    });
};

//Delete Curlingbio of specific player
exports.delete = function (req, res) {
    Curlingbio.deleteOne({
        _id: req.params.curlingbio_id
    }, function (err, contact) {
        if (err)
            res.send(err)
        res.json({
            status: "success",
            message: 'Curlingbio of Player Deleted'
        })
    })
}

//Attempt of finding lowest throw that stay in play.
//exports.view = function(req, res) {
    //Curlingbio.findByTotalthrowsthatstayinplay(req.params.curlingbio_totalthrowsthatstayinplay, function(err, curlingbio) {
        //if (err)
            //res.send(err);
        //res.json({
            //message: "Found Curlinbio Player that has a certain amount of total throws that stay in play",
            //data: curlingbio
        //})
    //})
//}

//View Curlingbio of Players on specific teams based on teamname.
//exports.view = function (req, res) {
    //Curlingbio.findByTeamname(req.params.curlingbio_teamname, function (err, curlingbio) {
        //if (err)
            //res.send(err);
        //res.json({
            //message: 'Curlingbio Team Player Details',
            //data: curlingbio
        //});
    //});
//};

//View Curlingbio of Players who have specific gender or in this case, male or female.
//exports.view = function (req, res) {
    //Curlingbio.findByGender(req.params.curlingbio_gender, function (err, curlingbio) {
        //if (err)
            //res.send(err);
        //res.json({
            //message: 'Curlingbio Team Player(s) who are this Gender Details',
            //data: curlingbio
        //})
    //})
//}

//Sorts players by years of experience in reverse order or in this case descending from most years to lowest.
//query can be used to order by specific query parameters or in this case, try to order from ascending to descending.
//exports.view = function (req, res) {
    //Curlingbio.sortByYearsofexperience(req.query.curlingbio_yearsofexperience, function (err, curlingbio) {
        //if (err)
            //res.send(err);
        //res.json({
            //message: 'Curlingbio Team Player(s) sorted by yearsofexperience',
            //data: curlingbio

        //})
    //})


//}

//Sorts players by age. Trying to sort players who are greater than 30 years old.
//exports.view = function(req, res) {
    //Curlingbio.filterByAge(req.query.curlingbio_age, function(err, curlingbio) {
        //if (err)
            //res.send(err);
        //res.json({
            //message: 'Curlingbio Team Player(s) filtered by age',
            //data: curlingbio

        //})
    //})
//}

//package.json file down below

{
  "name": "csc560a3p2new4",
  "version": "1.0.0",
  "description": "Creating API for CSC 560 assign 2",
  "main": "index4.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "restapi"
  ],
  "author": "Alex Cayer",
  "license": "ISC",
  "dependencies": {
    "body-parser": "^1.20.1",
    "express": "^4.18.2",
    "expression": "^0.0.1",
    "mongoose": "^6.9.1",
    "routes": "^2.1.0"
  }
}

//End of code here
