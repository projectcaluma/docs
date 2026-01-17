# Data sources

For `Choice` and `MultipleChoice` questions it's sometimes necessary to populate the choices with calculated data or data from external sources.

For this you can use the data\_source extension point.

An example data\_source looks like this:

```python
from caluma.caluma_data_source.data_sources import BaseDataSource
from caluma.caluma_data_source.utils import data_source_cache
import requests

class CustomDataSource(BaseDataSource):
   info = 'User choices from "someapi"'

   @data_source_cache(timeout=3600)
   def get_data(self, user, question, context):
       response = requests.get(
          f"https://someapi/?user={user.username}&question={question.slug}&key={context.key}"
       )
       return [result["value"] for result in response.json()["results"]]
```

This class needs also to be added to the `DATA_SOURCE_CLASSES` environment variable.

### Properties

* `info`: Descriptive text for the data source (can also be a multilingual dict)
* `default`: The default value to be returned if execution of `get_data()` fails. If this is `None`, the Exception won't be handled. Defaults to `None`.

### `get_data`-method

Must return an iterable. This iterable can contain strings, ints, floats and also iterables. Those contained iterables can consist of maximally two items. The first will be used for the option slug, the second one for it's label. If only one value is provided, this value will also be used as label.

For the label, it's possible to use a dict with translated values.

Note: If the labels returned from`get_data()` depend on the current user's language, you need to return a `dict` with the language code as keys instead of translating the value yourself. Returning already translated values is not supported, as it would break caching and validation.

### `on_copy`-method

When a an answer of type TYPE_DYNAMIC_CHOICE or TYPE_DYNAMIC_MULTIPLE_CHOICE gets copied the meaning of the slug and label of the dynamic option could potentially be changed when the datasource it's data changes. During the answer copy process the linked datasource it's on_copy method will be called to decide the desired result.

- return the same slug,label tuple to not perform any change (default behavior)
- return an altered slug or(/and) label to change the answer value
- return a None value for the slug, to discard the answer value

#### Arguments

* `user`: The OIDC user object for the request
* `question`: The question on which the data source was called upon. Defaults to `None` if the data source was called outside of question context.
* `context`: A `dict` containing additional information from the frontend. Defaults to `None`. This can be used to pass application specific information (such as the current case for example) to the data sources. To use it in the frontend, pass the needed data into the `@context` argument of the `<CfContent />` component.

### `validate_answer_value`-method

The default `validate_answer_value()`-method checks first if the value is contained in the output of `self.get_data()`. If it is, it returns the label. Else it makes a DB lookup to see if there is a `DynamicOption` with the same `document`, `question` and `slug` (that's the value). If there is at least one, it returns the label of the first one. Else it returns `False`.

If you override this method, make sure to return the label if valid, else `False`.

### `data_source_cache` decorator

This decorator allows for caching the data based on the DataSource name.

Django's cache framework is used, so you can also implement your own caching logic. When doing so, it is advisable to use the `data_source_` prefix for the key in order to avoid conflicts.

#### Some valid examples

```python
['my-option', {"en": "english description", "de": "deutsche Beschreibung"}, ...]

[['my-option', "my description"], ...]

['my-option', ...]

[['my-option'], ...]
```
