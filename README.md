![Cryptocurrency Exchange Platform - OpenDAX](https://github.com/openware/meta/raw/main/images/github_opendax.png)

<h3 align="center">
<a href="https://www.openware.com/sdk">Guide</a> <span>&vert;</span>
<a href="https://www.openware.com/sdk/api.html">API Docs</a> <span>&vert;</span>
<a href="https://www.openware.com/">Consulting</a> <span>&vert;</span>
<a href="https://t.me/peatio">Community</a>
</h3>
<h6 align="center"><a href="https://github.com/openware/opendax">OpenDAX Trading Platform</a></h6>

# OpenDAX

OpenDAX is an open-source cloud-native multi-service platform for building a Blockchain/FinTech exchange of digital assets, cryptocurrency and security tokens.

## Getting started with OpenDAX

### 1. Get a VM

Minimum VM requirements for OpenDAX:
 * 8GB of RAM (12GB recommended)
 * 4 cores vCPU (6 cores recommended)
 * 300GB disk space (SSD recommended)

A VM from any cloud provider like DigitalOcean, Vultr, GCP, AWS as well as any dedicated server with Ubuntu, Debian or Centos would work

### 2. Prepare the VM

#### 2.1 Create Unix user
SSH using root user, then create new user for the application
```bash
useradd -g users -s `which bash` -m app
```

#### 2.2 Install Docker and docker compose

We highly recommend using docker and compose from docker.com install guide instead of the system provided package, which would most likely be deprecated.

Docker follow instruction here: [docker](https://docs.docker.com/install/)
Docker compose follow steps: [docker compose](https://docs.docker.com/compose/install/)

#### 2.3 Install ruby in user app

##### 2.3.1 Change user using
```bash
su - app
```

##### 2.3.2 Clone OpenDAX
```bash
git clone https://github.com/openware/opendax.git
```

##### 2.3.3 Install RVM
```bash
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
cd opendax
rvm install .
```

### 3. Bundle install dependencies

```bash
bundle install
rake -T # To see if ruby and lib works
```

Using `rake -T` you can see all available commands, and can create new ones in `lib/tasks`


### 4. Run everything

#### 4.1 Configure your domain
If using a VM you can point your domain name to the VM ip address before this stage.
Recommended if you enabled SSL, for local development edit the `/etc/hosts`


Insert in file `/etc/hosts`
```
0.0.0.0 www.app.local
```

#### 4.2 Render all config files

Simply run `rake render:config`

#### 4.3 Set up Vault

    Note: Everything is persisted on the local filesystem, thus API keys and 2FA tokens are preserved between restarts. However, Vault needs to be unsealed after every stop/restart.

To set up Vault, go through the following steps:
  - `docker-compose up -d vault`
  - `docker-compose exec vault sh`
  - `vault operator init`
  - Save the output to a file in a secure place
  - Unlock Vault with three different unlock keys - `vault operator unseal *unseal_key*`
  - `vault login *root_token*`
  - `vault secrets enable totp`
  - `vault secrets enable transit`
  - `vault secrets disable secret`
  - `vault secrets enable -path=secret -version=1 kv`

Add the Vault root token to `config/app.yml`, render the configs and start the `app` services.
Afterwards, Vault should be fully configured and ready to work with Peatio and Barong.

#### 4.4 Bring everything up

```bash
rake service:all
```

You can login on `www.app.local` with the following default users from seeds.yaml
```
Seeded users:
Email: admin@barong.io, password: 0lDHd9ufs9t@
Email: john@barong.io, password: Am8icnzEI3d!
```

### [Optional] KYCAID

In order to  accelerate customer interaction, reduce risks and simplify business processes you can use KYC Verification Service from KYCaid.
KYC goal is to prevent fraud and to decline users that don’t fulfill certain standards of credibility.
To learn more about KYCaid and pricing you can visit their website - [kycaid.com](https://www.kycaid.com/)

#### How to configure KYCAID on the platform?

KYCAID is already integrated into our stack, to use it you'd need to create an account on [kycaid.com](https://www.kycaid.com/), and set up authentification creds there.

After that all you have to do is to change several lines in `config/app.yml`:

```yaml
kyc:
  provider: kycaid
  authorization_token: changeme             # your production API token from the 'Settings' section of kycaid.com
  sandbox_mode: true                        # 'true' for test environments - documents will be verified/rejected automatically, without payment for verification
  api_endpoint: https://api.kycaid.com/
```

##### Additional settings for KYCAID

* Be sure to check `BARONG_REQUIRED_DOCS_EXPIRE` ENV value inside `config/barong.env` to be `false` if you want to include `address` verification in your KYC process. You can set it to `true` if you need the document check only.
* Check if  you have the correct list of `document_types` in the `config/barong/barong.yml` file:
  - Passport
  - Identity card
  - Driver license
  - Address
* Frontend KYC steps can be configured in `templates/config/frontend/env.js.erb` via the `kycSteps` field
* Tower KYC labels can be configured in `templates/config/frontend/tower.js.erb` via the `labelSwitcher` field

## Usage

### Initial configuration

All the OpenDAX deployment files have their confguration stored in `config/app.yml`.

#### app.yml

The following table lists the configurable parameters of the config/app.yml configuration file and its default values.

Parameter | Description | Default
--- | --- | ---
`app.name` | global application name | `"OpenDax"`
`app.domain` | base domain name | `app.local`
`subdomain` | subdomain | `www`
`render_protect` | enable read-only mode for rendered files | `false`
`csrfEnabled` | enable CSRF protection on Barong | `false`
`ssl.enabled` | enable SSL certificate generation | `false`
`ssl.email` | email address used for SSL certificate issuing | `"support@example.com"`
`images` | Docker image tags per component
`vendor.frontend` | optional Git URL for a development frontend repo | `git@github.com:openware/baseapp.git`
`kyc.provider` |  KYC provider, can be `kycaid` or `local` | `kycaid`
`kyc.authorization_token` |  optional API token for KYCAID use | `changeme`
`kyc.sandbox` |  enable KYCAID test mode  | `true`
`kyc.api_endpoint` |  API endpoint for KYCAID | `https://api.kycaid.com/`
`vault.token` | Vault authentication token | `changeme `
`database.adapter`| database adapter kind either `mysql` or `postgresql` |`mysql`
`database.host` | database host name | `db`
`database.port` | database port | `3306 `
`database.user` | database username | `root`
`database.password` | database root password | `changeme`
`storage.provider` | object storage provider | `"Google"`
`storage.bucketname` | storage bucket name | `"opendax-barong-docs-bucket"`
`storage.endpoint` | S3-compatible storage API endpoint | `"https://fra1.digitaloceanspaces.com"`
`storage.region` | storage region | `"fra1"`
`storage.signatureVersion` | S3-compatible storage API signature version(2 or 4) | `"fra1"`
`storage.secretkey`, `storage.accesskey` | storage access keys | `"changeme"`
`twilio` | [Twilio](https://www.twilio.com/) SMS provider configs
`gaTrackerKey` | [Google Analytics](https://analytics.google.com/) tracker key inserted into the frontend app
`smtp` | SMTP configs used for sending platform emails
`captcha` | captcha configuration([Recaptcha](https://www.google.com/recaptcha) or [Geetest](https://www.geetest.com))
`wallets` | configs for wallets seeded during the initial deployment of Peatio
`parity` | Parity cryptonode configuration
`bitcoind` | Bitcoind cryptonode configuration
`litecoind` | Litecoind cryptonode configuration
`terraform.credentials` | local path to a GCP service account JSON key | `"~/safe/opendax.json"`
`terraform.project` | GCP project name | `"example-opendax"`

### utils.yml

The following table lists configurable parameters of the `config/utils.yml` file:

Parameter | Description | Default
--- | --- | ---
images | Docker image tags per component |
superset | Superset BI tool configs |
arke | Arke liquidity bot configs |

Once you're done with the configuration, render the files using `rake render:config`. You can easily apply your changes at any time by running this command.

    Note: be sure to append all the subdomains based on app.domain to your
    /etc/hosts file if you're running OpenDax locally

### Bringing up the stack

The OpenDAX stack can be brought up using two ways:

1. Bootstrap all the components at once using `rake service:all[start]`
2. Start every component one-by-one using `rake service:*component*[start]`

The components included in the stack are:

- `proxy` - [Traefik](https://traefik.io/), a robust cloud-native edge router/reverse proxy written in Go
- `backend` - [Vault](https://www.vaultproject.io), [MySQL](https://www.mysql.com), [Redis](https://redis.io) and [RabbitMQ](https://www.rabbitmq.com) grouped together
- `cryptonodes` - cryptocurrency nodes such as [parity](https://github.com/paritytech/parity-ethereum) **[Optional]**
- `daemons` - Peatio and Ranger daemons **[Optional]**
- `setup` - setup hooks for Peatio and Barong to run before the application starts (DB migration etc.)
- `app` - Peatio is the [crypto exchange software](https://www.openware.com/), [Barong](https://github.com/openware/barong) and the [Ambassador](https://www.getambassador.io) API gateway
- `frontend` - the frontend application located at `vendor/frontend`
- `tower` - the Tower admin panel application located at `vendor/tower`
- `monitoring` - [cAdvisor](https://github.com/google/cadvisor) and [Node Exporter](https://github.com/prometheus/node_exporter) monitoring tools **[Optional]**

For example, to start the `backend` services, you'll simply need to run `rake service:backend[start]`

    Note: all the components marked as [Optional] need to be installed using
    rake service:*component*[start] explicitly

Go ahead and try your own OpenDAX exchange deployment!

### Stopping and restarting components

Any component from the stack can be easily stopped or restarted using `rake service:*component*[stop]` and `rake service:*component*[restart]`.

For example, `rake service:frontend[stop]` would stop the frontend application container and `rake service:proxy[restart]` would completely restart the reverse proxy container.

# Managing component deployments

Each component has a config file (ex. `config/frontend/tower.js`) and a compose file (ex. `compose/frontend.yaml`).

All config files are mounted into respective component container, except from `config/app.yml` - this file contains all the neccessary configuration of opendax deployment

Compose files contain component images, environment configuration etc.

These files get rendered from their respective templates that are located under `templates` directory.

## How to update component image?

Modify `config/app.yml` with correct image and run `rake service:all`
This will rerender all the files from `templates` directory and restart all the running services.

Alternitavely you can update the following files:
  * `config/app.yml`
  * `templates/compose/*component*.yml`
  * `compose/*component*.yml`
And run `rake service:component[start]`

## How to update component config?

Modify `config/*component*/*config*` and run `rake service:component[start]`,
if you want the changes to be persistent, you also need to update `templates/config/*components*/*config*`

#### Render compose file
```
# Delete all generated files
git clean -fdx

# Re-generate config from config/app.yml values
rake render:config

# Restart the container you need to reload config
docker-compose up frontend -Vd
```

#### Clone the vendors and start
```
source ./bin/set-env.sh
rake vendor:clone
docker-compose -f compose/vendor.yaml up -d
```

## Terraform Infrastructure as Code Provisioning

You can easily deploy OpenDAX from scratch on Google Cloud Platform using [Terraform](https://www.terraform.io)!

To do this, just follow these simple steps:
  - Fill `app.yml` with correct values
  - Run `rake terraform:apply`
  - Access your VM from the GCP Cloud Console

To destroy the provisioned infrastructure, just run `rake terraform:destroy`

## Installer tool

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/openware/opendax/master/bin/install)"
```

## Happy trading with OpenDAX!
If you have any comments, feedback and suggestions, we are happy to hear from you here at GitHub or here: [crypto exchange software](https://www.openware.com/)
