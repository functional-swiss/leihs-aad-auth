Leihs - Azure Active Directory Open-ID Connect - Authentication System
=======================================================================

This project contains code and deployment recipes to use [Azure Active
Directory](https://azure.microsoft.com/de-de/services/active-directory/) via
[OpenID Connect](https://de.wikipedia.org/wiki/OpenID_Connect) as an external
authentication system for [leihs](https://github.com/leihs).

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


Configuration
-------------

The service is started with a configuration file, e.g.

    aad-leihs-authenticator -c /config.yml

The content of the configuration file looks like the following:


```
port: '3434'
external_base_url: 'https://hkb.leihs.app'
name: 'functional'
tenant: '9339951a-a51d-4fc7-b3fc-04c3835225e8'
client_id: 'b9ea77ea-ffe7-481f-8c73-84d8ac910a28'
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



