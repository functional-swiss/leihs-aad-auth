Leihs - MS Entra (Azure Active Directory) Open-ID Connect - Authentication System
=================================================================================

This project contains code and deployment recipes to use [Azure Active
Directory](https://azure.microsoft.com/de-de/services/active-directory/) via
[OpenID Connect](https://de.wikipedia.org/wiki/OpenID_Connect) as an external
authentication system for [leihs](https://github.com/leihs).


Quick Setup in Microsoft Entra
------------------------------

Within Entra https://entra.microsoft.com/ open your Leihs enterprise
application (or create a new one). Select **Authentication**. Click on **Add a
platform**. Select **Web**.


Set the **Redirect URI** (replace the hostname and name parameter _functional_) ,
e.g.:

    https://leihs.uni-muster.ch/authenticators/ms-open-id/functional/callback

Set the **Front-channel logout URL**, e.g.:

    https://leihs.uni-muster.ch/authenticators/ms-open-id/functional/sso-sign-out


Select the **checkbox** _ID tokens (used for implicit and hybrid flows)_.


Note the the **Application (client) ID** and provide it to functional (or see
confinguration below if you are maintainig this application yourself).






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

"session_state"=>"f791c9a6-a5cb-49dc-b15b-fe478b6b2470"

params seems to be equal to sid



Dec 01 20:48:12 uni-muster-leihs ruby[7546]: W, [2023-12-01T20:09:31.104425
#7546]  WARN -- : ["callback",
{"id_token"=>"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IlQxU3QtZExUdnlXUmd4Ql82NzZ1OGtyWFMtSSIsImtpZCI6IlQxU3QtZExUdnlXUmd4Ql82NzZ1OGtyWFMtSSJ9.eyJhdWQiOiJiOWVhNzdlYS1mZmU3LTQ4MWYtOGM3My04NGQ4YWM5MTBhMjgiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC85MzM5OTUxYS1hNTFkLTRmYzctYjNmYy0wNGMzODM1MjI1ZTgvIiwiaWF0IjoxNzAxNDU3NDcwLCJuYmYiOjE3MDE0NTc0NzAsImV4cCI6MTcwMTQ2MTM3MCwiYWlvIjoiQVZRQXEvOFZBQUFBOHZnTDUralVvNXl4VEt2YUp3c3RCMkhVWTNlQ3FtNTFzWXZZR2t0WDhoNk9uVXNiTmxsV3VFWmJYWTJFL2NYeGZKTFgwS2Iza04wTTR1Y3YwZHVvRldiYzl1NVhhSEVTSnAra0I5QTk0M0k9IiwiYW1yIjpbInB3ZCIsIm1mYSJdLCJmYW1pbHlfbmFtZSI6IlNjaGFuayIsImdpdmVuX25hbWUiOiJUaG9tYXMiLCJpcGFkZHIiOiIyMDAxOjE2MjA6OWE2OjA6ZWRlZjo2ODUzOjNjODA6ODQ4NyIsIm5hbWUiOiJUaG9tYXMgU2NoYW5rIiwibm9uY2UiOiJhMTQ0YjYzNi1hZmM0LTRhYjctODY4Ni1lN2FlNmU5MWE1MDgiLCJvaWQiOiJlY2Q4MmY5Yi0yNTQ5LTQ2NTQtOTI0NC0yMDEzZmYwNWIwNjciLCJyaCI6IjAuQVY0QUdwVTVreDJseDAtel9BVERnMUlsNk9wMzZybm5feDlJakhPRTJLeVJDaWhlQU5vLiIsInN1YiI6InBTMzVRWVR5QThHWTJDdDJKWHJhemwwRU9OaUFPekxVNTQydk1NQXQ4SW8iLCJ0aWQiOiI5MzM5OTUxYS1hNTFkLTRmYzctYjNmYy0wNGMzODM1MjI1ZTgiLCJ1bmlxdWVfbmFtZSI6InRob21hcy5zY2hhbmtAZnVuY3Rpb25hbC5zd2lzcyIsInVwbiI6InRob21hcy5zY2hhbmtAZnVuY3Rpb25hbC5zd2lzcyIsInV0aSI6Imo4M3QtTElLeDBHcmRodV9faEZzQUEiLCJ2ZXIiOiIxLjAifQ.luSgXw6Ppziz5SLh7SvUh4TUCXfoI_ePFGGBe235D70sc0_lTg0YJgT9JLy-ITjitAazsNhDzm6AapZSCsnKt_9u9CUHGZI2XVwLr4fznuSh_i6EPphkmPsn57JDDBO3t_mjZhZKXryCuynKLZVNQOPhmQFso3554VTNRvRaajh9I-lqzWA3jmadSpZIPz2Na7l37HyxamgmPbVlb1STmQh5SYhc6M8GHDnR_NlB7QGkEWkwTnKTCxO5u6XmRMUzrKznanE4LmbSp9EDngB2tOAEIGp-N9sScwz3hzxEZv3OL7GtFXEVWYIJeWJCxVuiWn2WYOo1o7OJJkOi5j4vkw",
"state"=>"eyJhbGciOiJFUzI1NiJ9.eyJzaWduX2luX3JlcXVlc3RfdG9rZW4iOiJleUpoYkdjaU9pSkZVekkxTmlKOS5leUpsYldGcGJDSTZJblJvYjIxaGN5NXpZMmhoYm10QVpuVnVZM1JwYjI1aGJDNXpkMmx6Y3lJc0lteHZaMmx1SWpwdWRXeHNMQ0p2Y21kZmFXUWlPbTUxYkd3c0ltVjRjQ0k2TVRjd01UUTFOemd5T0N3aWFXRjBJam94TnpBeE5EVTNOek00TENKelpYSjJaWEpmWW1GelpWOTFjbXdpT2lKb2RIUndjem92TDJ4bGFXaHpMblZ1YVMxdGRYTjBaWEl1WTJnaUxDSnlaWFIxY201ZmRHOGlPaUl2SWl3aWNHRjBhQ0k2SWk5emFXZHVMV2x1TDJWNGRHVnlibUZzTFdGMWRHaGxiblJwWTJGMGFXOXVMMjF6TFdaMWJtTjBhVzl1WVd3dmMybG5iaTFwYmlKOS5FQzRxS21wX2pORnlfSE5XV2IyNWJoTnlfSFBjOUMyMnIzYWlmT3pBbTJFQ2trQ0NiWm45OXlQbThmTEhHc0pyOGZ2VnNqc1NjSXVkOWZCTjdoZnUxZyJ9.ULq-NWfILljNz-cY6Zjq3lJ8ZK6RCh9g3okg197pK7rnvXkE54kep6IAODEv8rU5AV42ITzIVu1SM2xXiuRGWA",
"session_state"=>"f791c9a6-a5cb-49dc-b15b-fe478b6b2470",

```
WARN -- : ["sso-sign-out", {"sid"=>"f791c9a6-a5cb-49dc-b15b-fe478b6b2470", "name"=>"functional"}]
```


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
