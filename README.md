# What is Caluma?

Caluma is a collaborative form and workflow service. For a big picture and to learn what Caluma does for you, have a look at [caluma.io](https://caluma.io).

## Quickstart

### Installation

NOTE: We recommend using Caluma as a dedicated service. However, it is possible to integrate Caluma into a django project. You can read about this [here](docs/django-apps.md).

**Requirements**

* docker
* docker-compose

After installing and configuring those, download [docker-compose.yml](https://github.com/projectcaluma/caluma/blob/main/docker-compose.yml) and run the following command:

```bash
docker-compose up -d
```

Schema introspection and documentation is available at [http://localhost:8000/graphql](http://localhost:8000/graphql) and can be accessed using a GraphQL client such as [Altair](https://altair.sirmuel.design/). The API allows to query and mutate form and workflow entities which are described below.

Caluma is a [12factor app](https://12factor.net/) which means that configuration is stored in environment variables. Different environment variable types are explained at [django-environ](https://github.com/joke2k/django-environ#supported-types).

You can read more about running and configuring Caluma under [docs/configuration.md](docs/configuration.md)

### Debugging

Set environment variable `ENV` to `dev` to enable debugging capabilities. Don't use this in production as it exposes confidential information!

This enables [Django Debug Middleware](https://docs.graphene-python.org/projects/django/en/latest/debug/).

For profiling you can use `./manage.py runprofileserver`. See [docker-compose.override.yml](https://github.com/projectcaluma/caluma/blob/main/docker-compose.override.yml) for an example.

## License

Code released under the [GPL-3.0-or-later license](https://github.com/projectcaluma/caluma/blob/main/LICENSE).

For further information on our license choice, you can read up on the [corresponding GitHub issue](https://github.com/projectcaluma/caluma/issues/751#issuecomment-547974930).
