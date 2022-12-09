# Getting started

This guide gives a practical introduction to Caluma. It will show you how to

* get a simple app running,
* build a form, and
* design a simple workflow.

## Installation

### Part 1: Caluma API (backend)

To install Caluma, you'll need to have [Docker](https://docs.docker.com/install/) and [docker-compose](https://docs.docker.com/compose/install/) installed on your system.

Afterwards, create a new directory for your project and copy our [example docker-compose.yml file](https://github.com/projectcaluma/caluma/blob/main/docker-compose.yml) into it. Before you can start the containers, you'll need to do two more steps:

* Per default, Caluma is running with production settings. To bypass the security-related configuration steps needed for a production system, create a new file called `docker-compose.override.yml` with the following content:

```yml
version: "3.4"
services:
  db:
    environment:
      - POSTGRES_PASSWORD=caluma
  caluma:
    environment:
      - ENV=development
```

* Set the UID of the user inside the container::

```bash
echo "UID=$(id --user)" > .env
```

Now you can start the containers by running

```bash
docker-compose up -d
```

After, you should be able to access [GraphiQL](https://github.com/graphql/graphiql) at [http://localhost:8000/graphql](http://localhost:8000/graphql). We'll use GraphiQL to interact with the Caluma service in the coming sections.

### Part 2: ember-caluma (frontend)

First, you'll need to [install the Ember.js framework](https://guides.emberjs.com/release/getting-started/quick-start/).

After, create a new ember app by running

```bash
ember new caluma-demo
```

Inside the new app install the Ember.js addons for Caluma forms (`ember-form` is responsible for rendering Caluma forms, while `ember-form-builder` offers an UI to create and modify Caluma forms).

{% hint style="info" %}
The following steps are a more detailed version of the [ember-caluma installation docs](https://docs.caluma.io/ember-caluma/docs) - if this guide should not cover a recent change in ember-caluma, please let us know or send a PR!
{% endhint %}

```bash
ember install @projectcaluma/ember-form-builder @projectcaluma/ember-form
```

To make `@projectcaluma/ember-form-builder` work a few steps must be followed. The form builder is a [routable engine](http://ember-engines.com) which must be mounted in `app/router.js`:

```bash
Router.map(function() {
  // ...

  this.mount("@projectcaluma/ember-form-builder", {
    as: "form-builder",
    path: "/form-builder"
  });
});
```

To make `ember-apollo-client` work, we need to pass it as dependency for our engine in `app/app.js`. Additionally, we need to specify `ember-intl` as a dependency so that the the application has access to the addon's translations.

```bash
const App = Application.extend({
  // ...

  engines: {
    "@projectcaluma/ember-form-builder": {
      dependencies: {
        services: [
          "apollo", // ember-apollo-client for graphql
          "notification", // ember-uikit for notifications
          "intl", // ember-intl for i18n
          "caluma-options", // service to configure ember-caluma
          "validator" // service for generic regex validation
        ]
      }
    }
  }
});
```

Also, we'll need to tell the apollo client where the GraphQL API can be reached in `config/environment.js`:

```bash
module.exports = function(environment) {
  // ...

  apollo: {
    apiURL: "/graphql"
  },
};
```

To use the ember-uikit notification service, it ist important to configure some attributes in `ember-cli-build.js`:

```bash
module.exports = function(defaults) {
  let app = new EmberApp(defaults, {
    "ember-uikit": {
      notification: {
        "timeout": 5000,
        "group": null,
        "pos": "top-center",
      }
    }
  });
```

Last but not least import `ember-uikit`, `@projectcaluma/ember-form-builder` and `@projectcaluma/ember-form` to apply styling in `app/styles/app.scss`:

```bash
// https://github.com/uikit/uikit/blob/master/src/scss/variables-theme.scss
// custom variable definitions go here

$modal-z-index: 1;

@import "ember-uikit";

@import "@projectcaluma/ember-form";
@import "@projectcaluma/ember-form-builder";
```

To show some texts in the form-builder you can configure the locale setting of `ember-intl` to `en` (instead of the default, `en-us`), for example in `application/route.js` (run `ember g route application` if it doesn't exist yet):

```javascript
import Route from '@ember/routing/route';
import { inject as service } from '@ember/service';

export default class ApplicationRoute extends Route {
  @service intl;

  beforeModel() {
    this.intl.setLocale(['en']);
  }
}
```
