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