# populate

## Populating a model association

```javascript
populate('dad')
```

## Populating a collection association

```javascript
populate('currentSwords', {
  where: {
    color: 'purple'
  },
  limit: 3,
  sort: 'hipness DESC'
})
```
