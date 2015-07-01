# Mongoose Reverse Populate

[![Build Status](https://travis-ci.org/Prismatik/mongoose-reverse-populate.svg?branch=master)](https://travis-ci.org/Prismatik/mongoose-reverse-populate)

This module allows you to 'populate' a mongoose model (referred to as the 'model to populate') where the relationship ids are stored on another mongoose model that is related to this model (referred to as the 'related model').

For example, you have an Author Model and a Post Model and the Post Model stores the Author id.
Note: An Author can have many Posts but a Post can only have one Author.

You can use a standard populate call if you wish to retrieve all Posts and populate their Author. But you cannot use this if you want to retrieve all Authors and populate all of that Author's Posts as the ids reside on the wrong model. 

## Using the function

```
var reversePopulate = require('mongoose-reverse-populate');

//the next step requires access to the 'Author' and 'Post' mongoose model

Author.find().exec(function(err, authors) {

    var opts = {
        modelArray: authors,
        storeWhere: "posts",
        arrayPop: true,
        mongooseModel: Post,
        idField: "author"
    }


	reversePopulate(opts, function(err, popAuthors) {
		//popAuthors will be populated with posts under .posts property
	});
});

```
## Inputs

The function expects an opts object (options) and callback to be passed into it

function(opts, callback) {...}

The opts object should contain the following properties...

* modelArray - the array of 'models to populate' (authors)
* storeWhere - where should the 'related models' be stored (which property i.e. "posts") within the 'models to populate' (those in the modelArray).
* arrayPop - if the 'model to populate' has many 'related models' this should be set to true. This ensures the results of the reverse populate are stored as an array (e.g. an Author has many Posts). If the 'model to populate' can only have one 'related model' this should be set to false (e.g. a User has one Address).
* mongooseModel - the mongoose model object to use to find the 'related model' e.g. Post
* idField - the property of the 'related model' that contains the _id of the 'model to populate' e.g. "author"

The callback function will be run once the objects are populated and will receive the populated modelArray as the success parameter of the function.


## Why is this needed?

You could use .find to fetch all the Authors and then loop through all Authors and use another .find to fetch all the Posts that include that Author's Id but this would require one query to the database for every Author that was found in the result set. Calls to the database are expensive and performance suffers.

It would be much more efficient to fetch all of the Authors and then perform a single query for every Post that contains includes an author id from Authors you are looking for. The downside of this approach though is you then need to distribute all of the posts returned against their respective authors. This complexity is handled within the function which places the load on the server rather than the database for better performance.


## Where are the populated models?

You will notice if you perform a console.log statement on your 'models to populate' after using the reverse populate that it may appear as if no models were populated. This is because the property the 'related models' are being added to is not defined in the schema but be assured the population has still work.

In the above, if you console.log the popAuthors they will appear as if they have no .posts property. But if you loop through the popAuthors and console.log the posts property of each author you will then see the results as expected.