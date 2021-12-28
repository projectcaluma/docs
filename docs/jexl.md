# Javascript Expression Language

Caluma relies on Javascript Expression Language alias [JEXL](https://github.com/TomFrost/Jexl) for defining powerful yet simple expressions for complex validation and flow definitions. The reason for using JEXL over other languages is that it can be simply implemented in the backend and frontend thanks to its defined small scope. This allows frontends in the browser to validate data in exactly the same way as the server will, while not limiting validation to simple regular expressions.

JEXL expressions are used in the following contexts:

* Deciding whether a field should be visible or not (`is_hidden` property)
* Deciding whether a field is required (`is_required` property)
* Deciding what groups should be addressed in a `WorkItem`
* Deciding what groups should have a controlling function on a `WorkItem`
* Deciding what `Task` should be used for the next `WorkItem`

Within the expressions, you can access various information about the context.

Specific information about the usage of jexl expression can be found in their respective documentation:

* [Document validation](../forms/validation.md)
* [Workflow WorkItem assignments](../workflow/workflow-workitem-assignments.md)
