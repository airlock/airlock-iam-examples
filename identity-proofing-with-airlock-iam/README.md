# Identity Proofing with Airlock IAM

This repository accompanies the blog post [Identity Proofing with Airlock IAM](https://www.airlock.com/en/insights/airlock-blog/business-blog/identity-proofing-with-airlock-iam). It provides an almost-ready-to-use configuration for Airlock IAM.


## Requirements

- Airlock IAM 8.3 or later with license module 'Selfservice'
- A database server somewhere in your infrastructure
- The corresponding JDBC jar installed in instances/common
- Database created with default schema, as available in the subsections of [this documentation page](https://docs.airlock.com/iam/latest/#data/1579527945303.html).
- API key from [PXL Vision](https://www.pxl-vision.com/)
- Lua 5.4 and Luarocks installed (previous versions may work but have not been tested)


## Setup instance

Clone this Git repository:
```console
git clone https://github.com/airlock/identity-proofing-with-airlock-iam
```

Next, create a new instance and copy the necessary files:
```console
iam init -i identity-proofing
for f in instance.properties sensitive-values.jceks sensitive-values.properties
do
    cp identity-proofing-with-airlock-iam/"${f}" "${IAM_CONFIG_ROOT}"/instances/identity-proofing/"${f}"
done
cp -r identity-proofing-with-airlock-iam/adminapp-texts "${IAM_CONFIG_ROOT}"/instances/identity-proofing/
cp -r identity-proofing-with-airlock-iam/loginapp-texts "${IAM_CONFIG_ROOT}"/instances/identity-proofing/
cp -r identity-proofing-with-airlock-iam/design "${IAM_CONFIG_ROOT}"/instances/identity-proofing/
```

If you want to use an instance name other than 'identity-proofing', you can. Be sure to adapt all the instructions here and later also update the target URI resolver plugin ("Logout and restart") in the IAM configuration.

Ensure this instance uses available TCP ports. Replace '8700' and '8800' with the correct values for your environment:
```console
sed -i -e 's,iam.web-server.http.port = .*,iam.web-server.http.port = 8700,' identity-proofing-with-airlock-iam/instance.properties
sed -i -e 's,iam.web-server.https.port = .*,iam.web-server.https.port = 8800,' identity-proofing-with-airlock-iam/instance.properties
```

Depending on your setup, you may need to create systemd or other service files before starting the instance.


## Prepare Lua libraries

The Lua scripts require two additional libraries. Install them as follows:
```console
luarocks install --tree "${IAM_CONFIG_ROOT}"/instances/identity-proofing/scripting/lua lua-curl
luarocks install --tree "${IAM_CONFIG_ROOT}"/instances/identity-proofing/scripting/lua base64
```

Additional information is available in the [Airlock IAM documentation](https://docs.airlock.com/iam/latest/#data/1709045025978.html).


## Load configuration

Open the Adminapp at URL: http://\<your-ip-or-fqdn\>:\<the-port-selected-above\>/identity-proofing-admin/

If you do this on a new server, add the license.

Next, open the Config Editor and use the Upload button to load the configuration 'iam-configuration-identity-proofing.xml' from this repository.


## Mandatory configuration changes

### Database setup

You will likely need to adapt the first three properties of the following plugin:

- MAIN SETTINGS / Data Sources / User Data Source / Database User Persister / SQL Data Source

The fourth property 'Password', can either be overwritten here or changed in the next section.


### External secrets

There are two secrets that need to be changed according to your environment. Use the External Secrets button to edit:

- **db password**: Unless the database password property has been overwritten with the database setup
- **pxl-ident api key**: You will receive this value when you register for an account with PXL Vision


## Optional configuration changes

### Registration Email Validation OTP

When registering a new user using identity proofing, an email address must be specified as the username. This address is validated by sending a one-time password (OTP) to it, which the user must enter.

For this demo, this has been simplified to avoid a dependency on a working email setup. Instead, a hard-coded OTP is used.

If you want to change this, update the following:

- Loginapp / Self-Registration / Default Flow/ Steps / Email verification
    - **OTP Generator**: Select 'Real OTP Generator' or define a new plugin


### Email setup

The demo project is setup to not require a working email infrastructure. However, if you want to send real emails, you need to replace the dummy email plugin here:

- Loginapp / Self-Registration / Default Flow/ Steps / Email verification
    - **Email Service**: add a new plugin of type SMTP Email Service and configure it according to your environment
- Loginapp / Applications and Authentication / Default Application / Authentication Flow / Steps / Email OTP Check
    - **Email Service**: select the same plugin you created above


### User profile data items

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

