**A guide on *How to hygiene!***

In the context we are in, hygiene describes how to clear up the explores so that Explorer users are not overflooded with unnecessary amounts of data; there are a few ways to achieve that:

 - Use `hidden: yes` in order to hide fields being shown in the explorer view. This is straight forward and easy, but can be quite manual especially when you have to hide/unhide fields as you have to go through the code and toggle every dimension/measure manually
 - Use a `set` in order to define an array of fields that can be called by referencing the set. This takes a bit of initial set up, but is from my point of view way more scalable than the first option

Recommending the second version, this is how it can be achieved.

 - For every view in use, define a set of fields that you require:
 ```
 set:  fields_to_show  {
fields:  [field_a, dimension_b, measure_c]
}
 ```

 - In the explore definition, use the `fields:` parameter to define what fields are being shown in the defined explore
 ```
 explore: explore_1 {
  from: view_a
  fields: [view_a.fields_to_show*, view_b.fields_to_show*]
    join: view_b {}
}
```

 - Using this method, also enables to individualize the fields to be shown, based on the explore the view is used in; a view can be part of multiple explores, but you might want to have different fields presented --> use different sets for that, this is not possible by simply `hidden:yes` the fields.
 - Make sure that you define the fields on the **explore** level. Its also possible on the join level, but that would require you to add them on the explore level as well; this means additional work.

Talking about hygiene should not only factor in the obvious and visible things, but also the code. We have branched out the code into individual view files and explore files already, we have also utilized separate explore files to keep that clean (remember, these things need to be included in the model file so that they know which connection to use). One thing we did not cover but which I implemented was the use of refinements in order to implement a `query`. [Link to query documentation.](https://docs.looker.com/reference/explore-params/query)

The `query` parameter creates those quick start tiles that are visible when an explore opened; they are prefilled explores essentially. The code required for a `query` is quite extensive and goes directly into the explore definition, so if a number of those are wanted, code can get pretty messy. Example:
```
explore: explore_a {
  from: view_a
  label: "Example"
  join: view_b {
    sql_on: view_a.id = view_b.id
    relationship: many_to_one
    type: left_outer
  }
    join: view_c {
    sql_on: view_a.id = view_c.id
    relationship: many_to_one
    type: left_outer
  }
    join: view_d {
    sql_on: view_a.id = view_d.id
    relationship: many_to_one
    type: left_outer
  }
    query: cogs_by_tag {
    label: "COGS by product tag"
    description: "Cost of goods sold by product tag ATTN: this is using current cost of the good"
    dimensions: [products_domain.tags]
    measures: [order_items_domain.total_cogs]
    sorts: [order_items_domain.total_cogs: desc]
  }
}
```
* In order to not expand the explore definition even further, we can use the *[refinement](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwivj92PoIT3AhWOHewKHardAjYQFnoECAgQAQ&url=https%3A%2F%2Fdocs.looker.com%2Fdata-modeling%2Flearning-lookml%2Frefinements&usg=AOvVaw3kdOMmugIV-7NMOZDfAOoi)* feature to "outsource" the query definition and create a cleaner structure.
* On the CF instance, I have done that and the queries are stored in view files in the `queries` folder.
* The refinement is indicated by the `+` sign and the beginning of the explore definition
```
 explore: +already_existing+explore {}
```
* We facilitate that by `including` the original explore file to the new view file, and then refine the explore with the additional code that we want:
```
explore: +already_existing {
  query: query_name {
    label: "Label of query"
    description: "Description of query"
    dimension: [dimension_to_be_prefilled]
    measures: [measure_to_be_prefilled]
  }
}
```
* The code will be added to the original source; *refined*
