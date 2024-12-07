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
- E-Mail Service einreichten und mit `docker compose exec worker ak test_email ku@extradat.de` testen
- Self-Service Workflow definieren, einrichten, dokumentieren
- Gruppen definieren (Berggämse, ...)
- Permissions definieren
- Rollen definieren (UserManager, ...)
- Einladungsflow
- Anwendung einbinden (private Nextcloud Kasimir)
