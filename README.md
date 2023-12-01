Leihs - Azure Active Directory Open-ID Connect - Authentication System
=======================================================================

This project contains code and deployment recipes to use [Azure Active
Directory](https://azure.microsoft.com/de-de/services/active-directory/) via
[OpenID Connect](https://de.wikipedia.org/wiki/OpenID_Connect) as an external
authentication system for [leihs](https://github.com/leihs).


BREAKING Changes as of 2023-12
------------------------------

Concerns SSO Sign-Out:

* SSO sign-out triggered from leihs now requires a signed jwt-token. Before
  that a parameter in the url sufficed, which was open to CSRF missuese. The
  implications are very limited (start sign-out process) but potentially
  anoying for the user.

WHEN UPDATING: Upgrading leihs to 7.3.0 and later requires an update of the
authentication serivce to >= 2023-12 and upgades of the authentication service
require leihs 7.3.0 or later.




BREAKING Changes as of 2023-11
------------------------------

Deployment:

* the name of the default app, the location, and the default user have changed
  to be consistency with other (leihs-)apps

* the default config location has changed from the app directory to
  `/etc/leihs/{{app_name}}.config` to respect linux defaults and to
  automatically benefit from tools like etc-keeper

WHEN UPDATING: manually stop and (later) remove the old user, service and app
directory.




Choosing a Parameter Name
-------------------------

The internal path look for example like
'/authenticators/ms-open-id/:name/sign-in' where '/authenticators/ms-open-id/'
is the fixed prefix `:name` is a parameter to be chosen. We recommend to choose
a name based on the organization. However it should only contain lowercase
ascii characters, numbers `-` and `_`. It may in particular not contain dots or
slashes.

For example if your organization is `functional.swiss` good names would be
`functional` or  `functional_swiss`; however `functional.swiss` is not allowed
because of the dot.


Set up Microsoft OpenID Connect
-------------------------------

* https://aad.portal.azure.com/
* https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-protocols-oidc
* create an Application
* make sure: ID tokens (used for implicit and hybrid flows) is enabled
* set the redirect URL to `https://{YOUR_SERVER_NAME}/authenticators/ms-open-id/{NAME}/callback`.


### Outgoing Single Sign-Out Notifications

By our default configuration leihs will notify sign-outs from leihs to
Microsoft which will then perform a single sign-out from the whole platform.
Note: this applies only for the currently used session in the current browser.

To that end the authentication system within leihs uses the configuration
setting `external_sign_out_url`. And example value would be
`/authenticators/ms-open-id/{ID}/sign-out`.

This is how SSO on the Microsoft platform is intended to work. **Removing this
setting exposes some security risks**.



### Incomming Single Sign-Out Notification (Optional)

To sign out of leihs when the users signs out on Microsoft (or some other app
which has set up outgoing sign-out) set the "Front-channel logout URL" to the
following:

    https://{{YOUR_SERVER_NAME}}/authenticators/ms-open-id/{{NAME}}/sso-sign-out

Note: this is supported since 2023-12 and **requires leihs 7.3.0** or later.




Configuration
-------------

The service is started with a configuration file, e.g.

    aad-leihs-authenticator -c config.yml

The content of the configuration file looks like the following:


```
port: '3434'
send_login_hint: true
external_base_url: 'https://my.leihs.app'
name: 'functional'
tenant: 'REPLACE with ID (domain name should work too)'
client_id: 'REPLACE'
email_attribute: 'upn'

private_key: |
  -----BEGIN EC PRIVATE KEY-----
  …
  -----END EC PRIVATE KEY-----

public_key: |
  -----BEGIN PUBLIC KEY-----
  …
  -----END PUBLIC KEY-----

leihs_public_key: |
  -----BEGIN PUBLIC KEY-----
  …
  -----END PUBLIC KEY-----
```

Valid keys can be generated e.g. with

```
openssl ecparam -name prime256v1 -genkey -noout -out tmp/key.pem
openssl ec -in tmp/key.pem -pubout -out tmp/public.pem
```

### Disable `login_hint` if the UPN ist not equal to the E-Mail Address

Microsoft useses (in some cases) the `login_hint` to prefil parts of the
sign-in form.

The benefits of this are limited since saves the user to reenter this
information only if there is currently no SSO session.

Hovever, it there is an SSO session, and the login hint does not match the
information in the session Microsoft will interrupt the SSO flow and ask the
user for credetials.



Deployment
----------

The deployment uses [Ansible](https://docs.ansible.com/) and is expected to work
with a recent version of Ubuntu LTS. The following
variables must be supplied via the
[Ansible Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html):


Example:

    ansible-playbook -i HOSTSFILE -p deploy/deploy_play.yml -e 'name=NAME config_file=PATH_TO_CONFIG_FILE'



### Reverse Proxy

This service can be deployed on the same machine as leihs itself or on any
other internet host. The value of `adl_external_base_url` must be properly
adjusted as well as the corresponding value in leihs. In the case the service runs
on the leihs host `reverse_proxy_custom_config` should probably include something like:

    ProxyPass /authenticators/ms-open-id/ http://localhost:3434/authenticators/ms-open-id	nocanon retry=0

To start the deploy process invoke:

    ansible-playbook -i $INVENTORY_HOSTS_FILE -l $TARGET_MACHINE deploy/deploy_play.yml



Development
-----------

See `Gemfile` and `Gemfile.lock`.


Notes and Links
---------------


```
curl "https://login.microsoftonline.com/${TENNANT}/.well-known/openid-configuration"
```

### Single Sign-Out


https://github.com/MicrosoftDocs/azure-docs/issues/8779#issuecomment-391526377
