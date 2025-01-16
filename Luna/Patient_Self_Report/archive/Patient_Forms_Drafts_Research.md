# Patient Forms Drafts Research

## [draftpunk gem](https://github.com/stevehodges/draftpunk)
> Last updated: December 4th, 2018

First search result was a [Stack Overflow](https://stackoverflow.com/a/43793380/1407371) answer pointing to drafpunk gem.

Benefits of using this gem is it uses the same table as the being-saved model. It works by using a new column in the table. Kind of a foreign key to itself.

If chosen, bear in mind:

- Use the scope to list only approved versions. Normal queries would return all drafts and approved records.
## [drafting gem](https://github.com/ledermann/drafting)
> Last updated: March 29th, 2019

Requires to run a migration to create a table to store all drafts.

Usage is more straightforward than *draftpunk.* Place a macro in the model and use a method in the instance to save it as a draft.

## [draftman gem](https://github.com/jmfederico/draftsman)
> Last updated: March 25th, 2019

This one looks like the most robust of all. Also requires more attributes to work on the model to have drafts and uses a separate table.

## Ejemplo de Go Rails

GoRails guy did a test with *draftman* gem. [Source code](https://github.com/gorails-screencasts/autosave-draft-records).

