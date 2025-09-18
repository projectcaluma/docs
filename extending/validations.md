# Validations

Validation classes can validate or amend input data of any mutation. Each mutation is processed in two steps:

1. Permission classes check if a given mutation is allowed based on the object being mutated, and
2. Validation classes process (and potentially amend) input data of a given mutation

A custom validation class defining validations for various mutations looks like this:

```python
from caluma.caluma_core.validations import BaseValidation, validation_for
from caluma.caluma_form.schema import SaveForm
from caluma.caluma_core.mutation import Mutation
from rest_framework import exceptions


class CustomValidation(BaseValidation):
    @validation_for(Mutation)
    def validate_mutation(self, mutation, data, info):
        # add your default specific validation code here
        return data

    @validation_for(SaveForm)                                                  
    def validate_save_form(self, mutation, data, info):                        
        if data['meta'] and 'admin' not in info.context.user.groups:           
            raise exceptions.ValidationException('May not change meta on form')
        return data                                                            

```

### The main group of the user

By default, Caluma assumes the first group in the list of groups it receives from the OIDC provider as the group the request was made in the name of. This means, this group will be written into the `created_by_group` field on the records that get created during the request.

It is possible to manually set this group in the validation class, in order to override this behavior:

```python
from caluma.caluma_core.validations import BaseValidation


class CustomValidation(BaseValidation):
    def validate(self, mutation, data, info):
        # the list of groups received from the OIDC provider can be accessed with
        # info.context.user.groups
        # Overriding the main group can be achieved by setting the `group` property on the user object:
        info.context.user.group = "foobar"
        return data
```

