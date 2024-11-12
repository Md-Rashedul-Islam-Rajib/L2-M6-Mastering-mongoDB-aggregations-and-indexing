# Mastering MongoDB aggregation and indexing


## $match aggregation operator

`$match` operator functions similar to `find` command but it works only under the `aggregate` command and select the data which matches with the provided conditions

```javascript
db.test.aggregate([ //aggregate command
    {$match : {gender : "Male"}}  // $match operator with conditions
])
```


## $project aggregate operator

`$project` aggregate operator functions similar to `project` command. it takes the field name of a document what you want to return.


```javascript
db.test.aggregate([ //aggregate command
    {$match : {gender : "Male"}},  // $match operator with conditions
    {$project : {name: 1,age:1}} //field want to return with the document
])
```

Note : Never use `$project` aggregation operator in the first stage. it may create issue due to elimination of some fields with operator


## $addFields aggregation operator

`$addFields` aggregation operator adds field to a temporary document and return it. It's not modify the original document.


```javascript
db.test.aggregate([
    //stage-1
    {$match : {gender : "Male"}},
    //stage-2
    {$addFields : {hobby : "Traveling"}},
    //stage-3
    {$project : {name:1,gender: 1,hobby: 1}}
])
```

## $out aggregation operator

we know `$addFields` operator add field to a temporary document and won't modify the original document. so if we need to save the modified document to a new collection here come the `$out` operator handy.


```javascript
db.test.aggregate([
    //stage-1
    {$match : {gender : "Male"}},
    //stage-2
    {$addFields : {hobby : "Traveling"}},
    //stage-3
    {$project : {name:1,gender: 1,hobby: 1}}, //if you need all the fields of original document, then skip stage 3 
    //stage-4
    {$out : "newCollection"} // a collection will created with named "newCollection" and save the projected field with value
])
```