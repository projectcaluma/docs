# Form validation

Our rationale about using jexl can be found [here](../docs/jexl.md).

### Variables available for evaluation

Caluma provides a number of variables to use within JEXL expressions. We assume that you're familiar with the expression language itself, so we're not explaining the syntax here.

* `info` is a data structure containing information about the document and it's form. It's typology is roughly as outlined below. Common access patterns are:
  * `info.form` - The slug of the direct form where the question resides. Could be a row form if the question is within a table, or a FormQuestion's form, etc.
  * `info.formMeta` - The meta of the form.
  * `info.root` - The **root form** of the document.
    * `info.root.form` - The root form's slug.
       This is useful if you use a question in various forms, where it should behave differently. You could for example write an expression for `is_required` like this: `info.root.form == 'building_permit'`. If you use the question in a form named "building_permit", it would be required, but if you use it in a form named "general_request", it would not be required.
    * `info.root.formMeta` - The root form's meta.
  * `info.parent` - The **form above the current form**. May be useful in nested structures (e.g. subforms or table rows). This is `null` for questions in the root form.
    * `info.parent.form` - The parent form's slug.
    * `info.parent.formMeta` - The parent form's meta.
    * `info.parent.question` - The slug of the question the current form is attached to. This is the table question for a row form, or the form question for a subform.
  * `info.case` - The **case the document belongs to**. This works both for documents of a case, and for documents of a work item, where it refers to the work item's case. This is `null` if the document is not attached to a case.
    * `info.case.form` - The slug of the case's form.
    * `info.case.workflow` - The slug of the case's workflow.
    * `info.case.meta` - The case's meta.
    * `info.case.root` - The **root case**. For a case without a parent, this is the case itself.
      * `info.case.root.form` - The slug of the root case's form.
      * `info.case.root.workflow` - The slug of the root case's workflow.
      * `info.case.root.meta` - The root case's meta.
  * `info.workItem` - The work item the document belongs to. For a document of a case, this is the work item that created the case. This is `null` if the document is not attached to a work item (either via `document.work_item` or `document.case.parent_work_item`).
    * `info.workItem.task` - The slug of the work item's task.
    * `info.workItem.meta` - The work item's meta.

Expressions using the properties that can be `null` should be written to tolerate that: `info.case.meta` fails to evaluate on a document without a case, while `info.case != null && info.case.meta.foo == 'bar'` is safe.

### Transforms

Transforms are used to turn one type of information into another, similar to pipes in the unix shell. They can be chained, and may be parametrized as well.

Here are the available transforms:

* `answer`: When applied to a question slug, returns the answer to that question in the context of the current document. For example `'your-name'|answer` could evaluate to "David". In a `is_required` context, you'll need a boolean value, so you could for example use `'your-name'|answer == 'Fred'` in the `is_required` field of the "birthday" question, so only Fred needs to tell us his birthday. Normally, if the requested question is not found in the form it will throw an exception. However, sometimes it's not necessary to fail so the JEXL works in multiple environments. To allow this we can pass a parameter `default_value` to the answer transform: `'nonexistent-question'|answer('default')`. This parameter declares that the question doesn't have to exist in the form and will be returned if it doesn't exist.
* `mapby`: Extract a nested value from a list. Assuming you have a table of things currently in the fridge, and you want help the user to decide what to cook: `'ravioli' in 'fridge-contents'|answer|mapby('food-name')` will tell you whether there are ravioli. If you pass multiple keys, you get a list of lists instead: `'fridge-contents'|answer|mapby('food-name', 'amount')` could evaluate to `[['ravioli', 2], ['cheese', 1]]`. Returns `null` if applied to something that isn't a list.
* `length`: The number of entries in a list, or the number of characters in a text: `'fridge-contents'|answer|length > 3`. Returns `null` for values that have no length, such as numbers.
* `flatten`: Turns a list of lists into a flat list. `[[1, 2], [3]]|flatten` evaluates to `[1, 2, 3]`. This is useful for tables within tables, where `mapby` gives you one list per row. Returns `null` if applied to something that isn't a list.
* `min`, `max`, `sum`, `avg`: Aggregate a list of numbers, for example `'fridge-contents'|answer|mapby('amount')|sum`. Entries that aren't numbers are ignored, so this also works on tables where some rows are unanswered. On an empty list, `sum` gives `0`, while `min`, `max` and `avg` give `null`.
* `round`: Round a number to the given number of decimal places, `3.14159|round(2)` evaluates to `3.14`. Without a parameter it rounds to a whole number.
* `ceil`, `floor`: Round a number up resp. down to a whole number. `2.1|ceil` evaluates to `3`, `2.9|floor` to `2`.
* `stringify`: The JSON representation of a value, without any whitespace: `{ foo: 1 }|stringify` evaluates to `{"foo":1}`. This is mostly useful to compare structured values, as JEXL has no deep equality.

Only for debugging purposes:

* `debug`: Does not modify the value, but writes the value to the log. This is especially useful when you are exploring the data while building forms or workflows. The log message may appear in different places depending on where it's being run (Browser: console log in the debug tools, Server: System or container logs, might depend on your logging configuration). You can pass a label to tell multiple debug outputs apart: `'your-name'|answer|debug('name')`.

### Operators

Other operators that aren't transforms are also available:

* `in`: Tells us if a value (left) is contained in a list (right).
* `intersects`: Tells us if two lists have intersecting values. In a way, this is a generalized `in` operator. This can be used for example in multiple choice questions, as follows: To hide questions only relevant for vegetarians or vegans, hide those questions by asking some food choices first, then doing this: `'food_choices'|answer intersects ['meat', 'fish']` to see if you should ask specific questions

## FormatValidators

FormatValidator classes can validate input data for answers, based on rules set on the question.

There are some basic FormatValidators ready to use (see "base\_format\_validators" in [format\_validators.py](https://github.com/projectcaluma/caluma/blob/main/caluma/caluma_form/format_validators.py). There is also an [extension point](../general/extending/format-validators.md) for them.
