# Format Validators

This extension point can be used to define custom field-level validations. Custom validators can either be regex-based or implemented as Python code.

### Regex-based validators

A custom regex-based format validator class could look like this:

```python
from caluma.caluma_form.format_validators import BaseFormatValidator
from django.utils.translation import gettext_lazy as _

class CustomFormatValidator(BaseFormatValidator):
     slug = "email"
     name = _("E-mail")
     regex = r"(^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$)"
     error_msg = _("Not an e-mail address")
```

### Python-based validators

For more complex validators you might prefer not to use regex - in this case you can implement a custom `is_valid` class method yourself. The method takes two arguments:

* the **value** that is being validated, and
* the **document** as context for more elaborate validations

Example:

```python
from caluma.caluma_form.format_validators import BaseFormatValidator
from django.utils.translation import gettext_lazy as _

class CustomFormatValidator(BaseFormatValidator):
    slug = "even-date"
    name = _("Even date")
    error_msg = _("Not an even date")

    @classmethod
    def is_valid(cls, value, document):
        return value.day() % 2 == 0
```
