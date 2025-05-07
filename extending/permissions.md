# Permissions

Permission classes define who may perform which mutation. They can be configured with the environment variable `PERMISSION_CLASSES`.

The following pre-defined classes are available:

* `caluma.caluma_user.permissions.IsAuthenticated`: only allow authenticated users
* `caluma.caluma_core.permissions.AllowAny`: allow any users to perform any mutation.
* `caluma.caluma_user.permissions.CreatedByGroup`: Only allow mutating data that belongs to same group as current user

In case this default classes do not cover your use case, it is also possible to create your custom permission class defining per mutation and mutation object what is allowed.

Example:

```python
from caluma.caluma_core.permissions import BasePermission, permission_for, object_permission_for
from caluma.caluma_form.schema import SaveForm
from caluma.caluma_core.mutation import Mutation


class CustomPermission(BasePermission):
    @permission_for(Mutation)
    def has_permission_default(self, mutation, info):
        # change default permission to False when no more specific
        # permission is defined.
        return False

    @permission_for(SaveForm)
    def has_permission_for_save_form(self, mutation, info):
        return True

    @object_permission_for(SaveForm)
    def has_object_permission_for_save_form(self, mutation, info, instance):
        return instance.slug != 'protected-form'
```

Arguments:

* `mutation`: mutation class
* `info`: [The `info` object](../docs/interfaces.md#the-info-object)
* `instance`: instance being edited by specific mutation

Save your permission module as `permissions.py` and inject it as Docker volume to path `/app/caluma/extensions/permissions.py`, see [docker-compose.yml](https://github.com/projectcaluma/caluma/blob/main/docker-compose.yml) for an example.

Afterwards you can configure it in `PERMISSION_CLASSES` as `caluma.extensions.permissions.CustomPermission`.

If your permissions depend on the content of the mutation, you can access those values via the `get_params()` method on the mutation, like the following example shows:

```python
class CustomPermission(BasePermission):
    @permission_for(SaveTextQuestionInput)
    def has_question_save_permission(self, mutation, info):
        params = mutation.get_params(info)
        # hypothetical permission: disallow saving
        # questions whose label contains the word "blah"
        return "blah" not in params['input']['label']
```
