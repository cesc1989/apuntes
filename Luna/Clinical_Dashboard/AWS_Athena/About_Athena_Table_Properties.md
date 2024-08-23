# About Athena Table Properties
Similar to what happens with documentation about [+About Athena SerDe Library](https://paper.dropbox.com/doc/About-Athena-SerDe-Library-Qv8zDcqmfMts6p8h11T3p) there’s no much out there about *table properties*.

In [Athena docs](https://docs.aws.amazon.com/athena/latest/ug/alter-table-set-tblproperties.html), we find the `ALTER TABLE SET TBLPROPERTIES` statement listing some but not all.

In [Quora](https://www.quora.com/What-are-the-table-properties-parameters-in-Hive-Where-can-I-get-a-list-of-it), found that Table Properties are metadata defined by the user. There’s a predefined list, though.

Finally, in [Hive docs](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTableProperties) we find the following:

    ALTER TABLE table_name SET TBLPROPERTIES table_properties;
     
    table_properties:
      : (property_name = property_value, property_name = property_value, ... )


> You can use this statement to add your own metadata to the tables. Currently last_modified_user, last_modified_time properties are automatically added and managed by Hive. Users can add their own properties to this list. You can do DESCRIBE EXTENDED TABLE to get this information.

Related to [Hive docs](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties).

