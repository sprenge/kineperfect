# Emails versturen

Met behulp van dit scherm is het mogelijk om gepersonaliseerde emails te versturen naar patienten.

## SMTP-instellingen configureren voor e-mailverzending

Om e-mails vanuit de applicatie te kunnen versturen, moet u de SMTP-instellingen configureren. SMTP (Simple Mail Transfer Protocol) is het protocol dat gebruikt wordt om e-mails te verzenden via een externe e-mailserver. U dient de volgende gegevens in te vullen in het geval dat U brevo selecteert als provider maar U kan uiteraard gelijk welke smtp provider voorzien :

1. **SMTP-server (host)**: Dit is het adres van de server die de e-mails verstuurt.  
   **Voorbeeld**: `smtp-relay.brevo.com`

2. **SMTP-port**: Dit is de poort waarmee de e-mail wordt verstuurd. Gebruik de poort die overeenkomt met het type beveiliging dat u wilt gebruiken.  
   **Voorbeeld**:  
   - Voor TLS: poort `587`  
   - Voor SSL: poort `465`

3. **SMTP-username**: Dit is meestal uw volledige e-mailadres of een speciale gebruikersnaam die u van uw e-mailprovider hebt ontvangen.  
   **Voorbeeld**: `uwnaam@domein.com`

4. **SMTP-password**: Dit is het wachtwoord voor het e-mailadres of de API-sleutel van uw e-mailprovider.  
   **Voorbeeld**: Gebruik uw Brevo API-sleutel als wachtwoord.

5. **Beveiliging (encryption)**: Dit is het type beveiliging dat u wilt gebruiken voor het verzenden van e-mails.  
   **Voorbeeld**: Kies `TLS` voor poort 587 of `SSL` voor poort 465.

6. **SMTP source email**: Dit email adres zal de patient zien als afzender (in te stellen bij de provider)

---

### Voorbeeld SMTP-configuratie met Brevo:

- **SMTP-server**: `smtp-relay.brevo.com`
- **Port**: `587` (voor TLS)
- **Username**: `uwnaam@domein.com`
- **Password**: `[Uw API-sleutel]`
- **Beveiliging**: `TLS`

Als u vragen hebt of problemen ondervindt, neem dan gerust contact op met uw e-mailprovider voor verdere ondersteuning.

## Email templates

Hiermee kan je de tekst instellen die je naar de patienten wil sturen in functie van de actie.  Hierbij kan je gebruik maken van zogenaamde variabelen (omringd door dubbele accoladen) die vertalen in contextuele informatie.  Hieronder kan je een voorbeeld vinden voor de actie als een nieuwe gebruiker wordt aangemaakt :


```text
dag {{first_name}},

Welkom bij de kine : hierbij jou account gegevens :
username : {{username}}
password: {{password}}
server : {{server}}
port : {{port}}

Met vriendelijke groeten
```


Dit vertaalt zich bijvoorbeeld in een email met als inhoud :


```text
dag Jan,

Welkom bij de kine : hierbij jou account gegevens :
username : p945
password: een.veilig.password
server : kineperfect.duckdns.org
port : 8128

Met vriendelijke groeten
```

Je kan een gepersonaliseerde tekst opstellen voor de volgende actie :

* GDPR template : deze tekst wordt toegevoegd bij elke email en zal ook te zien zijn in de app.
* Aanmaken van een gebruiker
* Update van gebruikers profiel (uitgezonderd passwoord)
* Update van passwoord
* Verwijderen van de gebruiker

De volgende variablen kunnen gebuikt worden om de email te personaliseren :

* username
* password
* server
* port
* first_name
* last_name