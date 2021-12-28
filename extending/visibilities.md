# Visibilities

The visibility part defines what you can see at all. Anything you cannot see, you're implicitly also not allowed to modify. The visibility classes define what you see depending on your roles, permissions, etc. Building on top of this follow the permission classes (see below) that define what you can do with the data you see.

Visibility classes are configured using the environment variable `VISIBILITY_CLASSES`.

The following pre-defined classes are available:

* `caluma.caluma_core.visibilities.Any`: Allow any user without any filtering
* `caluma.caluma_core.visibilities.Union`: Union result of a list of configured visibility classes. May only be used as base class.
* `caluma.caluma_user.visibilities.Authenticated`: Only show data to authenticated users
* `caluma.caluma_user.visibilities.CreatedByGroup`: Only show data that belongs to the same group as the current user
* `caluma.caluma_workflow.visibilities.AddressedGroups`: Only show case, work item and document to addressed users through group

In case this default classes do not cover your use case, it is also possible to create your custom visibility class defining per node how to filter.

Example:

```python
from caluma.caluma_core.visibilities import BaseVisibility, filter_queryset_for
from caluma.caluma_core.types import Node
from caluma.caluma_form.schema import Form


class CustomVisibility(BaseVisibility):
    @filter_queryset_for(Node)
    def filter_queryset_for_all(self, node, queryset, info):
        return queryset.filter(created_by_user=info.context.user.username)

    @filter_queryset_for(Form)
    def filter_queryset_for_form(self, node, queryset, info):
        return queryset.exclude(slug='protected-form')
```

Arguments:

* `node`: GraphQL node filtering queryset for
* `queryset`: [Queryset](https://docs.djangoproject.com/en/2.1/ref/models/querysets/) of specific node type
* `info`: [The `info` object](../docs/interfaces.md#the-info-object)

Save your visibility module as `visibilities.py` and inject it as Docker volume to path `/app/caluma/extensions/visibilities.py`, see [docker-compose.yml](https://github.com/projectcaluma/caluma/blob/main/docker-compose.yml) for an example.

Afterwards you can configure it in `VISIBILITY_CLASSES` as `caluma.extensions.visibilities.CustomVisibility`.
