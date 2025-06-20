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
    cp identity-proofing-with-airlock-iam/full-config/"${f}" "${IAM_CONFIG_ROOT}"/instances/identity-proofing/"${f}"
done
cp -r identity-proofing-with-airlock-iam/full-config/adminapp-texts "${IAM_CONFIG_ROOT}"/instances/identity-proofing/
cp -r identity-proofing-with-airlock-iam/full-config/loginapp-texts "${IAM_CONFIG_ROOT}"/instances/identity-proofing/
cp -r identity-proofing-with-airlock-iam/full-config/design "${IAM_CONFIG_ROOT}"/instances/identity-proofing/
```

If you want to use an instance name other than 'identity-proofing', you can. Be sure to adapt all the instructions here and later also update the target URI resolver plugin ("Logout and restart") in the IAM configuration.

Ensure this instance uses available TCP ports. Replace '8700' and '8800' with the correct values for your environment:
```console
sed -i -e 's,iam.web-server.http.port = .*,iam.web-server.http.port = 8700,' "${IAM_CONFIG_ROOT}"/instances/identity-proofing/instance.properties
sed -i -e 's,iam.web-server.https.port = .*,iam.web-server.https.port = 8800,' "${IAM_CONFIG_ROOT}"/instances/identity-proofing/instance.properties
```

Depending on your setup, you may need to create systemd or other service files before starting the instance.


## Load configuration

Open the Adminapp at URL: http://\<your-ip-or-fqdn\>:\<the-port-selected-above\>/identity-proofing-admin/

If you do this on a new server, add the license.

Next, open the Config Editor and use the Upload button to load the configuration 'iam-configuration-identity-proofing.xml' from this repository.


## Mandatory configuration changes

### Database setup

You will likely need to adapt the first three properties of the following plugin:

- MAIN SETTINGS / Data Sources / User Data Source / Database User Persister / SQL Data Source

The fourth property 'Password', can either be overwritten here or changed in the next section.


### External secret

Unless defined directly with the database plugin, create the following external secret. Use the External Secrets button to edit:

- **db password**: Unless the database password property has been overwritten with the database setup

