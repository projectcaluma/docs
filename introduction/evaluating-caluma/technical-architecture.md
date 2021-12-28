# Technical architecture

At it's heart, Caluma offers a GraphQL API for forms and workflows. It is typically deployed in a container using standard tools (e.g. Kubernetes) and is backed by a PostgreSQL database. The frontend is custom-built for each project, but standard components (addons) exist for Ember.js, allowing the rapid development of applications. Caluma doesn't handle authentication by itself, but instead requires an IAM application that supports OpenID connect.

![](https://codimd.adfinis.com/uploads/upload\_0a2849e92e39656f876bc8e8c3a84d4d.png)

The following services are not part of Caluma, but were designed to work well with it:

* Alexandria for document management (REST API)
* Emeis for user management (REST API)
* document-merge-service for document generation (REST API)

These services are usually deployed in separate containers. Since all services build on a common technology stack (Python/Django), it is also possible to implement a tighter integration by installing them as Django apps to form a single, larger Django application. More details about this can be found in the section _"How can Caluma be integrated in an existing technical landscape?"_.
