# OIDC User factory

Especially when using Caluma embedded in another Django project, you need to ensure compatibility with other components. Caluma does not represent users as a database object, but instead uses a "shim" object in it's role as an OIDC relying party (RP) that represents the requesting user. Oher projects may have differing requirements however.

Therefore, you can define a custom `CALUMA_OIDC_USER_FACTORY`. The setting is a string that points to a callable, which is expected to return an OIDC user object. By default it points to `caluma.caluma_user.models.OIDCUser`.

The factory needs to provide the following interface:

```python
    def user_factory(token, userinfo=None, introspection=None):
        # Either `userinfo` or `introspection` is filled with a dict
        # with information about the user.
        return SomeOIDCUserObject(...)
```

The resulting object is expected to provide the following properties:

* `username` - returns the username.
* `group` - The primary group of the user. Used to fill in `created_by_group` etc
* `is_authenticated` - Boolean, should always be `True`

You may extend the object as you please, for example by providing shortcuts to data you need often (reference to a Django user model if you use one, etc).
