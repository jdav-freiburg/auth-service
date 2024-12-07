# JDAV Auth

- Ist-Zustand
- Requirements
- Actions

## Ist-Zustand

Services

- Nextcloud mit eigenem Zugang
- DAV360 mit eigenem Zugang
- Depot mit Zugang über auth.jdav-freiburg.de
- Auth-Manager: [Frontend](https://github.com/voegtlel/auth-manager-frontend), [Backend](https://github.com/voegtlel/auth-manager-backend)

Neue Juleis anzulegen ist momentan aufwendig, man muss vier verschiedene Accounts / User anlegen.
Kein On- oder Offboarding.
Workflow unklar.
Security fragwürdigt (weil selbst entwickelt und auf dem Stand von vor vier Jahren)

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
- (Auth-Manager dokumentieren: Workflows, APIs)
- Authentik testen / evaluieren
  - Nextcloud Einbindung testen
  - Depot-Einbindung testen / Neu entwickeln
- Migration aus Auth-Manager & Nextcloud
  - Mapping der User & User-Gruppen & Berechtigungen

## Authentik

- Mit Docker Compose einrichten [url](https://docs.goauthentik.io/docs/install-config/install/docker-compose)
- E-Mail Service einreichten und mit `docker compose exec worker ak test_email your-email@example.net` testen
- Self-Service Workflow definieren, einrichten, dokumentieren
- Gruppen definieren (Berggämse, ...)
- Permissions definieren
- Rollen definieren (UserManager, ...)
- Einladungsflow
- Anwendung einbinden (private Nextcloud Kasimir)

### Workflow: Ersteinrichtung



## Authentik einrichten

Siehe auch [Authentik > Installation > Docker Compose](https://docs.goauthentik.io/docs/install-config/install/docker-compose).

`.env` Datei anlegen und E-Mail Credentials für den E-Mail Relay Service hinterlegen.

```ini
# SMTP Host Emails are sent to
AUTHENTIK_EMAIL__HOST=<host>
AUTHENTIK_EMAIL__PORT=<port>
# Optionally authenticate (don't add quotation marks to your password)
AUTHENTIK_EMAIL__USERNAME=
AUTHENTIK_EMAIL__PASSWORD=
# Use StartTLS
AUTHENTIK_EMAIL__USE_TLS=false
# Use SSL
AUTHENTIK_EMAIL__USE_SSL=false
AUTHENTIK_EMAIL__TIMEOUT=10
# Email address authentik will send from, should have a correct @domain
AUTHENTIK_EMAIL__FROM=<authentik@localhost>
AUTHENTIK_ERROR_REPORTING__ENABLED=true
```

Postgres-Password und Authentik Secret Key generieren

```shell
echo "PG_PASS=$(openssl rand -base64 36 | tr -d '\n')" >> .env
echo "AUTHENTIK_SECRET_KEY=$(openssl rand -base64 60 | tr -d '\n')" >> .env
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

E-Mail Versand testen mit

```shell
docker compose exec worker ak test_email your-email@example.net
```

### Nextcloud Anbindung

[Authentic Nextcloud](https://docs.goauthentik.io/integrations/services/nextcloud/) mit OpenID Connect.

#### [Authentik: Provider und Anwendung anlegen](https://docs.goauthentik.io/integrations/services/nextcloud/#provider-and-application)

Authentik > Admin > Applications > Providers > Create

```
Name: Nextcloud
Authorization flow: default-provider-authorization-explicit-consent (Authorize Application)
Client type: Confidential
Redirect URIs/Origins (RegEx): https://<nextcloud-url>/apps/user_oidc/code
Signing key: Any valid certificate

Advanced Protocol Settings:
  Scopes:
    authentik default Oauth Mapping email
    authentik default Oauth Mapping profile
  Include claims in ID token: ✔️
```

Authentik > Admin > Applications > Applications > Create

```
Name: Nextcloud
Slug: nextcloud
Provider: Nextcloud
```

2. In Nextcloud die OpenID App installieren: Menü > Apps > Einbindung > OpenID Connect user backend
3. In Nextcloud einen neuen OpenID Provider anlegen: Menü > Einstellungen > OpenID Connect
