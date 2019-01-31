# Keycloak as Gerrit SAML provider

[Gerrit](https://www.gerritcodereview.com) is a code review and project
management tool for Git based projects.

[Keycloak](https://www.keycloak.org/) is open source Identity and Access
Management tool.

## Objective

This doument provides a step-by-step tutorial how to set-up Keycloak as
SAML provider for Gerrit Code Review for development only. For production
HTTPS protocol must be configured.

## Prerequisites

- Linux or MacOS operating system
- Docker-compose is installed

## Documentation

1. Install Keycloak official Docker image from this repository and start it:

```bash
  $ git clone https://github.com/jboss-dockerfiles/keycloak
  $ cd keycloak/docker-compose-examples
  $ docker-compose -f keycloak-postgres.yml up
```

2. Login to Keycloak using admin/Pa55w0rd credentials and import keycloak
[json file](../master/resources/keycloak-gerrit-client-export.json).

3. Create test user: John Doe, with username "jdoe", email: john@doe.org, with
password "secret", Temporary=OFF.

4. Configure gerrit according to provided [gerrit.config](../master/resources/gerrit.config).
Note this Keycloak SAML provider configuration section:

```
[saml]
    serviceProviderEntityId = SAML2Client
    keystorePath = /Users/davido/projects/test_site_saml/etc/samlKeystore.jks
    keystorePassword = pac4j-demo-password
    privateKeyPassword = pac4j-demo-password
    metadataPath = http://localhost:8080/auth/realms/master/protocol/saml/descriptor
    userNameAttr = UserName
    displayNameAttr = DisplayName
    emailAddressAttr = EmailAddress
    computedDisplayName = true
    firstNameAttr = firstName
    lastNameAttr = lastName
```

5. Generate keystore in `$gerrit_site/etc` local keystore:

```
keytool -genkeypair -alias pac4j -keypass pac4j-demo-password \
  -keystore samlKeystore.jks \
  -storepass pac4j-demo-password -keyalg RSA -keysize 2048 -validity 3650
```

6. Set up gerrit site using latest released gerrit.war and select OAUTH
authentication scheme using:

```bash
  $ java -jar gerrit.war init -d gerrit_site_oauth
```

7. Build saml module from this change (it's not merged yet) in gerrit tree:
https://gerrit-review.googlesource.com/c/plugins/saml/+/212177.

```
  $ cd gerrit/plugins
  $ ln -s ../../saml .
  $ cd ..
  $ cp ../saml/external_plugin_deps.bzl plugins
  $ bazel build plugins/saml
```

8. Copy saml.jar module to <gerrit_site>/lib.

9. Start gerrit using: `<gerrit_site>/bin/gerrit.sh start`

10. Enter gerrit URL in browser: http://localhost:8081 and hit "Sign In" button

11. Keycloak Login Dialog should appear

12. Enter user: "jdoe" and password: "secret"

13. You are redirected to gerrit and the first user/admin John Doe is created
in gerrit with the right user name and email address.

14. Congrats, you have Gerrit / Keycloak OAuth2 integration up and running.

