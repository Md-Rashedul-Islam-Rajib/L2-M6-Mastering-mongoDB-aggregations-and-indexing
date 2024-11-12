# Mastering MongoDB aggregation and indexing


## $match aggregation operator

`$match` operator functions similar to `find` command but it works only under the `aggregate` command and select the data which matches with the provided conditions

```javascript
db.test.aggregate([ //aggregate command
    {$match : {gender : "Male"}}  // $match operator with conditions
])
```

