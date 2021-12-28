# Dynamic groups

Caluma allows you to extend the JEXL evaluation in the `address_groups` and `control_groups` properties of a `Task`. The result of this evaluation ends up in the generated `WorkItem`'s `addressed_groups` and `controlling_groups` fields respectively.

A typical use-case for such "dynamic groups" would be to define role based groups.

To configure a dynamic groups class, you can set the `DYNAMIC_GROUPS_CLASSES` environment variable. When using the Caluma service, you can place your class in `caluma.extensions.dynamic_groups` and set `DYNAMIC_GROUPS_CLASSES` to `["caluma.extensions.dynamic_groups.CustomDynamicGroups"]`.

### Example

```python
from your.project.code import find_legal_dept
from caluma.caluma_workflow.dynamic_groups import (
    BaseDynamicGroups,
    register_dynamic_group,
)


class CustomDynamicGroups(BaseDynamicGroups):
    @register_dynamic_group("legal_dept")
    def resolve_legal_dept(self, task, case, user, prev_work_item, context):
        # custom business logic which returns either a string or a list of strings
        return find_legal_dept(case)
```

You can then use that defined dynamic group `"legal_dept"` in the `Task` properties by postfixing them with a transform: `"legal_dept"|groups`. The created `WorkItem` would then have an evaluated value like `["legal-dept-boston"]` or `["legal-dept-nyc"]` for that property.

The `context` can hold an arbitrary dict passed in the `context` variable in a `SaveCase` or `CompleteWorkItem` mutation. Use it for passing additional info and implement logic on it, for example run-time dependent `"addressed_groups"` that should be added to newly generated `WorkItems`.

We can extend our example above:

```python
class CustomDynamicGroups(BaseDynamicGroups):
    @register_dynamic_group("legal_dept")
    def resolve_legal_dept(self, task, case, user, prev_work_item, context):
        # context may contain arbitrary data from the client to affect behaviour...
        legal_depts = find_legal_dept(case, coast=context.get("coast"))

        # ... or plain groups to include
        return legal_depts + context.get("additional_groups", [])
```

Now if the `context` looks like `{"coast": "west", "additional_groups": ["board", "administration"]}`, the resulting groups are `["legal-dept-san-francisco", "legal-dept-seattle", "board", "administration"]`.
