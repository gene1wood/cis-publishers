# cis-publishers

This is the repo containing the CIS publishers, or pointers to where they are contained.

* Auth0 - Written in Javascript, and contained in the [Auth0 rules](https://github.com/mozilla-iam/auth0-deploy/blob/master/rules/activate-new-users-in-CIS.js)
  - Note that this is the only publisher that should be creating profiles
* DinoPark - Contained [elsewhere](https://github.com/mozilla-iam/dino-park) in mozilla-iam
* HRIS Publisher - [Still in CIS](https://github.com/mozilla-iam/cis/blob/master/python-modules/cis_publisher/cis_publisher/hris.py), should be moved to using this code as it will be much faster and reliable.
* LDAP Publisher - Here, in cis_publishers/ldap

## Environmental variables

* `DRY_RUN` - run as dry run when set to True
* `LDAP_CACHE_S3_BUCKET` - bucket containing the LDAP dump (should be cache.ldap.sso.mozilla.com)
* `LDAP_CACHE_S3_KEY` - file name in S3 of the LDAP dump (should be ldap_users.json.xz)
* `LDAP_CACHE_FILENAME` - in lieu of the two above, you can run it against local LDAP cache
* `OAUTH_CLIENT_ID` - client ID in Auth0 to get Bearer token to Person/Change API (this is contained in SSM when run as Lambda)
* `OAUTH_CLIENT_SECRET` - client secret in Auth0 to get Bearer token to Person/Change API (this is contained in SSM when run as Lambda)
* `PUBLISHER_NAME` - the name of the publisher (e.g. ldap, cis, hris, etc.)
* `PUBLISHER_SIGNING_KEY` - the JSON of the publisher's signing key (this is contained in SSM when run as Lambda)

## Signing key

The signing key was generated by using the RSA private key in SSM and running it through the following
Javascript. Note that this requires uuid and [https://github.com/OADA/rsa-pem-to-jwk](rsa-pem-to-jwk):

```javascript
const fs = require("fs");
const rsaPemToJwk = require("rsa-pem-to-jwk");
const { v4: uuidv4 } = require("uuid");

const jwk = Object.assign(rsaPemToJwk(fs.readFileSync("signing_key.pem"), {use: "sig"}, "private"), {kid: uuidv4()});

JSON.stringify(jwk);
```

## Running locally

You can run the LDAP publisher locally to verify that it works. Make sure to set DRY_RUN if you want it to dry run.

Make sure to run MAWS first, so you have access to the SSM parameters in `mozilla-iam`.

```bash
$ PYTHONPATH="." serverless invoke local -f ldap --stage production
```

## Use as a library

The cis_publishers.common code is intended to be used as a library for whatever you like. For example, assuming
you have `OAUTH_CLIENT_ID` and `OAUTH_CLIENT_SECRET` set, you can read in a CIS profile like so:

```python
>>> from cis_publishers.common import Profile
>>> april = Profile(email="apking@mozilla.com")
>>> april
{'access_information': {'access_provider': None, 'hris': {'egencia_pos_country': 'US', 'employee_id': '123456', 'managers_primary_work_email': '...'}
```

## TODO

There are some things left to do before this can go to production:

* Remove forced dry run status
* Remove LDAP prefix code (and inclusion from documents, etc.)

Here are things that would be great:

* Tests, ugh I'm so bad
* Adding the ability to look at the last run date and only process users that have actually changed in LDAP