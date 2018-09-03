# M121
## Connection string to 'aggregations' database
mongo "mongodb://cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017/aggregations?replicaSet=Cluster0-shard-0" --authenticationDatabase admin --ssl -u m121 -p aggregations --norc


##Template
db.movies.aggregate([
    {
        $match: {
            "year": {$gte:2017}
        }
    }
])

## Chapter 1
### Lab - $match
var pipeline = [{$match: {"imdb.rating":{$gte:7}, "genres": {$nin:["Crime","Horror"]}, "rated": {$in: ["PG", "G"]}, $and:[{"languages":"English"}, {"languages": "Japanese"}] }}]

db.movies.aggregate(pipeline).itcount()

### Lab - $project
var pipeline = [{$match: {"imdb.rating":{$gte:7}, "genres": {$nin:["Crime","Horror"]}, "rated": {$in: ["PG", "G"]}, $and:[{"languages":"English"}, {"languages": "Japanese"}] }}, {$project: { _id:0, title:1, rated: "$imdb.rating" }} ]

### Lab - Computing field

db.movies.aggregate([
    {
        $project: {
            _id:0,
            title:1,
            numberOfWords: {$size: {$split: ["$title"," "]}}
        }
        
    },
    {$match: {
        numberOfWords: {$eq:1}
        }
    }
]).itcount()

## Chapter 2

### Lab - Using Cursor-like Stages

db.movies.aggregate([
    {
        $project: {
            _id:0,
            title:1,
            numberOfWords: {$size: {$split: ["$title"," "]}}
        }
        
    },
    {$match: {
        numberOfWords: {$eq:1}
        }
    }
])

db.movies.aggregate([
    {
        $match: {
            "countries": "USA",
            "tomatoes.viewer.rating": {$gte:3}
        }
    },
    {
        $project: {
            _id:0,
            title:1,
            rated: "$tomatoes.viewer.rating"
           
        }
    },
    {
        $sort: {
            rated:-1,
            title: -1
        }
    }
])


##Chapter 3

### $group
db.movies.aggregate([
    {
        $group: {
            _id: "$year",
            num_films_in_year: {$sum:1}
        }
    },
    {
        $sort: {
            num_films_in_year: -1
        }
    }
])

### 
db.movies.aggregate([
    {
        $group: {
            _id: {
                numDirectors: {
                    $cond: [{$isArray: "$directors"},{$size: "$directors"}, 0]
                }
            }
        }
    },
    {
        $sort: {"_id.numDirectors":-1}
    }
])

### Lab- $group and Accumulators
db.movies.aggregate([
    {
        $match: {
            "awards": {$regex: /Won\s\d{1,2}\sOscar/}
            }
    },
    {
        $group: {
            _id: null,
            average_rating: {$avg: "$imdb.rating"},
            deviation: {$stdDevSamp: "$imdb.rating"},
            highest_rating: {$max: "$imdb.rating"},
            lowest_rating: {$min: "$imdb.rating"},
            total: {$sum: 1}
        }
    }
])

