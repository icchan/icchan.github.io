---
title: Geospatial Querying with GoLang and MongoDB
layout: post
---

This week at work I had to build a pretty easy geospatial query api with go and mongodb, with some help from [mgo](https://labix.org/mgo) MongoDB driver.
MongoDB has [changed a lot](http://blog.mongolab.com/2014/08/a-primer-on-geospatial-data-and-mongodb/) since I last used it, so I had to learn how to use it again.

In short it now uses this thing called [GeoJSON](http://geojson.org/) which looks like this:

```Javascript
"location" : {
  "type" : "Point", 
  "coordinates" : [ 151.20699, -33.867487 ] 
}
```

It can actually represent a lot of other things like lines and polygons, but we are only interested in single points today,

Enough background, lets do this. 
This guide was written using Go v1.3 and MongoDB v2.6.5 and assumes you have go installed and also mongodb somewhere.

To do a geospatial query in mongodb for using longitude and latutide of places on Earth you'll probably want to use a "2dphere" index. 
So lets insert some documents into a mongodb and set a "2dsphere" index, type this in your mongo console: 

```Javascript
db.shops.insert({ "name" : "Shop in Sydney", "location" : { "type" : "Point", "coordinates" : [ 151.20699, -33.867487 ] } })
db.shops.insert({ "name" : "Studio Alta", "location" : { "type" : "Point", "coordinates" : [ 139.7011064, 35.692474 ] } })
db.shops.insert({ "name" : "ビックロ", "location" : { "type" : "Point", "coordinates" : [ 139.70328368, 35.69146649 ] } })
db.shops.insert({ "name" : "Keio Plaza Hotel", "location" : { "type" : "Point", "coordinates" : [ 139.69489306, 35.68999278 ] } })
db.shops.insert({ "name" : "明治神宮", "location" : { "type" : "Point", "coordinates" : [ 139.69936833, 35.67612138 ] } })
db.shops.insert({ "name" : "Hachiko", "location" : { "type" : "Point", "coordinates" : [ 139.7005894, 35.65905387 ] } })
db.shops.insert({ "name" : "Haneda Airport", "location" : { "type" : "Point", "coordinates" : [ 139.78569388, 35.54958159 ] } })
db.shops.ensureIndex({location:"2dsphere"})
```

This is just a bunch of places around Tokyo with GeoJSON coordinates. The last line is the important one as it sets the geospatial index.

Try out a geospatial query in the mongo console like this: 

```Javascript
db.shops.find({ location: { $nearSphere: { $geometry: { type: "Point", coordinates: [139.701642, 35.690647], }, $maxDistance : 50000 } } } )
```

This queries for the shops near a point in Shinjuku, Tokyo up to 5km away, sorted by distance. You should see all the shops except for "Shop in Sydney" (which is much more than 5km away)

Lets write some code, if you don't have mgo yet you can get it with this: 

```Bash
go get labix.org/v2/mgo
```

If you'd rather read code than words, you can view the full source here:
https://gist.github.com/icchan/bd42095afd8305594778

First create some structs to hold the data:

```Go
type ShopLocation struct {
	ID       bson.ObjectId `bson:"_id,omitempty" json:"shopid"`
	Name     string        `bson:"name" json:"name"`
	Location GeoJson       `bson:"location" json:"location"`
}

type GeoJson struct {
	Type        string    `json:"-"`
	Coordinates []float64 `json:"coordinates"`
}
```

Then get a mongodb session however you normally do it. Probably something like this: 

```Go
	cluster := "localhost" // mongodb host
	// connect to mongo
	session, err := mgo.Dial(cluster)
	if err != nil {
		log.Fatal("could not connect to db: ", err)
		panic(err)
	}
	defer session.Close()

```

Now do your query, note the query is basically the same as in the console but using a bson go struct.


```Go
	// search criteria
	long := 139.701642 
	lat := 35.690647
	scope := 3000 // max distance in metres

	var results []ShopLocation // to hold the results
	
	// query the database
	c := session.DB("test").C("shops")
	err = c.Find(bson.M{
		"location": bson.M{
			"$nearSphere": bson.M{
				"$geometry": bson.M{
					"type":        "Point",
					"coordinates": []float64{long, lat},
				},
				"$maxDistance": scope,
			},
		},
	}).All(&results)
```

If you run the gist from above, you should see an output like this: 

```Javascript
[
    {
       "shopid": "544215c13ef9cb393418ea25",
       "name": "ビックロ",
       "location": {
          "coordinates": [
             139.70328368,
             35.69146649
          ]
       }
    },
    {
       "shopid": "544215c13ef9cb393418ea24",
       "name": "Studio Alta",
       "location": {
          "coordinates": [
             139.7011064,
             35.692474
          ]
       }
    },
    {
       "shopid": "544215c13ef9cb393418ea26",
       "name": "Keio Plaza Hotel",
       "location": {
          "coordinates": [
             139.69489306,
             35.68999278
          ]
       }
    },
    {
       "shopid": "544215c13ef9cb393418ea27",
       "name": "明治神宮",
       "location": {
          "coordinates": [
             139.69936833,
             35.67612138
          ]
       }
    }
 ]
```
