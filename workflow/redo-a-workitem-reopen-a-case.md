# Redo a WorkItem / Reopen a Case

### Overview

In the context of Caluma workflows we offer a redo process for work items (similar to an undo feature) and a `reopen` process for cases.

For more information on work items and cases see  [entities.md](../forms/entities.md "mention").

### Redo WorkItems

The [redo pattern](http://workflowpatterns.com/patterns/resource/detour/wrp34.php) allows us to redo an already finished work item including all subsequent work items.

> The ability for a resource to redo a work item that has previously been completed in a case. Any subsequent work items (i.e. work items that correspond to subsequent tasks in the process) must also be repeated.

#### Implications

Using this pattern implies the possibility of taking another path in subsequent runs. So a previously completed work item could stay in the `REDO` state. The `REDO` state on a work item should, especially in visibilities & permissions, be treated as if the work item didn't exist.

### Reopen a case

Once all the work items that belong to a case are `COMPLETED`, `CANCELED` or `SKIPPED` Caluma automatically closes the corresponding case.

However, in some circumstances it might be necessary to reopen a previously closed case. For such a situation Caluma offers the ability to reopen a case.

In order to reopen a closed case we use the reopen case mutation and supply it with one or multiple work items that will be set to the `READY` status. It is only possible to ready work items at the end of a workflow and that belong to the case that is being reopened.

