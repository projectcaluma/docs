# Visibilities

With the help of custom visibilities you can define which objects users can see on the Caluma GraphQL API. A typical implementation of a custom visibility is based on user's roles, permissions, etc. Objects that a user cannot see are also not allowed to be modified. If an object is visible, you can define under which circumstances changes to that object are allowed using custom permissions (which are explained in the next chapter).

Visibility classes are configured using the environment variable `VISIBILITY_CLASSES`.

The following pre-defined classes are available:

* `caluma.caluma_core.visibilities.Any`: Allow any user without any filtering
* `caluma.caluma_core.visibilities.Union`: Union result of a list of configured visibility classes. May only be used as base class.
* `caluma.caluma_user.visibilities.Authenticated`: Only show data to authenticated users
* `caluma.caluma_user.visibilities.CreatedByGroup`: Only show data that belongs to the same group as the current user
* `caluma.caluma_workflow.visibilities.AddressedGroups`: Only show case, work item and document to addressed users through group

In case these default classes do not cover your use case, you can create your own custom visibility class. Here is a basic example:

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

Each method decorated by `filter_queryset_for` receives three arguments (excluding `self`):

* `node`: GraphQL node filtering queryset for
* `queryset`: [Queryset](https://docs.djangoproject.com/en/2.1/ref/models/querysets/) of specific node type
* `info`: [The `info` object](../docs/interfaces.md#the-info-object)

Save your visibility module as `visibilities.py` and inject it as Docker volume to path `/app/caluma/extensions/visibilities.py`, see [docker-compose.yml](https://github.com/projectcaluma/caluma/blob/main/docker-compose.yml) for an example.

Afterwards you can configure it in `VISIBILITY_CLASSES` as `caluma.extensions.visibilities.CustomVisibility`.

### Performance considerations

Visibilities are checked for every relationship, which means that for a single GraphQL query, the visibility layer might run many times. For this reason, complex visibilities can cause performance issues. If you run into such problems, try profiling the query (e.g. using Django Silk) to identify the bottleneck.

#### Suppressable visibilities

Since Caluma v10, visibilities are also checked on foreign key relationships (e.g. the relationship between a `Question` and its `Form`). In practice, the visibility of such relationships is often not limited (since typically it does not make sense to see a `Question`, but not it's `Form`). In order to suppress visibility checks for specific foreign key relationships, you can set the `suppress_visibilities` property in your visibility class. For example:

```python
class MyCustomVisibility(BaseVisibility):
    suppress_visibilities = [
        "WorkItem.child_case",
        ...
    ]
    
    @filter_queryset_for(Case)
    def filter_queryset_for_case(self, node, queryset, info):
        # this method will not be executed when resolving a workitems' child case
```

The supported keys for the `suppress_visibilites` property can be identified by checking the [callsites of `suppressable_visibility_resolver`](https://github.com/search?q=repo%3Aprojectcaluma%2Fcaluma%20suppressable_visibility_resolver\&type=code).
