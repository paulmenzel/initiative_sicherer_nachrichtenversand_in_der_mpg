% Sicherer Nachrichtenversand in der MPG
% Paul Menzel (Max-Planck-Institut für molekulare Genetik)
% 8. November 2018

## Wer bin ich?

![Logo of Max Planck Institute for Molecular Genetics](images/MPIMG_helix_rgb.png){ height=25% }\


- Systemarchitekt beim [Max-Planck-Institut für molekulare Genetik](https://www.molgen.mpg.de/)
- Diplom-Wirtschaftsmathematiker ([TU Berlin](https://www.tu-berlin.de/))
- FLOSS-Befürworter

## Präsentation

Folien in Markdown mit [Pandoc](https://pandoc.org/) nach LaTeX-Beamer umgewandelt, verfügbar auf GitHub.

TinyURL: <https://tinyurl.com/smtpinitiative>

<https://github.com/paulmenzel/initiative_sicherer_nachrichtenversand_in_der_mpg>

# Problemstellung

## Ziel

-  Sichere Übertragung von Daten innerhalb der MPG
-  Geheim und authentifiziert
-  Ausweitung auf alle Universitäten und Forschungseinrichtungen

### Betrachtung in Vortrag

-  SMTP: Zwischen SMTP-Servern (MTA)

## Angriffsmodell

-  Annahme: Keine Übernahme der Server durch Angreifer
-  Annahme (SMTP): Vertrauen in Betreiber der Server auf Sender- und (Ziel-)Empfängerseite
-  Mittelsmannangriff

## Mittelsmannangriff

![Mittelsmannangriff (<https://www.elie.net/blog/understanding-how-tls-downgrade-attacks-prevent-email-encryption)>](images/how_email_encryption_works.jpg){ height=50% }

## Realistisch?

-  Innerhalb der MPG: DFN-Netz separat vom „Internet“
-  Dienste außerhalb

```
$ host -t mx maxplanckflorida.org
maxplanckflorida.org mail is handled by \
0 maxplanckflorida-org.mail.protection.outlook.com.
$ host -t mx cbs.mpg.de
cbs.mpg.de mail is handled by \
10 mx0-cbs-mpg.heinlein-support.de.
cbs.mpg.de mail is handled by 20 \
mx1-cbs-mpg.heinlein-support.de.
```

-  Netzwerkgeräte meist im Ausland produziert und enthalten Blobs
-  Snowden-Veröffentlichungen zeigen, dass realistisch

## Lösungen (TLS)

-  SMTP: `STARTTLS`

-  Zertifizierungsstellen (DFN, [Let’s Encrypt](https://letsencrypt.org/))
-  [Monkeysphere Project](http://web.monkeysphere.info/)
-  DNSSEC/DANE

### Nur bei SMTP

-  Ende-zu-Ende-Verschlüsselung (PGP/GPG, S/MIME)

## Zielumsetzung

### SMTP

-  Authentifizierung: Zustellung an korrekten Server
-  Schutz der Metadaten
-  Geheime Übertragung auch bei nicht Ende-zu-Ende-Verschlüsselung

# Angriffe

## Poodle, DROWN, …

Verschiedene Angriffe.

1.  Downgrade-Attacke (STARTTLS)
2.  Poodle, DROWN
3.  Unsichere Chiffren

## Sichere Konfiguration

<postmaster@….mpg.de> nach RFC 822 Pflicht!

1.  [BetterCrypto.org](https://bettercrypto.org/)
1.  [Mozilla Wiki: Security/Server Side TLS](https://wiki.mozilla.org/Security/Server_Side_TLS)
1.  [Cipherli.st](https://cipherli.st/)

## Test

### WWW

1.  [Hardenize](https://www.hardenize.com/)
1.  [SSL-Tools](https://ssl-tools.net/)
1.  [SSL Server Test von Qualys SSL Labs](https://www.ssllabs.com/ssltest/analyze.html?d=login.rz.ruhr-uni-bochum.de)

### Kommandozeile

1.  OpenSSL, GnuTLS
1.  [Nmap](https://nmap.org/)
1.  [SSLyze](https://nabla-c0d3.github.io/)
1.  SMTP: `posttls-finger`

# SMTP

## Ideal für SMTP

Mehrere Komponenten: DNS, Zertifikate

### TLS

-   MX-Eintrag stimmt mit Servernamen überein

### DNSSEC/DANE

-  TLSA-DNS-Einträge

## Beispiel zu posttls-finger

    $ /usr/sbin/posttls-finger -c -l secure \
    -P /etc/ssl/certs mpifr-bonn.mpg.de
    posttls-finger: mail2.mpifr-bonn.mpg.de[134.104.18.60]:25: \
    Matched subjectAltName: mail2.mpifr-bonn.mpg.de
    posttls-finger: mail2.mpifr-bonn.mpg.de[134.104.18.60]:25 \
    CommonName mail2.mpifr-bonn.mpg.de
    posttls-finger: mail2.mpifr-bonn.mpg.de[134.104.18.60]:25: \
    subject_CN=mail2.mpifr-bonn.mpg.de, issuer_CN=MPG CA, \
    fingerprint=CA:5D:E7:7C:8A:6B:C5:4B:CC:7E:DB:F1:0C:43:C1:76:48:15:8C:38, \
    pkey_fingerprint=FD:27:CA:F2:DD:0B:AD:91:9C:6E:83:90:5E:A4:D7:DF:1A:50:BB:F5
    posttls-finger: Verified TLS connection established to \
    mail2.mpifr-bonn.mpg.de[134.104.18.60]:25: \
    TLSv1.2 with cipher \
    ECDHE-RSA-AES256-GCM-SHA384 \(256/256 bits)

## Probleme mit DFN-Mailsupport

1.  DFN kein DNSSEC
1.  Seit zwei Jahren
1.  Stand: 1. Halbjahr 2019

## Postfix-Konfiguration

1.  tls_polcy: dane
1.  Zitat:

# MPG

# Inkorrekte MX-Einträge

## Beispiel GV

    $ host -t mx mpg.de
    mpg.de mail is handled by 5 mx1.mpg.de.
    mpg.de mail is handled by 5 mx2.mpg.de.
    $ host mx1.mpg.de
    mx1.mpg.de has address 194.95.232.60
    mx1.mpg.de has address 194.95.238.60
    mx1.mpg.de has address 194.95.234.60
    $ host 194.95.232.60
    60.232.95.194.in-addr.arpa domain name pointer \
    mfilter-123-1-1.mx.srv.dfn.de.

Keine Antwort von postmaster@mpg.de auf Nachricht.

## Betroffen (9. November 2017)

-  Mindestens 15 Einrichtungen mit veralteten mx??.mpg.de MX-Einträgen.
-  Mindestens 4 Einrichtungen mit veralteten mx??.gwdg.de MX-Einträgen.

## Problem Verwaltungsadressen

```
$ host vw.molgen.mpg.de
vw.molgen.mpg.de mail is handled by 5 mx2.mpg.de.
vw.molgen.mpg.de mail is handled by 5 mx1.mpg.de.
```

```
$ host vw.molgen.mpg.de
vw.molgen.mpg.de mail is handled by \
5 mfilter-123-1-2.mx.srv.dfn.de.
vw.molgen.mpg.de mail is handled by \
5 mfilter-123-1-1.mx.srv.dfn.de.
vw.molgen.mpg.de mail is handled by \
5 mfilter-123-1-3.mx.srv.dfn.de.
```

Bitte überprüfen!

GWDG und DFN sollten aktiv werden.

# Initiative

## Geschichte

1.  2011(?) Jan Behrendt TLS
1.  Community-Award

## Aktueller Stand

1.  DNSSEC und DANE wenig verbreitet
1.  MTA-STS noch in Kinderschuhen
    1.  Erste Verbindung ungesichert
1.  Eigene Lösung

##  Textdatei mit Domains mit korrektem Zertifikat

1.  Ähnlich HTTPS-Everywhere
1.  Git-Depot: https://gitlab.com/dpkg/tls-policy
1.  Zusammenführungsanfragen (Merge-Requests)
1.  Für alle MTA-Betreiber (insbesondere Unis)

## Beispiel Postfix

1.  tls\_policy
1.  Kommasepariert
1.  cron-job oder abonnieren der Änderungen

## Fazit

1.  MPG-Netz auch Vorbildwirkung
1.  Mehr Gewissenhaftigkeit
1.  Mehr Bewusstsein (DNSSEC, DFN)
1.  Ohne DANE keine automatische Konfiguration möglich, manuelle Konfiguration erforderlich
1.  Unerstützung von MTA-STS
1.  Überprüfung von MTA-Servern
1.  Ende-zu-Ende-Verschlüsselung

# Fragen
