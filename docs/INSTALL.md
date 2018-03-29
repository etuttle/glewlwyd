# Installation

1. [Debian-ish packages](#debian-ish-packages)
2. [Pre-compiled packages](#pre-compiled-packages)
3. [Docker](#docker)
4. [Manual install from Github](#manual-install-from-github)
   * [CMake](#cmake)
   * [Good ol' Makefile](#good-ol-makefile)
5. [Configuration](#configuration)
   * [SSL/TLS](#ssltls)
   * [login and grant URLs](#login-and-grant-urls)
   * [Reset password](#reset-password)
   * [Digest Algorithm](#digest-algorithm)
   * [Database back-end initialisation](#database-back-end-initialisation)
   * [Authentication back-end configuration](#authentication-back-end-configuration)
   * [JWT configuration](#jwt-configuration)
   * [Install as a service](#install-as-a-service)
6. [Run Glewlwyd](#run-glewlwyd)
7. [Front-end application](#front-end-application)

## Debian-ish packages

[![Packaging status](https://repology.org/badge/vertical-allrepos/glewlwyd.svg)](https://repology.org/metapackage/glewlwyd)

Glewlwyd is now available in Debian Buster (testing) and some Debian based distributions. To install it on your device, use the following command as root:

```shell
# apt install glewlwyd
```

Then, you must initialise your database, setup your jwt key and setup your `glewlwyd.conf` file

### Pre-compiled packages

You can install Glewlwyd with a pre-compiled package available in the [release pages](https://github.com/babelouest/glewlwyd/releases/latest/). The package files `glewlwyd-full_*` contain the package libraries of `orcania`, `yder`, `ulfius` and `hoel` pre-compiled for `glewlwyd`, plus `glewlwyd` package. To install a pre-compiled package, you need to have installed the following libraries:

```
libmicrohttpd
libjansson
libcurl-gnutls
uuid
libldap2
libmariadbclient
libsqlite3
libconfig
libgnutls
libssl
```

For example, to install Glewlwyd with the `glewlwyd-full_1.3.2_Debian_stretch_x86_64.tar.gz` package downloaded on the `releases` page, you must execute the following commands:

```shell
$ sudo apt install -y autoconf automake make pkg-config libjansson-dev libssl-dev libcurl3 libconfig9 libcurl3-gnutls libgnutls30 libgcrypt20 libmicrohttpd12 libsqlite3-0 libmariadbclient18 libtool uuid
$ wget https://github.com/benmcollins/libjwt/archive/v1.9.tar.gz
$ tar -zxvf v1.9.tar.gz
$ cd libjwt-1.9
$ autoreconf -i
$ ./configure
$ make && sudo make install
$ wget https://github.com/babelouest/glewlwyd/releases/download/v1.3.2/glewlwyd-full_1.3.2_Debian_stretch_x86_64.tar.gz
$ tar xf hoel-dev-full_1.4.0_Debian_stretch_x86_64.tar.gz
$ sudo dpkg -i liborcania_1.2.0_Debian_stretch_x86_64.deb
$ sudo dpkg -i libyder_1.2.0_Debian_stretch_x86_64.deb
$ sudo dpkg -i libhoel_1.4.0_Debian_stretch_x86_64.deb
$ sudo dpkg -i libulfius_2.3.0_Debian_stretch_x86_64.deb
$ sudo dpkg -i glewlwyd_1.3.2_Debian_stretch_x86_64.deb
```

If there's no package available for your distribution, you can recompile it manually using `CMake` or `Makefile`.

## Docker

[Rafael](https://github.com/rafaelhdr/) has made [docker images](https://github.com/rafaelhdr/glewlwyd-oauth2-server) for Glewlwyd, Kudos to him!

### Quickstart SQLite3

> Quickstart image is not supposed to be used in production environments. It is for testing only.
> The JWT configuration uses sha algorithm with the secret `secret` hard coded. So the tokens generated are **_NOT SAFE_**!

To run the quickstart image, you can execute the following command.

```shell
$ docker run --rm -it -p 4593:4593 rafaelhdr/glewlwyd-oauth2-server:2.0-quickstart
```

After creating the Quickstart, you can connect with admin login (username: *admin*, password: *password*) at [http://localhost:4593](http://localhost:4593).

If you want to be able to reuse the quickstart instance by having the SQLite3 database file outside of the docker image, you must mount a volume to the docker image path `/var/cache/glewlwyd/`.

```shell
$ docker run --rm -it -p 4593:4593 -v /path/to/cache:/var/cache/glewlwyd rafaelhdr/glewlwyd-oauth2-server:2.0-quickstart
```

### Create your own docker image

You can also use Docker base images to create one with your own settings (database back-end, private/public jwt key, etc). Check out the [documentation](https://github.com/rafaelhdr/glewlwyd-oauth2-server#installation) for more information.

## Manual install from Github

You must install the following libraries including their header files:

```
libmicrohttpd
libjansson
libcurl-gnutls
uuid
libldap2
libmariadbclient
libsqlite3
libconfig
libgnutls
libssl
```

On a Debian based distribution (Debian, Ubuntu, Raspbian, etc.), you can install those dependencies using the following command:

```shell
$ sudo apt-get install libmicrohttpd-dev libjansson-dev libcurl4-gnutls-dev uuid-dev libldap2-dev libmariadbclient-dev libsqlite3-dev libconfig-dev libgnutls28-dev libssl-dev
```

### Libssl vs libgnutls

Both libraries are mentioned required, but you can get rid of libssl if you install `libjwt` with the option `--without-openssl`. `gnutls` 3.5.8 minimum is required. For this documentation to be compatible with most Linux distributions (at least the one I use), I don't remove libssl from the required libraries yet.

### Libmicrohttpd bug on POST parameters

With Libmicrohttpd 0.9.37 and older version, there is a bug when parsing `application/x-www-form-urlencoded` parameters. This is fixed in later version, from the 0.9.38, so if your Libmicrohttpd version is older than that, I suggest getting a newer version of [libmicrohttpd](https://www.gnu.org/software/libmicrohttpd/).

### Build Glewlwyd and its dependencies

#### CMake

Download and install libjwt, then download Glewlwyd from GitHub, then use the CMake script to build the application:

```shell
# Install libjwt
# libtool and autoconf may be required, install them with 'sudo apt-get install libtool autoconf'
$ git clone https://github.com/benmcollins/libjwt.git
$ cd libjwt/
$ autoreconf -i
$ ./configure # use ./configure --without-openssl to use gnutls instead, you must have gnutls 3.5.8 minimum
$ make
$ sudo make install

# Install Glewlwyd
$ git clone https://github.com/babelouest/glewlwyd.git
$ mkdir glewlwyd/build
$ cd glewlwyd/build
$ cmake ..
$ make 
$ sudo make install
```

### Good ol' Makefile

Download Glewlwyd and its dependencies hosted on github, compile and install.

```shell
# Install libjwt
# libtool and autoconf may be required, install them with 'sudo apt-get install libtool autoconf'
$ git clone https://github.com/benmcollins/libjwt.git
$ cd libjwt/
$ autoreconf -i
$ ./configure # use ./configure --without-openssl to use gnutls instead, you must have gnutls 3.5.8 minimum
$ make
$ sudo make install

# Install Orcania
$ git clone https://github.com/babelouest/orcania.git
$ cd orcania/
$ make
$ sudo make install

# Install Yder
$ git clone https://github.com/babelouest/yder.git
$ cd yder/src/
$ make
$ sudo make install

# Install Ulfius
$ git clone https://github.com/babelouest/ulfius.git
$ cd ulfius/src/
$ make
$ sudo make install

# Install Hoel
$ git clone https://github.com/babelouest/hoel.git
$ cd hoel/src/
$ make DISABLE_POSTGRESQL=1
$ sudo make install

# Install Glewlwyd
$ git clone https://github.com/babelouest/glewlwyd.git
$ cd glewlwyd/src/
$ make 
$ sudo make install
```

## Configuration

Copy `glewlwyd.conf.sample` to `glewlwyd.conf`, edit the file `glewlwyd.conf` with your own settings.

### SSL/TLS

OAuth 2 specifies that a secured connection is mandatory, via SSL or TLS, to avoid data and token to be stolen, or Man-In-The-Middle attacks. Glewlwyd supports starting a secure connection with a private/public key certificate, but it also can be with a classic non-secure HTTP connection, and be available to users behind a HTTPS proxy for example. Glewlwyd won't check that you use it in a secure connection.

### login and grant URLs

Update the entries `login_url` and `grant_url` in the configuration file to fit your installation, for example:

```
login_url="http://localhost:4593/app/login.html?"
grant_url="http://localhost:4593/app/grant.html?"
```

### Reset password

If you enable the reset password setting in the configuration file, you must specify a SMTP server that can be used to send the e-mails, and copy the file `reset.eml` to the specified location.

```
# Reset password configuration
reset_password=true # optional, default false
reset_password_config = 
{
# SMTP parameters
  smtp_host="localhost"         # mandatory
  smtp_port=25                  # optional, default 25
  smtp_use_tls=false            # optional, default false
  smtp_verify_certificate=false # optional, default false
#  smtp_user="user"             # optional, default no value
#  smtp_password="password"     # optional, default no value
  
  token_expiration=604800                                                                     # in seconds, optional, default 7 days
  email_from="glewlwyd@example.org"                                                           # mandatory
  email_subject="Glewlwyd password reset"                                                     # mandatory
  email_template="reset.eml"                                                                  # mandatory
  page_url_prefix="https://example.org/glewlwyd/app/reset.html?user=$USERNAME&code=$TOKEN"    # mandatory
}
```

### Digest algorithm

Specify in the config file the parameter `hash_algorithm` to store passwords with sqlite3 back-end, and token digests.

Algorithms available are SHA1, SHA256, SHA512, MD5.

### Database back-end initialisation

For a Mariadb/Mysql database, you must create a database or use an existing one first, example:

```sql
-- Create database and user
CREATE DATABASE `glewlwyd`;
GRANT ALL PRIVILEGES ON glewlwyd.* TO 'glewlwyd'@'%' identified BY 'glewlwyd';
GRANT ALL PRIVILEGES ON glewlwyd.* TO 'glewlwyd'@'localhost' identified BY 'glewlwyd';
FLUSH PRIVILEGES;
```

Then, use the script that fit your database back-end and Digest algorithm in the [database](database) folder:

- `database/init-mariadb.sql`
- `database/init-sqlite3-md5.sql`
- `database/init-sqlite3-sha.sql`
- `database/init-sqlite3-sha256.sql`
- `database/init-sqlite3-sha512.sql`

For example, initialise a MariaDB database:

```shell
$ mysql
mysql> CREATE DATABASE `glewlwyd`;
mysql> GRANT ALL PRIVILEGES ON glewlwyd.* TO 'glewlwyd'@'%' identified BY 'glewlwyd';
mysql> FLUSH PRIVILEGES;
mysql> USE glewlwyd
mysql> SOURCE database/init-mariadb.sql
```

Initialise a SQLite3 database:

```shell
$ sqlite3 /var/cache/glewlwyd/glewlwyd.db < database/init-sqlite3-sha256.sql
```

#### Security warning!

Those scripts create a valid database that allow to use glewlwyd but to avoid potential security issues, you must make the following changes before opening Glewlwyd API to the wild web:
- Change the admin password when you connect to the application
- Change the redirect_uri value for the client `g_admin` with an absolute redirect_uri value, e.g. `http://localhost:4593/app/`, then uncomment the corresponding line in [glewlwyd.react.js](https://github.com/babelouest/glewlwyd/blob/master/webapp/app/glewlwyd.react.js#L47) and set with your value.
- Change the values `login_url` and `grant_url` in your configuration file for absolute URLs, e.g. `http://localhost:4593/app/login.html?`

#### Admin scope value

If you want to use a different name for admin scope (default is `g_admin`), you must update the init script with your own `gs_name` value before running it, change the last line which reads:

```sql
INSERT INTO g_scope (gs_name, gs_description) VALUES ('g_admin', 'Glewlwyd admin scope');
```

### Authentication back-end configuration

For the authentication back-end, you can use a LDAP server, your database or a remote HTTP service using Basic Auth. If you use multiple back-ends like database and LDAP back-ends, then on an authentication process, the user or the client will be tested in the LDAP first, then in the database if not found, then eventually in the HTTP Basic Auth.

The HTTP Basic Auth back-end is to be used only without scope (config file `use_scope=false`) by design.

#### Database authentication

To use Database authentication, you must set the config parameter `database_auth` to `true`. Then your database can be used to store users and clients.

#### LDAP authentication

To use LDAP authentication, you must set the config parameter `ldap_auth` to `true`. Then you must setup the LDAP access parameters:

```
   uri                      = "ldap://localhost"               # uri of the LDAP server
   bind_dn                  = "cn=operator,dc=example,dc=org"  # bind_dn used to connect to the LDAP
   bind_passwd              = "password"                       # password used to connect to the LDAP
   base_search_user         = "ou=user,dc=example,dc=org"      # LDAP base search for users
   base_search_client       = "ou=client,dc=example,dc=org"    # LDAP base search for clients
```

You can use a LDAP server to store users and/or clients. If you want to store users only, you must use database authentication as well in order to be able to store clients, and vice-versa.

The LDAP read parameters are required to specify the LDAP properties to map.

```
# Users read parameters
   filter_user_read               = "objectClass=inetOrgPerson" # filter used to search users
   login_property_user_read       = "cn"                        # property used to map login
   scope_property_user_read       = "o"                         # property used to map scope
   email_property_user_read       = "mail"                      # property used to map e-mail
   name_property_user_read        = "sn"                        # property used to map display name
   additional_property_value_read = "memberOf"                  # will fill the jwt property `group` with the content of the LDAP property `memberOf`, optional, leave empty if no use
# Clients read parameters
   filter_client_read                = "objectClass=inetOrgPerson" # filter used to search client
   client_id_property_client_read    = "cn"                        # property used to map client_id
   scope_property_client_read        = "o"                         # property used to map scope
   name_property_client_read         = "sn"                        # property used to map client name
   description_property_client_read  = "description"               # property used to map client description
   redirect_uri_property_client_read = "labeledURI"                # property used to map redirect uris
   confidential_property_client_read = "employeeType"              # property used to map client confidential flag
```

The LDAP write parameters are mandatory if you want to be able to modify the users and/or the clients stored in the LDAP back-end. When a property specifies `Multiple values separated by a comma`, this means that the written value can be duplicated in multiple properties if necessary, for example having the display name in the LDAP properties `sn` and `display_name`.

```
# Users write parameters
   ldap_user_write                 = true
   rdn_property_user_write         = "cn"           # Single value
   login_property_user_write       = "cn"           # Multiple values separated by a comma
   scope_property_user_write       = "o"            # Multiple values separated by a comma
   email_property_user_write       = "mail"         # Multiple values separated by a comma
   additional_property_value_write = "memberOf"     # Multiple values separated by a comma
   name_property_user_write        = "sn"          
   password_property_user_write    = "userPassword" # Single value
   password_algorithm_user_write   = "SSHA"         # Single value, values available are SSHA, SHA, SMD5, MD5 or PLAIN
   object_class_user_write         = "top,person,organizationalPerson,inetOrgPerson" # Multiple values separated by a comma
# Clients write parameters
   ldap_client_write                  = true
   rdn_property_client_write          = "cn"          # Single value
   client_id_property_client_write    = "cn"          # Multiple values separated by a comma
   scope_property_client_write        = "o"           # Multiple values separated by a comma
   name_property_client_write         = "sn"          # Multiple values separated by a comma
   description_property_client_write  = "description" # Multiple values separated by a comma
   redirect_uri_property_client_write = "labeledURI"  # Multiple values separated by a comma
   password_property_client_write     = "userPassword"# Single value
   confidential_property_client_write = "employeeType"
   password_algorithm_client_write    = "SSHA"        # Single value, values available are SSHA, SHA, SMD5, MD5 or PLAIN
   object_class_client_write          = "top,person,organizationalPerson,inetOrgPerson" # Multiple values separated by a comma
```

### Administrator user

An administrator must be present in the back-end to use the application (manage scopes, users, clients, resources, authorization types).

An administrator in the LDAP back-end is a user who has the `admin_scope` (default `g_admin`) in its scope list.

Scope list is stored in the config file parameter `scope_property_user_read` (`o` by default).

### JWT configuration

You can choose between SHA (HS256, HS384, HS512), RSA (RS256, RS384, RS512) and ECDSA (ES256, ES384, ES512) algorithms to sign the tokens. Note that if you use SHA, you will need to share the `sha_secret` value with the resource providers and keep it safe in all places. If you use RSA or ECDSA algorithm, you will need to share the public key specified in `[rsa|ecdsa]_pub_file` with resource providers, and you will need to keep the private key `[rsa|ecdsa]_key_file` safe.

The values available for the parameter `key_size` are 256, 284 and 512 only. To choose your signature algorithm, set the value `true` to the parameter `use_[rsa|ecdsa|sha]` you want, and `false` to the other ones. Finally, set the additional parameter used for your algorithm:
- `*_key_file` and `*_pub_file` if you choose ECDSA or RSA signatures, with the path to the public and private signature files
- `sha_secret` if you choose SHA signatures, with the value of the secret

#### RSA private/public key creation

You can use the following example commands to create a pair of private and public keys for the algorithms RSA or ECDSA:

```SHELL
$ # RS512
$ # private key
$ openssl genrsa -out private-rsa.key 4096
$ # public key
$ openssl rsa -in private-rsa.key -outform PEM -pubout -out public-rsa.pem

$ # ES512
$ # private key
$ openssl ecparam -genkey -name secp521r1 -noout -out private-ecdsa.key
$ # public key
$ openssl ec -in private-ecdsa.key -pubout -out public-ecdsa.pem
```

For more information about generating keys, see [OpenSSL Documentation](https://www.openssl.org/docs/)

### Install as a service

The files `glewlwyd-init` (SysV init) and `glewlwyd.service` (Systemd) can be used to have glewlwyd as a daemon. They are fitted for a Raspbian distribution, but can easily be changed for other systems.

#### Install as a SysV init daemon and run

```shell
$ sudo cp glewlwyd-init /etc/init.d/glewlwyd
$ sudo update-rc.d glewlwyd defaults
$ sudo service glewlwyd start
```

#### Install as a Systemd daemon and run

```shell
$ sudo cp glewlwyd.service /etc/systemd/system
$ sudo systemctl enable glewlwyd
$ sudo sudo systemctl start glewlwyd
```

## Run Glewlwyd

Run the application using the service command if you installed the init file:

```shell
$ sudo service glewlwyd start
```

You can also manually start the application like this:

```shell
$ ./glewlwyd --config-file=glewlwyd.conf
```

By default, Glewlwyd is available on TCP port 4593. You can use the test page `tests/test-token.html` to validate the behaviour. To access it, copy the file into webapp and go to the url: [http://localhost:4593/app/test-token.html](http://localhost:4593/app/test-token.html).

## Resource server authorization usage and access tokens

Glewlwyd provides [JSON Web Tokens](https://jwt.io/) which is a standard way to validate a token without asking the authorization server.
A JSON Web Token (JWT) comes with a signature that authenticates itself.

There is no way for a resource server to determine if a token is valid, except by checking the Authorisation token. i.e., there is no API where you could ask Glewlwyd server if an access token is valid or not. Therefore, the resource server MUST verify the access token using its signature.

Examples of resource server access token validation are available in the folder [clients](https://github.com/babelouest/glewlwyd/tree/master/clients/).

There are 2 ways to sign a token:
- SHA symmetric encryption
- RSA asymmetric encryption

The token parameters are located in the `jwt` block in the configuration file:
```
jwt =
{
   use_rsa = true
   rsa_key_file = "/usr/local/etc/glewlwyd/private.key"
   rsa_pub_file = "/usr/local/etc/glewlwyd/public.pem"

   use_sha = false
   sha_secret = "secret"

}
```

Depending on the algorithm you choose, you will need to share information with the resource services.
- With RSA encryption, the resource service will have to authenticate the tokens using the `rsa_pub_file` file content, you will have to keep secret the `rsa_key_file` file content to prevent token forgery.
- With SHA encryption, the resource service will have to authenticate the tokens using the `sha_secret` value, you must share the `sha_secret` value between the authentication server and the resource services, and keep it private between them to prevent token forgery.

In your resource server, you must validate all API call with the token given in the `Authorization` header as described in [RFC6750](https://tools.ietf.org/html/rfc6750). You can use a jwt library available for the language and architecture of your choice, to validate the token signature and content. If the token signature is verified, you MUST manually validate the token time validity using the values `iat` and `expires_in` in the token payload.

Every `access_token` has the following header and payload format:

```javascript
// Header
{
  "typ": "JWT",
  "alg": "RS512" // for RSA signatures, HS512 for SHA signatures
}
// Payload
{
  "expires_in": 3600,       // Token expiration in seconds, default is 1 hour
  "iat": 1484278795,        // Issued at, time in UNIX epoch format
  "salt": "abcd1234",       // A random string
  "scope": "scope1 scope2", // The scope values
  "type": "access_token",   // The token type
  "username": "admin"       // The username who was granted access for this scope
}
```

Refresh and session tokens are also JWTs, but their payload have slightly different values. A session token doesn't have a scope value, and the `type` values are respectively `refresh_token` and `session_token`. Although these tokens are validated by the Glewlwyd server directly.

## Front-end application

The front-end management application is a tiny single page app (SPA) written in ReactJS/JQuery, responsive as much as I can, not the best design in the world, but useful anyway.

All front-end pages have a minimal design, feel free to modify them for your own need, or create your own application.

### Glewlwyd manager

Glewlwyd comes with a small front-end that uses the back-end API to manage profile, users, clients, scopes, resources and authorization types.

#### Configuration

The config file `glewlwyd.conf` contains the following variables: `static_files_path` and `static_files_prefix`, `static_files_path` is the path to the front-end application. Set it to the location of your webapp folder before running glewlwyd, e.g. `"/home/pi/glewlwyd/webapp"`, `static_files_prefix` will be the url path to access to the front-end application, default is [http://localhost:4953/app/](http://localhost:4953/app/).

#### Change front-end path

If you want to change the path to the front-end application, e.g. change it from http://localhost:4953/app/ to http://localhost:4953/admin/ for example, there are 2 steps to follow.

- 1: Change the values `static_files_prefix`, `login_url` and `grant_url` in the configuration file

The value `static_files_prefix` must match your new path, e.g. `admin`, the `login_url` and `grant_url` must be changed accordingly, e.g. `"../admin/login.html?"` and `"../admin/grant.html?"`.

- 2: Change the `redirect_uri` value of the g_admin client in the database, e.g.:

```SQL
UPDATE g_redirect_uri set gru_uri='../admin/index.html' where gc_id=(SELECT gc_id from g_client WHERE gc_client_id='g_admin');
```

#### Scope

To connect to the management application, you must use a user that have `g_admin` scope.

### tests/test-token.html

This page is here only for oauth2 tests and behaviour validation. If you want to use it, you need to update the `glewlwyd_api` value and all parameters provided, such as `redirect_uri`, `scope` and `client`.

Beware, all password inputs are of type `text`, so a typed password is not hidden from a hidden third-party dangerous predator.