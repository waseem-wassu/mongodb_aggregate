# mongodb_aggregate
db.col1.aggregate([
  {
    $match: {
      $and: [
        {
          "createdAt": {
            $gte: "2022-10-01"
          }
        },
        {
          "createdAt": {
            $lte: "2022-10-10"
          }
        }
      ]
    }
  },
  {
    "$addFields": {
      "myField": {
        "$toDate": "$createdAt"
      }
    }
  },
  {
    $group: {
      _id: {
        day: {
          $dayOfMonth: "$myField"
        }
      },
      "count": {
        $sum: "$count"
      }
    }
  },
  {
    $project: {
      _id: 0,
      day: "$_id.day",
      count: 1
    }
  },
  {
    $group: {
      _id: null,
      points: {
        $push: "$$ROOT"
      }
    }
  },
  {
    $project: {
      points: {
        $map: {
          input: {
            $range: [
              1,
              5
            ]
          },
          as: "day",
          in: {
            $let: {
              vars: {
                hourIndex: {
                  "$indexOfArray": [
                    "$points.day",
                    "$$day"
                  ]
                }
              },
              in: {
                $cond: {
                  if: {
                    $ne: [
                      "$$hourIndex",
                      -1
                    ]
                  },
                  then: {
                    $arrayElemAt: [
                      "$points",
                      "$$hourIndex"
                    ]
                  },
                  else: {
                    day: "$$day",
                    count: null
                  }
                }
              }
            }
          }
        }
      }
    }
  }
])
