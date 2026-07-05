# Terminbuchung – technische Anforderungen (standardisiert)

Dieses Dokument beschreibt, was eine **echte** Online-Terminbuchung benötigt –
im Gegensatz zur aktuellen Demo, bei der die freien Zeiten fest im JavaScript
stehen und nichts gespeichert wird.

> **Kernaussage:** Für eine Agentur, die das an viele kleine Kunden verkauft,
> lohnt sich **Eigenbau selten**. Der standardisierte Weg ist, einen fertigen
> Buchungsdienst einzubetten – idealerweise das Open-Source-Produkt
> **Cal.com** (self-hostbar, DSGVO-freundlich, white-label).

---

## 1. Backend + Datenbank (der Kern)

Die Demo läuft rein im Browser – zwei Personen könnten denselben Slot „buchen".
Sobald es echt wird, braucht es Server + Datenbank, die Verfügbarkeiten und
Buchungen persistent speichern.

- **Doppelbuchung verhindern:** atomare Slot-Sperre / Datenbank-Transaktion.
  Zwei gleichzeitige Anfragen auf 9:00 Uhr → nur eine gewinnt.
- Zustände je Buchung: `angefragt → bestätigt → wahrgenommen / storniert / no-show`.

## 2. Verfügbarkeits-Logik (komplexer als gedacht)

- Öffnungszeiten, Pausen
- Dauer je Leistung (Zahnreinigung 45 min ≠ Kontrolle 10 min)
- Pufferzeiten zwischen Terminen
- Urlaub, Feiertage, Krankheit
- Mehrere Behandler / Ressourcen mit eigenen Kalendern

## 3. Kalender-Synchronisation – die relevanten Standards

| Standard | Zweck |
|----------|-------|
| **iCalendar (RFC 5545)** | Dateiformat für Termine (`.ics`) |
| **CalDAV (RFC 4791)** | offenes Sync-Protokoll (Apple, Nextcloud, Fastmail) |
| **Google Calendar API** | Sync mit Google-Kalender |
| **Microsoft Graph API** | Sync mit Outlook / Microsoft 365 |

**Zwei-Wege-Sync ist entscheidend:** Der Betrieb sieht Buchungen in seinem
gewohnten Kalender; trägt er dort selbst etwas ein, wird der Online-Slot
automatisch blockiert.

## 4. Benachrichtigungen

- **E-Mail:** SendGrid / Postmark / SMTP – Bestätigung, Erinnerung, Storno
- **SMS:** Twilio / MessageBird – Erinnerungen senken No-Shows deutlich
- **.ics-Anhang** zum Eintragen in den eigenen Kalender
- Storno-/Umbuchungslink pro Buchung

## 5. Zeitzonen

Immer in **UTC** speichern, lokal (Europe/Berlin) anzeigen, Sommerzeit beachten.

## 6. DSGVO – bei Ärzten der kritischste Punkt

- Name/Telefon = personenbezogene Daten.
- Bei einer Arztpraxis ist schon der **Behandlungsgrund ein Gesundheitsdatum**
  (Art. 9 DSGVO, besondere Kategorie) → deutlich strengere Anforderungen.
- Konkret nötig: EU-Hosting / Datenresidenz, Verschlüsselung,
  **AV-Vertrag** mit jedem Dienstleister (Twilio, Google, …), Einwilligung,
  Löschkonzept.
- Deshalb sind reine US-SaaS für Praxen oft heikel.

## 7. Missbrauchsschutz

Rate-Limiting, ggf. Captcha, Bestätigungslink (Double-Opt-in), No-Show-Handling.

## 8. Admin-Oberfläche

Der Kunde muss Zeiten pflegen, Buchungen sehen/absagen und Urlaub eintragen
können.

---

## Standardisiert = nicht selbst bauen

| Weg | Wann sinnvoll |
|-----|---------------|
| **Cal.com einbetten** (Open Source, self-host) | 👍 Unser Fall – volle Kontrolle, DSGVO, white-label, ein System für alle Kunden |
| **Calendly / Acuity / SimplyBook** (SaaS) | Schnell, aber US-Anbieter → für Handwerk ok, für Ärzte DSGVO prüfen |
| **Doctolib / samedi / Jameda** | Speziell Arztpraxen in DE, oft schon vorhanden → dann nur einbetten |
| **Selbst bauen** | Fast nie – 3+ der obigen Punkte sind je ein eigenes Projekt |

### Empfehlung für MonkeyPages

Ein self-hosted **Cal.com** als zentrale Buchungs-Engine, pro Kunde ein
Team/Event-Type, per iFrame oder Embed-Widget an genau die Stelle der Seite,
wo jetzt der Demo-Kalender steht. So gibt es **eine** standardisierte,
DSGVO-konforme Lösung für alle Branchen, statt pro Kunde neu zu basteln.

---

## Umsetzungs-Stufen (Vorschlag)

1. **MVP:** Cal.com self-hosted, ein Embed pro Kundenseite, E-Mail-Bestätigung.
2. **Ausbau:** SMS-Erinnerungen, Google/Outlook-Sync für den Kunden.
3. **Praxen:** AV-Verträge, EU-Hosting bestätigen, ggf. samedi/Doctolib statt
   Eigenlösung, wenn der Kunde das bereits nutzt.
