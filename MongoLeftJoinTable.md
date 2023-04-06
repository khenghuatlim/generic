# Possible Left Join on different Collection?
##### by khenghuatlim@gmail.com

How to query each user with respective usergroup (on separate collect) ?

### <span style="color:#ffbf00">Sample of collection - users</span>
```
{
    "_id" : ObjectId("63e22392aa7962100ca161cd"),
    "username" : "testing@coba.com",
    "firstname" : "myFN",
    "lastname" : "myLN",
    "usergroup_id" : [
        "63e129ed3f2ffe151537e3ae",
        "63e2060e4471155098c84db2"
    ]
}
```
At the usergroup_id has 2 ids referring to another collection ```user_groups```
```
{
    "_id" : ObjectId("63e129ed3f2ffe151537e3ae"),
    "name" : "Sungai01",
    "description" : "Group for user in Area001"
}

{
    "_id" : ObjectId("63e2060e4471155098c84db2"),
    "name" : "Sungai02",
    "description" : "Area002"
}
```

## Query on left join on `users`
```
db.users.aggregate([
  { "$addFields": {
      "ugId": {
        "$map": {
          "input": "$usergroup_id",
          "in": { "$toObjectId": "$$this" }
        }
      }
  }
      
  } , //endof addFields
  {
      $lookup:{
          localField: "ugId",
          from: "user_groups",
          foreignField: "_id",
          as: "output"
      }
  }
])
```


## Result
```
{
    "_id" : ObjectId("63e22392aa7962100ca161cd"),
    "username" : "testing@coba.com",
    "firstname" : "myFN",
    "lastname" : "myLN",
    "usergroup_id" : [
        "63e129ed3f2ffe151537e3ae",
        "63e2060e4471155098c84db2"
    ],
    "ugId" : [
        ObjectId("63e129ed3f2ffe151537e3ae"),
        ObjectId("63e2060e4471155098c84db2")
    ],
    "output" : [
        {
            "_id" : ObjectId("63e129ed3f2ffe151537e3ae"),
            "name" : "Sungai01",
            "description" : "Group for user in Area001"
        },
        {
            "_id" : ObjectId("63e2060e4471155098c84db2"),
            "name" : "Sungai02",
            "description" : "Area002",
        }
    ]
}
```

### Explanation
<b>Let's take a look on the Mongo variables explanation first.</b>

aggregate - `Based on the official documentation`- Aggregation operations process multiple documents and return computed results. You can use aggregation operations to:
Group values from multiple documents together.
Perform operations on the grouped data to return a single result.
Analyze data changes over time.

$addFields - Adds new fields to documents. $addFields outputs documents that contain all existing fields from the input documents and newly added fields.
At here, I used it as part of the comparison

$map - Applies an expression to each item in an array

$lookup - Performs a left outer join to a collection in the same database to filter in documents from the "joined" collection for processing. 
Syntax:
```
{
   $lookup:
     {
       from: <collection to join>,
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}
```

Since "usergroup_id" is an array. Thus it required to iterate each of the array and convert to Object Id via ($toObjectId) and assign to `ugId` for matching
```
  { 
    "$addFields": {
      "ugId": {
        "$map": {
          "input": "$usergroup_id",
          "in": { "$toObjectId": "$$this" }
        }
      }
  }
])
```


At $lookup, each of the ugId from iteration and match with _id in "user_groups" collection. Define the output as "output"
```
  {
      $lookup:{
          localField: "ugId",
          from: "user_groups",
          foreignField: "_id",
          as: "output"
      }
  }
```


---

How about query each `usergroups` who are the user on it?

```
db.user_groups.aggregate([
  { "$addFields": { "ugId": { "$toString": "$_id" }}},
  { "$lookup": {
    "from": "users",
    "localField": "ugId",
    "foreignField": "usergroup_id",
    "as": "output"
  }}
])
```

This is the result
```
{
    "_id" : ObjectId("63e129ed3f2ffe151537e3ae"),
    "name" : "Sungai",
    "description" : "Group for user in Sungai",
    "ugId" : "63e129ed3f2ffe151537e3ae",
    "output" : [        
        {
            "_id" : ObjectId("63e22392aa7962100ca161cd"),
            "username" : "testing@coba.com",
            "firstname" : "myFN",
            "lastname" : "myLN",
            "usergroup_id" : [
                "63e129ed3f2ffe151537e3ae",
                "63e2060e4471155098c84db2"
            ]
        }
    ]
}
{
    "_id" : ObjectId("63e2060e4471155098c84db2"),
    "name" : "punya Ryan",
    "description" : "gundul km",
    "ugId" : "63e2060e4471155098c84db2",
    "output" : [
        {
            "_id" : ObjectId("63e22392aa7962100ca161cd"),
            "username" : "testing@coba.com",
            "firstname" : "myFN
            "lastname" : "myLN",            
            "usergroup_id" : [
                "63e129ed3f2ffe151537e3ae",
                "63e2060e4471155098c84db2"
            ]
        }
    ]
}
```
