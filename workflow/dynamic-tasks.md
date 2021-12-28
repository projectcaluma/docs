# Dynamic tasks

## Dynamic Tasks

Caluma allows you to extend the JEXL evaluation in the `next` property of a flow. This `next` property decides which task(s) will be started on completing or skipping a work item.

This concept can be used to implement an exclusive choice flow. E.g if the document of the case has a certain answer it will start a different task than normally.

To configure a dynamic tasks class, you can set the `DYNAMIC_TASKS_CLASSES` environment variable. When using the Caluma service, you can place your class in `caluma.extensions.dynamic_tasks` and set `DYNAMIC_TASKS_CLASSES` to `["caluma.extensions.dynamic_tasks.CustomDynamicTasks"]`.

### Example

```python
from caluma.caluma_workflow.dynamic_tasks import (
    BaseDynamicTasks,
    register_dynamic_task,
)


class CustomDynamicTasks(BaseDynamicTasks):
    @register_dynamic_task("audit-forms")
    def resolve_audit_forms(self, case, user, prev_work_item, context):
        needs_extra_audit = case.document.answers.filter(
            question_id="is-extra-complicated",
            value="is-extra-complicated-yes"
        ).exists()
        next_tasks = ["audit"]

        if needs_extra_audit:
            next_tasks.append("extra-audit")

        return next_tasks
```

You can then use that defined dynamic task `"audit-forms"` in the `next` property by postfixing it with a `task` or `tasks` transform: `["audit-forms"]|tasks`. Please keep in mind that if you use the singular `task` transform, the dynamic task method can only return a single task or it will raise an exception.
