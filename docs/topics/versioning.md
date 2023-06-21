# Major vs Minor Version (Best Practices)

The general guideline is to increment major version if introducing a breaking change for the REST API consumer, and increment minor version when all changes are non-breaking changes. We can assume the API consumer are following the guidelines specified [here](../front-end/best-practices.md). Below are some examples of when to increment each.

Increment Major Version When:

- Deleting or renaming a dataset
- Deleting or renaming column names
- Deleting or renaming parameter names
- Deleting a selection parameter option id (renaming an id should be prohibited)
- Changing the bounds for a number parameter

Increment Minor Version When:

- Adding a dataset
- Adding column names
- Adding parameters
- Adding selection parameter options
- Setting an existing dataset as deprecated (capability TBA) 
- Setting an existing column name as deprecated (capability TBA) 
- Setting an existing parameter as deprecated (capability TBA) 
- Setting an existing selection parameter option as deprecated (capability TBA) 
