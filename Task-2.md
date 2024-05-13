### Que: 1. Retrieve the count of individuals who are active (isActive: true) for each gender.

#### Ans: 1.

```
db.BigData.aggregate([
    { $match: { isActive: true } },
    { $group: { _id: "$gender", count: { $sum: 1 } } }
])

```

---
