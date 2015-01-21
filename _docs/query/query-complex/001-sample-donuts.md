---
title: "Sample Data: Donuts"
parent: "Querying Complex Data"
---
The complex data queries use sample `donuts.json` and `moredonuts.json` files.
Here is the single complete "record" (`0001`) from the `donuts.json `file. In
terms of Drill query processing, this record is equivalent to a single record
in a table.

    {
      "id": "0001",
      "type": "donut",
      "name": "Cake",
      "ppu": 0.55,
      "batters":
        {
          "batter":
            [
               { "id": "1001", "type": "Regular" },
               { "id": "1002", "type": "Chocolate" },
               { "id": "1003", "type": "Blueberry" },
               { "id": "1004", "type": "Devil's Food" }
             ]
        },
      "topping":
        [
           { "id": "5001", "type": "None" },
           { "id": "5002", "type": "Glazed" },
           { "id": "5005", "type": "Sugar" },
           { "id": "5007", "type": "Powdered Sugar" },
           { "id": "5006", "type": "Chocolate with Sprinkles" },
           { "id": "5003", "type": "Chocolate" },
           { "id": "5004", "type": "Maple" }
         ]
    }

The data is made up of maps, arrays, and nested arrays. Name-value pairs and
embedded name-value pairs define the contents of each record. For example,
`type: donut` is a map. Under `topping`, the pairs of `id` and `type` values
belong to an array (inside the square brackets).