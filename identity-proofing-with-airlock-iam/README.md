# Identity Proofing with Airlock IAM

There are two options to get to a working Airlock IAM configuration supporting identity proofing:

- [Full Config](/full-config) - The "just add water" (sorry, database) method.
- [Config Snippet](/snippet) - Easily enrich existing project without manually re-doing big parts of the configuration.


## Additional steps required for both methods

### Prepare Lua libraries

The Lua scripts require two additional libraries. Install them as follows:
```console
luarocks install --tree "${IAM_CONFIG_ROOT}"/instances/identity-proofing/scripting/lua lua-curl
luarocks install --tree "${IAM_CONFIG_ROOT}"/instances/identity-proofing/scripting/lua base64
```

Additional information is available in the [Airlock IAM documentation](https://docs.airlock.com/iam/latest/#data/1709045025978.html).


### Mandatory configuration changes

#### External secrets

Accessing the PXL Vision identity proofing service requires an API key which should be defined as external secret:

- **pxl-ident api key**: You will receive this value when you register for an account with PXL Vision


### Optional configuration changes

#### Registration Email Validation OTP

When registering a new user using identity proofing, an email address must be specified as the username. This address is validated by sending a one-time password (OTP) to it, which the user must enter.

For this demo, this has been simplified to avoid a dependency on a working email setup. Instead, a hard-coded OTP is used.

If you want to change this, update the following:

- Loginapp / Self-Registration / (Flow) / Steps / Email verification
    - **OTP Generator**: Select an existing or define a new plugin


#### Email setup

The demo project is setup to not require a working email infrastructure. However, if you want to send real emails, you need to replace the dummy email plugin here:

- Loginapp / Self-Registration / (Flow) / Steps / Email verification
    - **Email Service**: add a new plugin of type SMTP Email Service and configure it according to your environment
- Loginapp / Applications and Authentication / Default Application / Authentication Flow / Steps / Email OTP Check
    - **Email Service**: select the same plugin you created above


#### User profile data items

As we are using the standard database schema, there are no dedicated fields for some data retrieved from the identity proofing service. Some of the existing fields are "misused" for them:

- **document type**: stored in field 'state'
- **document id**: stored in field 'company'

If you add dedicated (string) fields to the database, you need to update the mapping in the following plugins:

- MAIN SETTINGS / Data Sources / User Data Source / Database User Persister / String Context Data Item docid
    - **Database Column Name**: specify database field

- MAIN SETTINGS / Data Sources / User Data Source / Database User Persister / String Context Data Item doctype
    - **Database Column Name**: specify database field


# Demo instructions

## Self-Registration

Once the instance has been setup correctly, use the following information to demonstrate the identity proofing feature:

- URL: http://\<your-ip-or-fqdn\>:\<port-as-per-instance.properties\>/identity-proofing-login/
- Select "sign up here"
- Email verification security code: 1234
- Use a smartphone to scan the QR code to initiate the identity proofing workflow
- After completing the workflow, don't forget to click "Accept" on the browser page


## Authentication with new user

After a successful registration, the functionality can be "proven" by logging in:

- URL: http://\<your-ip-or-fqdn\>:\<port-as-per-instance.properties\>/identity-proofing-login/
- Enter same email address as used for registration
- Unlike registration, a real OTP is used here. However, by default, no email is sent to the entered address. You can extract the security code from the logs:
    ```console
    grep "Your security code for the user" "${IAM_CONFIG_ROOT}"/instances/identity-proofing/logs/loginapp.log
    ```
- As there is no application integrated in this demo, a simple message is shown to confirm successful authentication. When continuing, the user is automatically logged out.

