# Lecciones y Aprendizajes #1

- Rails autoloading [http://urbanautomaton.com/blog/2013/08/27/rails-autoloading-hell/](http://urbanautomaton.com/blog/2013/08/27/rails-autoloading-hell/)
- About [foreign_keys](https://robots.thoughtbot.com/referential-integrity-with-foreign-keys)
- Multiple foreign keys to same table: [SO](http://stackoverflow.com/questions/2166613/multiple-foreign-keys-referencing-the-same-table-in-ror)
- Remove postgresql from server [http://stackoverflow.com/questions/2748607/how-to-thoroughly-purge-and-reinstall-postgresql-on-ubuntu#2748644](http://stackoverflow.com/questions/2748607/how-to-thoroughly-purge-and-reinstall-postgresql-on-ubuntu#2748644)
- [create_table migration](http://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/create_table)
- `[add_foreign_key](http://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/add_foreign_key)` migration
- % syntax https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Literals

Do not validate belongs_to: [SO](http://stackoverflow.com/questions/38689139/remove-required-validation-for-belong-to-attribute-on-rails-5)
User:

    belongs_to :user, optional: true

Get an enum integer value: [SO](http://stackoverflow.com/questions/25570209/how-get-integer-value-from-a-enum-in-rails)
Rails 5: [SO](http://stackoverflow.com/a/26756192/1407371)


## Problemas con consultas para datos tipo float (bet)
1. http://stackoverflow.com/questions/26920559/postgres-rails-active-record-query-for-rounded-float-value
2. http://stackoverflow.com/questions/17069895/sql-server-like-against-a-float-field-produces-inconsistent-results
3. [Lectura](http://stackoverflow.com/questions/8514167/float-vs-decimal-in-activerecord)


## use `_prefix` or `_suffix` to reutilize enum values
- http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html
- https://github.com/rails/rails/issues/17511


- Sugerencias y caso similar [http://stackoverflow.com/questions/23712485/share-enum-declaration-values-with-multiple-attributes](http://stackoverflow.com/questions/23712485/share-enum-declaration-values-with-multiple-attributes)


## `Enumerable#include?(obj)` finds efficiently an item in an array

http://ruby-doc.org/core-2.3.1/Enumerable.html#method-i-include-3F

    nots = [8, 16, 32]
    p nots.include?(12)
    => false

`Enumerable#find` did not convince me because it iterates over the array

    nots = [8, 16, 32]
    nots.find {|item| p item == 19}
    => false
    => false
    => false

