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

## $avg aggregation operator

`$avg` operator calculates the average of specific numeric fields within the range of each group


```javascript
db.test.aggregate([
    {$group : {
        _id : "gender",
        avgSalary : { $avg : "$salary"} // avgSalary will store the average salary of salary field from each group
    }}
])
```

## $min aggregation operator

`$min` operator finds the minimum numeric value of a field within the range of a each


```javascript
db.test.aggregate([
    {$group : {
        _id : "gender",
        minSalary : { $min : "$salary"} // minSalary will store the minimum salary of salary field from each group
    }}
])
```

## $max aggregation operator

`$max` operator finds the maximum numeric value of specific field within the range of a group


```javascript
db.test.aggregate([
    {$group : {
        _id : "gender",
        maxSalary : { $max : "$salary"} // maxSalary will store the maximum salary of salary field from each group
    }}
])
```


## $first aggregation operator 

`$first` operator select the value of a specific field from the first document within the range of a group. it's very useful when we need to work with ordered data 



```javascript
db.test.aggregate([
    {$group : {
        _id : "gender",
        firstSalary : { $first : "$salary"} // firstSalary will store the first salary of salary field from each group
    }}
])
```

## $last aggregation operator


`$last` operator selects the value of a specific field from the last document within the range of a group. It's also very useful for working with ordered data


```javascript
db.test.aggregate([
    {$group : {
        _id : "gender",
        lastSalary : { $last : "$salary"} // lastSalary will store the last salary of salary field from each group
    }}
])
```



## $unwind aggregation operator

`$unwind` operator explode each array element into individual document with the exact same id or identifier. MongoDB will create a new document for each element in the array of an array field.


let think we have document structured like this

```json
{
    persons : {
        {
            name : 'mr x',
            age : 22,
            interests : ['gaming','traveling','hiking']
        },{
            name: 'mr y',
            age: 26,
            interests : ['reading','traveling','hiking']
        },{
            name: 'mr z',
            age: 25,
            interests : ['reading','riding', 'hiking']
        }
    }
}

```

now we want to to create group based on each unique value of interest array.  so we have to use `$unwind` operator for explode the element of interests array then try to use `$group` operator for creating group based on unique value of interests array.

```javascript
db.test.aggregate([
    {
        $unwind : "$interests" // flattening the interests array by using unwind operator
    },
    {
        $group : {
            _id : "$interests", // creating group based on the unique value of interests array
            count : {$sum : 1}
        }
    }
])
```


## $bucket aggregation operator

`$bucket` operator groups document based on a specific numeric field and put them into different bucket which are based on range-based interval.


```javascript
db.test.aggregate([
    {
        $bucket : {
            groupBy : "$age", //required parameter and it creates groups by the value of age field
            boundaries : [20,40,60], //required parameter and it's the interval of value for creating each bucket
            default : "above 60", // optional parameter and hold the data from outside boundaries
            output : { // optional parameter for define additional fields and accumulator
                count : { $sum : 1},
                fullDoc : { $push : "$$ROOT"}
            }
        }
    }
])
```

## $sort aggregation operator

`$sort` operator sorts documents based on a specific field's value and direction



```javascript
db.test.aggregate([
    {
        $bucket : {
            groupBy : "$age", 
            boundaries : [20,40,60],
            default : "above 60", 
            output : { 
                count : { $sum : 1},
                fullDoc : { $push : "$$ROOT"}
            }
        }
    },
    {$sort : {age : 1}} // use -1 for descending ordered sorting
])
```

## $limit aggregation operator

`$limit` operator restricts the number of document pass through aggregation pipeline to a specific number


```javascript
db.test.aggregate([
    {
        $bucket : {
            groupBy : "$age", 
            boundaries : [20,40,60],
            default : "above 60", 
            output : { 
                count : { $sum : 1},
                fullDoc : { $push : "$$ROOT"}
            }
        }
    },
    {$limit : 2} 
])
```