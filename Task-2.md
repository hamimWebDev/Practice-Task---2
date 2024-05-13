### Que: 1. Retrieve the count of individuals who are active (isActive: true) for each gender.

#### Ans: 1.

```
db.BigData.aggregate([
    { $match: { isActive: true } },
    { $group: { _id: "$gender", count: { $sum: 1 } } }
])

```

---

### Que: 2.Retrieve the names and email addresses of individuals who are active(`isActive: true`) and have a favorite fruit of "banana".

#### Ans: 2.

```
db.BigData.find(
    { isActive: true, favoriteFruit: "banana" },
    { _id: 0, name: 1, email: 1 }
)

```

---

### Que: 3.Find the average age of individuals for each favorite fruit, then sort the results in descending order of average age.

#### Ans: 3.

```
db.BigData.aggregate([
    { $group: { _id: "$favoriteFruit", averageAge: { $avg: "$age" } } },
    { $project: { _id: 0, favoriteFruit: "$_id", averageAge: 1 } },
    { $sort: { averageAge: -1 } }
])

```

---

### Que: 4.Retrieve a list of unique friend names for individuals who have at least one friend, and include only the friends with names starting with the letter "W".

#### Ans: 4.

```
db.BigData.aggregate([
    {
        $unwind: "$friends"
    },
    {
        $match: { "friends.name": /^W/ }
    },
    {
        $group: {
            _id: "$_id", unickFriends: {
                $addToSet: "$friends.name"
            }
        }
    }
])

```

---

### Que: 5.Use $facet to separate individuals into two facets based on their age: those below 30 and those above 30. Then, within each facet, bucket the individuals into age ranges (e.g., 20-25, 26-30, etc.) and sort them by name within each bucket.

#### Ans: 5.

```
db.BigData.aggregate([
  {
    $facet: {
      "below_30": [
        {
          $match: {
            age: { $lt: 30 }
          }
        },
        {
          $bucket: {
            groupBy: "$age",
            boundaries: [20, 25, 30],
            default: "30+",
            output: {
              "names": { $push: "$name" }
            }
          }
        }
      ],
      "above_30": [
        {
          $match: {
            age: { $gte: 30 }
          }
        },
        {
          $bucket: {
            groupBy: "$age",
            boundaries: [31, 35, 40],
            default: "40+",
            output: {
              "names": { $push: "$name" }
            }
          }
        }
      ]
    }
  },
  {
    $project: {
      "below_30": {
        $map: {
          input: "$below_30",
          as: "ageGroup",
          in: {
            age_range: {
              $concat: [
                { $toString: { $arrayElemAt: ["$below_30.boundaries", { $indexOfArray: ["$below_30.names", "$$ageGroup.names"] }] } },
                "-",
                { $toString: { $arrayElemAt: ["$below_30.boundaries", { $add: [{ $indexOfArray: ["$below_30.names", "$$ageGroup.names"] }, 1] }] } }
              ]
            },
            names: "$$ageGroup.names"
          }
        }
      },
      "above_30": {
        $map: {
          input: "$above_30",
          as: "ageGroup",
          in: {
            age_range: {
              $concat: [
                { $toString: { $arrayElemAt: ["$above_30.boundaries", { $indexOfArray: ["$above_30.names", "$$ageGroup.names"] }] } },
                "-",
                { $toString: { $arrayElemAt: ["$above_30.boundaries", { $add: [{ $indexOfArray: ["$above_30.names", "$$ageGroup.names"] }, 1] }] } }
              ]
            },
            names: "$$ageGroup.names"
          }
        }
      }
    }
  }
])


```

---

### Que: 6.Calculate the total balance of individuals for each company and display the company name along with the total balance. Limit the result to show only the top two companies with the highest total balance.

#### Ans: 6.

```
db.BigData.aggregate([
  {
    $addFields: {
      balance: {
        $toDouble: {
          $replaceAll: {
            input: { $substr: ["$balance", 1, { $strLenCP: "$balance" }] },
            find: ",",
            replacement: ""
          }
        }
      }
    }
  },
  {
    $group: {
      _id: "$company",
      totalBalance: { $sum: "$balance" }
    }
  }
])

```

---
