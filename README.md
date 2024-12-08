# JDAV Auth

## Ist-Zustand

Services

- Nextcloud mit eigenem Zugang
- DAV360 mit eigenem Zugang
- Depot mit Zugang über auth.jdav-freiburg.de
- Auth-Manager: [Frontend](https://github.com/voegtlel/auth-manager-frontend), [Backend](https://github.com/voegtlel/auth-manager-backend)

Neue Juleis anzulegen ist momentan aufwendig, man muss vier verschiedene Accounts / User anlegen.
Kein On- oder Offboarding.
Workflow unklar.
Security fragwürdig (weil selbst entwickelt und auf dem Stand von vor vier Jahren)

Aktueller Onboarding-Prozess Auth-Manager (wird nur beim Depot genutzt):

1. Admin legt Account an
1. Admin schickt Link zum Registrieren
1. User registriert sich selbst: Passwort vergeben, Stammdaten eingeben
1. User hat Zugang zum Dienst

Nextcloud: eigener Onboarding-Prozess

DAV360: Onboarding über Geschäftsstelle

## Requirements

- On-/Offboarding Prozesse
- User-self-services: PW reset, Stammdaten editieren
- Hierarchische Gruppen (JuLei > Jugendgruppe)
- (Zukunft: Mattermost für Alpingruppe)
- Integration im Depot
- Integration in Nextcloud
- ? Willkommens-Mail über DAV360 mit weiteren Instruktionen? Möglich?

## Actions / TODOs

- [x] Authentik Test-Instanz
....
Todos siehe [github issues](https://github.com/jdav-freiburg/auth-service/issues)

## Authentik Grundeinrichtung

Siehe auch [Authentik > Installation > Docker Compose](https://docs.goauthentik.io/docs/install-config/install/docker-compose).

Postgres-Password und Authentik Secret Key generieren

```shell
echo "PG_PASS=$(openssl rand -base64 36 | tr -d '\n')" >> .env
echo "AUTHENTIK_SECRET_KEY=$(openssl rand -base64 60 | tr -d '\n')" >> .env
```

E-Mail Credentials für den E-Mail Relay Service hinterlegen.

```ini
# SMTP Host Emails are sent to
AUTHENTIK_EMAIL__HOST=mail
AUTHENTIK_EMAIL__PORT=25
# Optionally authenticate (don't add quotation marks to your password)
AUTHENTIK_EMAIL__USERNAME=""
AUTHENTIK_EMAIL__PASSWORD=""
# Use StartTLS
AUTHENTIK_EMAIL__USE_TLS=false
# Use SSL
AUTHENTIK_EMAIL__USE_SSL=false
AUTHENTIK_EMAIL__TIMEOUT=10
# Email address authentik will send from, should have a correct @domain
AUTHENTIK_EMAIL__FROM=support@jdav-freiburg.de
# Auskommentieren um Benachrichtigungen zu Fehlern per Mail zu bekommen
# AUTHENTIK_ERROR_REPORTING__ENABLED=true
```

Ggf. Ports in der `.env` setzen, die default Ports sind `9000` und `9443`

```ini
# COMPOSE_PORT_HTTP=80
# COMPOSE_PORT_HTTPS=443
```

Authentik starten mit

```shell
docker compose pull
docker compose up -d
```

Ersteinrichtung über http://localhost:9000/if/flow/initial-setup/, es müssen eine E-Mail Adresse und ein Passwort für den Admin-Account `akadmin` eingegeben werden.

E-Mail Versand testen (erfordert, dass der [Mail-Service](https://github.com/jdav-freiburg/mail-relay) bereits läuft)

```shell
docker compose exec worker ak test_email <your-email@example.net>
```

### Nextcloud Anbindung

[Authentic Nextcloud](https://docs.goauthentik.io/integrations/services/nextcloud/) mit OpenID Connect.

#### Authentik: Provider und Anwendung anlegen

See also: [Authentik: Provider und Anwendung anlegen](https://docs.goauthentik.io/integrations/services/nextcloud/#provider-and-application)

Authentik > Admin > Applications > Providers > Create

```
Name: Nextcloud
Authorization flow: default-provider-authorization-explicit-consent (Authorize Application)
Client type: Confidential
Client ID: will be needed for Nextcloud
Client Secret: will be needed for Nextcloud
Redirect URIs/Origins (RegEx): https://cloud.jdav-freiburg.de/apps/user_oidc/code
Signing key: Any valid certificate

Advanced Protocol Settings:
  Scopes:
    authentik default Oauth Mapping email
    authentik default Oauth Mapping profile
  # Use Authentik Login-Email as ID
  Subject Mode: Based on the User's Email
  Include claims in ID token: ✔️
```

Authentik > Admin > Applications > Applications > Create

```
Name: Nextcloud
Slug: nextcloud
Provider: Nextcloud
```

### Nextcloud OpenID App und Einstellungen

See also: [Authentik: Nextcloud Integration](https://docs.goauthentik.io/integrations/services/nextcloud/#nextcloud-1)

Nextcloud > Menu > Apps > Integration > "OpenID Connect user backend" installieren

Nextcloud > Menu > Administration Settings > OpenID Connect > Register New Provider

- Discovery endpoint: the slug "nextcloud" was set earlier in Authentik (see above)
- Discovery endpoint: the domain can be replaced with an internal FQDN when behind a Reverse Proxy with [extra configuration](https://docs.goauthentik.io/integrations/services/nextcloud/#extra-configuration-when-running-behind-a-reverse-proxy)

```
Identifier: Authentik
Client ID: The client ID from the provider
Client secret: The secret ID from the provider
Discovery endpoint: https://cloud.jdav-freiburg.de/application/o/nextcloud/.well-known/openid-configuration
Scope: email profile

Attribute mapping
  User ID mapping: sub

Extra attributes mapping
  Display name mapping: name
  Email mapping: email

Authentication and Access Control Settings
  # Use Authentik Groups in Nextcloud
  ✔️ Use group provisioning
  # Maybe not needed?
  ✔️ Send ID token hint on logout
```

## Authentik: Backups

Einstellungen, Providers, Policies, etc. werden in einer PostgreSQL Datenbank gespeichert. Vorschaubilder für Anwendungen werden im Ordner `/media` abgespeichert. Diese beiden Orte sollten bei einem Backup berücksichtigt werden.

Ggf. auch das Volume `redis:/data` sichern um "Inconviniences" zu vermeiden ([github issue](https://github.com/goauthentik/authentik/issues/8411#issuecomment-1940493275))
