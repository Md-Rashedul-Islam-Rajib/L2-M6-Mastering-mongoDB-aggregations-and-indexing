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

## $merge aggregation operator

after using `$addFields` operator, if we need to save the new field's in the original document then we can use the `$merge` operator. it will merge those fields what created by `$addFields` and return the new modified document


```javascript
db.test.aggregate([
    //stage-1
    {$match : {gender : "Male"}},
    //stage-2
    {$addFields : {hobby : "Traveling"}},
    //stage-3
    {$project : {name:1,gender: 1,hobby: 1}}, //if you need all the fields of original document, then skip stage 3 
    //stage-4
    {$merge : "test"} // hobby field will merge in the test collection
])
```

## $group aggregation operator

`$group` operator used to grouping documents by a specific field or expression.

```javascript
db.test.aggregate([
    {
        $group : {
            _id : "$gender"  //this will create group for each different value of gender field in the document
        }
    }
])
```

## $sum aggregate operator

after grouping document by using `$group` operator, if we need to know that how many data hold by each group then the `$sum` operator comes handy. it's an accumulator what sums up the total data from each different groups

```javascript
db.test.aggregate([
    {
        $group : {
            _id : "$gender",
            count : {$sum : 1} // this will show a field named "count" and it's value will be the total data of each groups holds
        }
    }
])
```

another example

```javascript
db.test.aggregate([
   {
      $group: {
         _id: "$category",      // Group by the `category` field
         totalSales: { $sum: "$amount" }   // Sum up the `amount` field's value for each group
      }
   }
]);

```


## $push aggregate operator

`$push` operator collects fields with it's value and create an array within the range of each group.

```javascript
db.test.aggregate([
    {$group : {
        _id : $gender,  // grouped by gender field
        count : { $sum : 1}, // accumulate the data count of each group
        returnedDoc : {$push : "$name"} // each group have only their "name" field what collected from the document and shown in the array named "returnedDoc"
    }}
])
```

another example

```javascript
db.test.aggregate([
   {
      $group: {
         _id: "$category",      // Group by `category`
         products: { $push: "$productName" }  // Collect all `productName` values into an array
      }
   }
]);

```
Note : if we want to add all the field to products array replace `"$productName"` with `"$$ROOT"`





















