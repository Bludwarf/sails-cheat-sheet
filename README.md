# Docs

  - http://expressjs.com/fr/api.html

# Blueprint (REST)

## [Overridable methods][OVMethods]

  - index : GET all items
  - find : GET one item
  - findOne (**attention** syntaxe complètement différents pour les findOneBy...)
  - [create](#blueprint_create)
  - update (3 params + return Array data)
  - destroy
  - populate
  - add
  - remove

## create <a name="blueprint_create"></a>

Replace the default create controller allowing to call the default view after an element creation :

```javascript
create: function(req, res) {
    Model.create(req.params.all()).exec(function(err, created) {
      if (err) return res.serverError(err);
      return res.redirect('/model/'+created.id);
    });
  }
```

## update

Uniquement depuis un **POST** client.
Attention Model.update renvoie un **tableau** d'objets modifié (et pas seulement un seul objet).

Exemple pour intercepter l'update dans le Controller correspondant :

```javascript
update: function(req, res) {
  // On supprime tous les paramètres vides qui devraient contenir des entiers pour éviter l'erreur "(E_VALIDATION) :: 3 attributes are invalid" grace à la Policy emptyBodyParams

  Model.update({id: req.params.id}, bodyParams).exec(function(err, updatedArray) {
      if (err) return res.serverError(err);
      var updated = updatedArray[0];
      return res.redirect(req.body.redirect_uri ? req.body.redirect_uri : '/model/'+updated.id);
    });
}
```


# Controller

  - [req](http://sailsjs.org/documentation/reference/request-req)
  - [res](http://sailsjs.org/documentation/reference/response-res)

## Multi model action

```javascript
action: function(req, res) {
  async.auto({

    items1: function(cb) {
      Model1.findOne({id: req.params.id})
        .populate('attribs1')
        .populate('attribs2')
        .exec(cb);
    },

    items2: function(cb) {
      Model2.find().exec(cb);
    }

  }, function(err, results) {
    if (err) return res.serverError(err);
    return res.view({
      items1: results.items1,
      items2: results.items2
    });
  });
}
```

# Policies

## emptyBodyParams [gist][gist1]

Empêche le client d'envoyer des paramètres vides ("") dans le body.

```javascript
module.exports = function(req, res, next) {

  _.forEach(req.body, function(bodyParamValue, bodyParamName) {
    if (bodyParamValue === '') {
      req.body[bodyParamName] = null;
      sails.log("req.body."+bodyParamName+" = \"\" -> null");
    }
  });

  next();
};
```

# Socket

## Client side

GET request

```javascript
io.socket.get('/url', {}, function(data, jwres) {
  // jwres.statusCode
  // jwres.headers
  // data = jwres.body
});
```

POST form request...
```javascript
// Sails data format
var data = _.reduce($(form).serializeArray(), function(hash, value) {
  var key = value['name'];
  hash[key] = value['value'];
  return hash;
});
```

... and return HTML content to update on server's side :
```javascript
Lien.create({
    param1: req.body.param1,
    param2: req.body.param2
  }).exec(function(err, created) {
  res.render('model/partial/view', {
    layout: false, // render only partial
    created: created
  });
});

// Controller gets data through req.body.input1 OR req.params.input1 OR req.param('input1')
io.socket.post(url, data, function(data, jwres) {
  // jwres.statusCode
  // jwres.headers
  // data = jwres.body
  if (jwres.statusCode != 200) {
    console.error(jwres.error);
    return alert(jwres.error.message);
  }
  $("#list").append(data);
});
```

# Waterline (ORM)

[Contrôles][https://github.com/balderdashy/waterline-docs/blob/master/models/validations.md].

## findOneBy...

```javascript
Model.findOneById(someValue,[options], function(err, item) {
  
});
```

## Basic controller

```javascript
module.exports = {
  action: function(req, res) {
    Model.find().exec(function(err, results) {
      if (err) return res.serverError(err);
      
      res.view({
        results: results
      });
    });
  }
}
```

## Errors

Duplicate entry

```javascript
{
  code: 'E_VALIDATION',
  invalidAttributes: {
    file: [{
      message: "A record with that `file` already exists (`C:\fakepath\file.jpg`).",
      rule: "unique",
      value: "C:\fakepath\file.jpg"
	}]
  },
  _e: [Error],
  rawStack: '...',
  reason: '1 attribute is invalid',
  status: 400,
  model: undefined,
  details: 'Invalid attributes sent to undefined:\n • file\n   • A record with that `file` already exists (`C:\fakepath\file.jpg`).\n'
}
```

## [populate]

Populating a model association

```javascript
populate('dad')
```

Populating a collection association. Second parameter is a [query].

```javascript
populate('currentSwords', {
  where: {
    color: 'purple'
  },
  limit: 3,
  sort: 'hipness DESC'
})
```

## [sort]

Sort by name in ascending order

```javascript
sort: 'name'
```


Sort by name in descending order

```javascript
sort: 'name DESC'
```


Sort by name in ascending order

```javascript
sort: 'name ASC'
```


Sort by binary notation

```javascript
sort: {
  'name': 1
}
```


Sort by multiple attributes

```javascript
sort: {
  name:  1,
  age: 0
}
```

[populate]: http://sailsjs.org/documentation/reference/waterline-orm/queries/populate
[query]: https://github.com/balderdashy/waterline-docs/blob/master/queries/query-language.md
[sort]: http://sailsjs.org/documentation/reference/waterline-orm/queries/sort
[OVMethods]: http://sailsjs.com/documentation/reference/blueprint-api#?blueprint-actions
[gist1]: https://gist.github.com/Bludwarf/821d2614ebbb5594fa1109007509d570
