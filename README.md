README Tecnico - WebApp Prenotazione Mezzi
Architettura del Sistema

Il portale è una Single Page Application (SPA) basata su architettura serverless ibrida:

    Frontend: Hostato su GitHub Pages. Realizzato in HTML5, CSS3, e Vanilla JavaScript.

    Backend & Database: Basato su Google Apps Script (GAS) collegato a Google Calendar. Calendar agisce non solo come interfaccia visiva per l'autoparco, ma come un vero e proprio database NoSQL basato su eventi temporali.

    Gestione Notifiche: API MailApp / GmailApp di Google.

Struttura del Frontend (GitHub)

    Interfaccia UI/UX: L'interfaccia utilizza SweetAlert2 per la gestione modale degli alert e la conferma delle azioni, garantendo una UX moderna e mobile-responsive.

    Validazione Dati: Il JS effettua controlli lato client su:

        Formattazione date e incongruenze temporali (es. data di fine antecedente alla data di inizio).

        Preavviso (regola delle 24 ore o limitazioni di tempo).

    Fetch API: Tutti i payload JSON vengono inviati tramite chiamate asincrone (fetch) verso l'endpoint in formato stringa testuale, gestendo le risposte CORS in modo trasparente.

Struttura del Backend (Google Apps Script)

Il codice è il cuore operativo, diviso in due funzioni principali di trigger HTTP:
1. Endpoint doPost(e) - Creazione Richieste

    Riceve il payload JSON dal Frontend.

    Sistema anti-collisione: Prima di creare l'evento, esegue una query su Google Calendar filtrando per l'ID o il nome del mezzo selezionato.

    Se trova eventi con diciture "Fermo Tecnico" o "Approvato", abortisce la richiesta restituendo status error.

    Se libero, crea l'evento in Calendar inserendo tag specifici (es. [IN ATTESA]) nel titolo.

    Genera e spedisce la mail HTML con i token sicuri (link con parametri URL unici) all'Autorizzatore per gestire la richiesta.

2. Endpoint doGet(e) - Gestione Approvazioni

Gestisce i click sui link contenuti nelle mail inviate agli autorizzatori. Analizza i parametri dell'URL (?action=approve&eventId=XXXXX).

    Azione APPROVE: Cerca l'evento tramite ID, modifica il titolo sostituendo [IN ATTESA] con [APPROVATO], cambia il colore dell'evento nel calendario e usa MailApp per inviare la notifica di successo al militare.

    Azione REJECT: Cambia lo stato in [RIFIUTATO], libera gli orari sul calendario, e avvisa l'utente tramite mail.

    Azione REVOKE: Pensata per necessità operative superiori. Sovrascrive un'approvazione già avvenuta, oscura l'evento e dirama un avviso urgente di revoca al richiedente.

Vantaggi della Struttura Implementata

    Costo Zero & Alta Affidabilità: Sfruttando GitHub e l'infrastruttura Enterprise di Google, il sistema gode di uptime del 99.9% senza necessità di hosting a pagamento.

    Sicurezza per Obscurity e ID Routing: L'esposizione pubblica del front-end non compromette il backend, poiché le chiavi del calendario e la logica di approvazione risiedono inaccessibili dentro i server Google. Le mail di approvazione contengono ID univoci generati da Google Calendar, impossibili da prevedere.

    Visualizzazione Centralizzata (Dashboard Autoparco): Il Sergente incaricato della logistica mezzi non deve guardare dashboard complessi; gli basta aprire il proprio account Google Calendar dedicato per avere la panoramica visiva, mensile o settimanale, del dislocamento di tutti i veicoli, compresi i fermi tecnici inseriti manualmente.
