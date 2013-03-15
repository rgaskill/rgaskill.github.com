---
layout: post
title: IndexedDb deleteDatabase
---

So you want to programarically delete your IndexedDb client-side datastore.  It is actually pretty easy. Just call deleteDatabae like
so:

    window.indexedDB.deleteDatabase('some_database_name');

An important item to be a aware of is the promise that is returned from the deleteDatabase method will not complete until
all the other connections to the database are closed.  I have created a simple [testacular](http://testacular.github.com/0.6.0/index.html) test
[here](https://github.com/rgaskill/indexeddb-test) to illistrate this. To run this test pull the repository and run
using testacular.

    git clone git@github.com:rgaskill/indexeddb-test.git

    ## If testacular isn't installed
    npm install -g testacular

    cd indexeddb-test

    testacular start

The test is currently configured to run in chrome but it will run in Firefox as well.
If you want to run the test in Firefox update the [testacular.conf.js](https://github.com/rgaskill/indexeddb-test/blob/master/testacular.conf.js)

    //replace
    browsers = ['Chrome'];

    //with
    browsers = ['Firefox'];


#### The Tests

In the [test/IndexedDbSpec.js](https://github.com/rgaskill/indexeddb-test/blob/master/test/IndexedDbSpec.js) file you will see a few brief tests.
Listed below is a brief overview of the tests.

##### it("should delete database after explicit close", function() {

In this test the first "runs" function opens a connection to the database and then the second "runs" function deletes
the database.  If you were to comment out [line 34](https://github.com/rgaskill/indexeddb-test/blob/master/test/IndexedDbSpec.js#L34) (`db.close();`) this test would timeout waiting for the databaseDelete promise
to complete. *The runs functions are a [Jasmine](https://github.com/pivotal/jasmine/wiki/Asynchronous-specs) construct to allow code to be run serially.*

    runs(function(){

        db.close(); //EXPLICIT DB CLOSE

        var request = window.indexedDB.deleteDatabase(dbName);
        var done = false;

        request.onerror = function(event) {
            console.log("delete error", event);
            done = true;
        };

        request.onsuccess = function(event) {
            console.log("delete success", event);
            done = true;
        };

        waitsFor(function() {
            return done;
        },5000);


    });

##### it("should delete database after version change event", function() {

In this test rather than closing the database explicitly prior to deleting the database I added a listener at [line 76](https://github.com/rgaskill/indexeddb-test/blob/master/test/IndexedDbSpec.js#L76)
(`db.onversionchange = function(event) {`) that is fired when the deleteDatabase is called. If the version property of the event
is empty it is assumed the database is attempting to be deleted.

    db.onversionchange = function(event) {
        console.log("version change", event);
        //empty version implies that the database is being deleted
        if ( !event.version ){
            db.close();
        }
    };

##### describe('IndexedDb Test Template', function() {

This is just a simple template to show one possible way to open a connection to the IndexedDb before each test and
delete the IndexedDb test after each test is run.

    beforeEach(function() {

        runs(function() {

            var done = false;
            var request = window.indexedDB.open(dbName);


            request.onerror = function(event) {
                console.log("open error", event);
                done = true;
            };

            request.onsuccess = function(event) {
                console.log("open success", event);
                db = this.result;
                done = true;
            };

            waitsFor(function() {
                return done;
            },5000);

        });

    });

    afterEach(function() {

        runs(function(){

            db.close();

            var request = window.indexedDB.deleteDatabase(dbName);
            var done = false;

            request.onerror = function(event) {
                console.log("delete error", event);
                done = true;
            };

            request.onsuccess = function(event) {
                console.log("delete success", event);
                done = true;
            };

            waitsFor(function() {
                return done;
            },5000);


        });

    });
