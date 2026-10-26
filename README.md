# Passly Proof · Wächter

Dieses Repo beobachtet das öffentliche Register hinter **Passly Proof**
([getpassly.de/proof](https://getpassly.de/proof)). Passly schreibt jedes Ticket
und jeden Zustandswechsel (ausgestellt, weitergegeben, eingelassen, widerrufen)
als Eintrag in ein nur wachsendes Register und unterschreibt regelmäßig dessen
Stand. Dieses Repo hält jeden Stand fest, damit eine nachträgliche Änderung
der Geschichte auffällt.

**Wer es betreibt:** Passly selbst, getrennt von der Anwendung (eigener
Schlüssel, eigenes Repo, GitHub Actions). Es schützt damit gegen Fehler und
gegen Eingriffe von außen. Gegen Passly selbst schützt am besten, wer den
Stand ebenfalls mitschreibt: Spiegel dieses Repo oder führe die Prüfung unten
selbst aus. Alles hier ist offen.

## Was hier jede Stunde passiert

Die GitHub Action [`waechter.yml`](.github/workflows/waechter.yml):

1. lädt den aktuellen Stand (`checkpoint`) aus den öffentlichen Dateien
   ([`basis`](basis)) und prüft die Unterschrift gegen den hier festgehaltenen
   Prüfschlüssel ([`vkey`](vkey)),
2. beweist, dass der neue Stand den zuletzt hier gespeicherten **fortsetzt**
   (Konsistenzbeweis nach RFC 9162): nichts wurde gelöscht, geändert oder
   umsortiert,
3. liest **jeden Eintrag** und rechnet die Wurzel und jede Kachel neu,
4. macht dasselbe noch einmal mit einer **unabhängigen Implementierung**
   ([`gegenprobe/`](gegenprobe), `golang.org/x/mod/sumdb/tlog`, die Bibliothek
   der Go-Prüfsummendatenbank),
5. committet den neuen Stand, einmal am Tag mit einem **RFC-3161-Zeitstempel**
   ([`zeitstempel/`](zeitstempel)),
6. zeichnet den geprüften Stand als **Zeuge** gegen ([`bezeugt`](bezeugt),
   C2SP `tlog-cosignature/v1`, öffentlicher Schlüssel in [`zeuge.vkey`](zeuge.vkey)).
   Passly hängt diese Bestätigung an jede Prüfung im Browser: Setzt der Stand,
   den Passly gerade zeigt, den bezeugten nicht fort, schlägt die Prüfung fehl.

Schlägt eine Prüfung fehl, bleibt der alte Stand hier stehen und der Lauf wird
rot. Die Git-Historie dieses Repos ist damit ein Protokoll aller Stände.

Solange hier keine Datei `vkey` liegt, ruht der Wächter: Das Register läuft in
Produktion erst ab der Umstellung von Passly.

## Selbst nachprüfen

Mit Node 24 (keine Installation nötig):

```sh
node pruefen.mjs stand --base "$(cat basis)" --vkey vkey --vorher checkpoint --voll
```

Mit Go:

```sh
cd gegenprobe && go run . -base "$(cat ../basis)" -vkey ../vkey -vorher ../checkpoint
```

Einen Zeitstempel prüfen:

```sh
openssl ts -verify -in zeitstempel/<tag>.tsr -data zeitstempel/<tag>.checkpoint \
  -CAfile zeitstempel/freetsa-cacert.pem -untrusted zeitstempel/freetsa-tsa.crt
```

## Für Kassenprüfer

Veranstalter laden im Passly-Dashboard („Mit Passly Proof nachzählen“) eine
Datei mit dem Schlüssel ihres Events. Damit zählt man unabhängig von Passly
nach, wie viele Tickets ausgestellt, eingelassen und widerrufen wurden:

```sh
node pruefen.mjs nachzaehlen --datei nachzaehlen-<event>.json
```

## Datenschutz

Das Register enthält außer der Art eines Eintrags nichts Lesbares: keine
Namen, keine E-Mail-Adressen, keine Konten, keine Veranstaltungen. Dieses Repo
speichert nur die unterschriebenen Gesamtstände, keine Einträge. Einzelheiten:
[getpassly.de/datenschutz](https://getpassly.de/datenschutz), Ziffer 7.

## Woher die Dateien stammen

`register.ts` und `pruefen.mjs` sind Kopien aus dem Passly-Quellcode
(`packages/core/src/register.ts`, `scripts/register/pruefen.mjs`); die
Spezifikation steht dort in `docs/register-spec.md`. `gegenprobe/` benutzt
davon nichts.
