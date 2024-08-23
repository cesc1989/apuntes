# 🔑 Athena SerDe Library Options 
SerDe stands for Serialization/Deserialization. Athena [provides libs](https://docs.aws.amazon.com/athena/latest/ug/supported-serdes.html) when working with CSV tables.

# LazySimpleSerDe

Docs: [LazySimpleSerDe](https://docs.aws.amazon.com/athena/latest/ug/lazy-simple-serde.html)
Library name: `org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe`

Notes:

> Specifying this SerDe is optional. This is the SerDe for data in CSV, TSV, and custom-delimited formats that Athena uses by default. This SerDe is used if you don't specify any SerDe and only specify `ROW FORMAT DELIMITED`. Use this SerDe if your data does not have values enclosed in quotes.


# OpenCSVSerDe

Docs: [OpenCSVSerDe](https://docs.aws.amazon.com/athena/latest/ug/csv-serde.html)
Library name: `org.apache.hadoop.hive.serde2.OpenCSVSerde`

Notes:

> If data contains values enclosed in double quotes (`"`), you can use the OpenCSV SerDe to deserialize the values in Athena.

**Issue with OpenCSVSerDe**
The table `"``patient-forms-production-db``"``.``"``production``"` was giving trouble querying because it was set to use the `OpenCSVSerDe` lib and the data isn’t enclosed in quotes.

    HIVE_BAD_DATA: Error parsing field value '' for field 3: For input string: ""

When I changed the SerDe library to `LazySimpleSerDe`, I could query everything as normal

# About Serde Properties

I couldn’t find a list of serde properties. I took a look into several pages in the docs of Apache Hive and found nothing resembling a list.

## field.delim or separatorChar

Depending on which Servo motor was choosen, one can use the `field.delim` property or the `separatorChar` one.


- When using `LazySimpleSerDe` the available property is `field.delim`.
- When using `OpenCSVSerde` the available property is `separatorChar`.
## skip.header.line.count

Set it to 1 in “Table properties” section to skip the first row of the csv. Usually that rows contains the names of the exported fields.


## Additional Links
- [Athena Guide: Working with CSVs](https://athena.guide/articles/working-with-csv/)


## More about properties
- [In AWS](https://docs.aws.amazon.com/athena/latest/ug/lazy-simple-serde.html)
- [In Confluence: Hive SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)
- [Row Formats and SerDe](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-RowFormats&SerDe)
- [Add SerDe Properties](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AddSerDeProperties)

It looks like it can be any arbitrary property:

    CREATE TABLE apachelog (
      host STRING,
      identity STRING,
      user STRING,
      time STRING,
      request STRING,
      status STRING,
      size STRING,
      referer STRING,
      agent STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
    WITH SERDEPROPERTIES (
      "input.regex" = "([^]*) ([^]*) ([^]*) (-|\\[^\\]*\\]) ([^ \"]*|\"[^\"]*\") (-|[0-9]*) (-|[0-9]*)(?: ([^ \"]*|\".*\") ([^ \"]*|\".*\"))?"
    )
    STORED AS TEXTFILE;

But according to [this other doc page](https://cwiki.apache.org/confluence/display/Hive/MultiDelimitSerDe), the serde_properties will depend on the SERDE library.

    CREATE TABLE test (
     id string,
     hivearray array<binary>,
     hivemap map<string,int>) 
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.MultiDelimitSerDe'                  
    WITH SERDEPROPERTIES ("field.delim"="[,]","collection.delim"=":","mapkey.delim"="@");

