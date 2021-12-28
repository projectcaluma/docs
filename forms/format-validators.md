# Format validators



Custom `FormatValidator` classes can be created to validate input data for answers, based on rules set on the question.

There are some [core FormatValidators](format-validators.md#formatvalidators) you can use.

A custom FormatValidator looks like this:

```python
from caluma.caluma_form.format_validators import BaseFormatValidator


class MyFormatValidator(BaseFormatValidator):
    slug = "my-slug"
    name = {"en": "english name", "de": "Deutscher Name"}
    regex = r"(^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$)"
    error_msg = {"en": "Not valid", "de": "Nicht gültig"}
```

For more complex validations, you can also additionally override the `validate()` method. Obviously, everything that happens in there must also be implemented in the corresponding frontend validation, if any.
