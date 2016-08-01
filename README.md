# Blueprint (REST)

## create

Replace the default create controller allowing to call the default view after an element creation :

```javascript
create: function(req, res) {
    Model.create(req.params.all()).exec(function(err, created) {
      if (err) return res.negotiate(err);
      return res.redirect('/model/'+created.id);
    });
  }
```

# Socket

## Client side

Get request

```javascript
io.socket.get('/url', {}, function(data, jwres) {
  // jwres.statusCode
  // jwres.headers
  // data = jwres.body
}
```

# Waterline (ORM)

## Basic controller

```javascript
module.exports = {
  action: function(req, res) {
    Model.find().exec(function(e, results) {
      if (e) {console.trace(e); return res.status(500).send(e);}
      
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
