# 🚦Creating New Tables in Glue

Specially for Patient Forms exports.

Related doc [+🔑 Athena SerDe Library Options](https://paper.dropbox.com/doc/Athena-SerDe-Library-Options-Qv8zDcqmfMts6p8h11T3p) 

# Important Properties

![[creating.table.athena.png]]

When working with values in the CSV that aren’t enclosed in quotes (e.g “lefs”), we have to use `LazySimpleSerDe`.

Using that library means we have to use another couple of parameters and properties to be able to query.


## Serde parameters

Use `field.delim` set to comma to indicate the separator in the CSV. In Patient Forms we separate them by commas.


## Table Properties

Use `skip.header.line.count` set to number 1 to indicate first row of the csv file has to be skipped. That row has the names of the exported fields.

