# Chrístes

**Chrístes** (dal greco *χρήστες*, "utenti") è un modulo Drupal per la **gestione degli utenti della scuola** all'interno della distribuzione Ouitoulía.

Automatizza alcune operazioni ricorrenti sul ciclo di vita degli account del personale scolastico (docenti e ATA): normalizzazione dei dati anagrafici, blocco automatico alla scadenza dell'incarico e alcune personalizzazioni dei form utente.

- **Package:** Neapoli
- **Compatibilità core:** Drupal `^10.3 || ^11`

## Funzionalità

### 1. Blocco automatico alla scadenza dell'incarico (cron)
Tramite `hook_cron`, ogni esecuzione cerca gli utenti con:
- tipologia incarico **"Tempo determinato"** (campo `field_tipologia_incarico`, vocabolario `tipologia_incarichi`);
- stato **attivo**;
- data di scadenza incarico (`field_data_scadenza_incarico`) **scaduta** (≤ data odierna).

Per ciascuno: rimuove tutti i ruoli, **blocca** l'utente e registra un messaggio nel log del canale `christes`.

> La pianificazione del cron è gestita da **Ultimate Cron** (dipendenza del modulo).

### 2. Normalizzazione dei dati anagrafici (`hook_user_presave`)
Prima di ogni salvataggio utente (da form, import o API):
- `field_nome` e `field_cognome` → iniziale maiuscola di ogni parola (`mb_convert_case`, gestione corretta degli accenti italiani);
- `field_codice_fiscale` → tutto **maiuscolo**.

### 3. Campo "Nome utente" nascondibile nel profilo
Nel form di **modifica profilo** (`user_form`), il campo *Nome utente* può essere nascosto.

Il comportamento è controllato da un'impostazione in **Amministrazione → Configurazione → Persone → Impostazioni account** (`/admin/config/people/accounts`), sezione **Chrístes**:
- ✅ **flag attivo** (default): campo *Nome utente* nascosto;
- ⬜ **flag disattivo**: campo *Nome utente* visibile.

Il valore è salvato nella config `christes.settings` (chiave `hide_username_field`).

> In fase di **registrazione** il campo nome utente è già gestito (nascosto/auto-generato) dal modulo **Auto Username** (`auto_username`), quindi qui non viene toccato.

### 4. Notifica all'utente in creazione account
Nel form di registrazione, l'opzione *"Notifica l'utente"* è preselezionata di default.

### 5. Import automatico delle configurazioni in installazione
All'installazione (`hook_install`) il modulo importa e **sovrascrive** le configurazioni presenti in:
- `config/optional/`
- `config/install/override/`

e infine svuota le cache. Utile per forzare configurazioni di base anche su siti già esistenti.

## Dipendenze

| Modulo | Scopo |
|---|---|
| `user` | Gestione utenti (core) |
| `auto_username` | Generazione automatica dello username |
| `conditional_fields` | Campi condizionali nei form |
| `login_emailusername` | Login con email o username |
| `mailsystem` | Astrazione invio email |
| `phpmailer_smtp` | Invio email via SMTP |
| `pathauto` | Alias URL automatici |
| `theme_change` | Cambio tema per contesto/utente |
| `ultimate_cron` | Pianificazione dei task cron |

## Configurazione

| Config | Chiave | Default | Descrizione |
|---|---|---|---|
| `christes.settings` | `hide_username_field` | `true` | Nasconde il campo *Nome utente* nel form di modifica profilo |

L'impostazione è modificabile da `/admin/config/people/accounts`.

## Installazione

Il modulo fa parte della distribuzione e viene installato tramite Composer come dipendenza del progetto.

```bash
drush en christes -y
drush cr
```

## Struttura del modulo

```
christes/
├── christes.info.yml      # Definizione modulo e dipendenze
├── christes.module        # Hook: cron, form alter, presave
├── christes.install       # hook_install + import configurazioni
├── composer.json          # Metadati e requisiti Composer
├── config/
│   ├── install/           # christes.settings.yml (default)
│   └── schema/            # christes.schema.yml (validazione config)
└── README.md
```

## Licenza

AGPL-3.0-only — Maintainer: Crescenzo Velleca.
