# Change Form Type

# About

Jira ticket → https://lunacare.atlassian.net/browse/LPF-417
Ticket name → Review returning patient process

# Steps and Validations

Currently, we change the form type regardless of patient being first time or returning one.

First, Luxe makes a request to change the form type.

It sends two values:


- form type
- boolean to indicate whether or not to delete existing answers

The form type only changes if:


- The form is not completed,
- The new form type is a valid one,
- The form is NOT in progress
    - This means no question in the Functional Limitations section has been answered.

The last condition has an exception. When the boolean value mentioned above is send as “true” we can change the form type even if the form has answers.

If all validations pass, we change the form type, destroy any existing answers in the database and recreate all necessary data for the form to work correctly.

