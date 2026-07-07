# GUIDA DETTAGLIATA ALLA MODELLAZIONE IN ASTAH
## Sistema di Gestione Identità Digitale AFAM

Questa guida spiega come ricreare in **Astah** tutti i diagrammi del RAD:
**7 diagrammi dei casi d'uso**, **24 sequence diagram**, **8 diagrammi delle classi**
(7 sottosistemi + 1 complessivo Entities).

I diagrammi rispettano le **euristiche di Bruegge** per i sequence e la separazione
**Boundary / Control / Entity** (BCE). Leggi prima la sezione 0 (regole generali), poi
le sezioni specifiche.

---

## 0. REGOLE GENERALI (valide per tutti i diagrammi)

### 0.1 Stereotipi e convenzioni di nome
- **Boundary** (`<<boundary>>`): schermate e interfacce verso attori esterni. Nome che finisce
  in `BND` (es. `LoginBND`, `DBMSBND`).
- **Control** (`<<control>>`): logica di un caso d'uso. Nome che finisce in `CTRL`
  (es. `LoginCTRL`).
- **Entity** (`<<entity>>`): dati di dominio persistenti (es. `StudenteAFAM`, `Portfolio`).
- Gli oggetti nei sequence si scrivono con i due punti davanti: `:LoginBND`, `:LoginCTRL`.

### 0.2 Come applicare uno stereotipo in Astah
1. Crea la classe/lifeline.
2. Tasto destro → *Stereotype* → digita `boundary`, `control` o `entity` (senza `<< >>`).
3. In alternativa, dal pannello proprietà, campo *Stereotype*.
   Astah mostrerà automaticamente `<<boundary>>` sopra il nome. Per l'icona grafica
   (cerchio/freccia) puoi usare *Base Model → Notation → Iconic*.

### 0.3 Le 7 euristiche di Bruegge per i sequence (DA RISPETTARE SEMPRE)
1. **La prima colonna (lifeline) è l'ATTORE** che inizia il caso d'uso.
2. **La seconda colonna è la BOUNDARY** che l'attore usa per iniziare il caso d'uso.
3. **La terza colonna è il CONTROL** che gestisce il resto del caso d'uso.
4. **I control sono creati dalle boundary** che iniziano il caso d'uso
   (freccia `<<create>>` dalla boundary al control).
5. **Le boundary (successive) sono create dai control**
   (es. il control apre una nuova schermata: `CTRL -> NuovaBND : mostra()`).
6. **Le entity sono accedute da control e boundary** (ricevono messaggi da essi).
7. **Le entity NON accedono MAI a boundary o control** (non partono frecce dalle entity
   verso boundary/control). Questo permette di condividere le entity fra più casi d'uso.

### 0.4 Regole specifiche del nostro progetto (in aggiunta alle euristiche)
- **DBMS è un attore**, ma prima di esso c'è **sempre** la boundary `DBMSBND`.
  Ogni accesso ai dati è: `CTRL -> DBMSBND : queryXxx()` e poi `DBMSBND -> DBMS : queryXxx()`.
  Non collegare mai direttamente il control al DBMS saltando la DBMSBND.
- **Entity StudenteAFAM legata al token**: quando un'operazione recupera/salva dati a partire
  dall'utente autenticato, il control chiede prima la matricola all'entity
  (`CTRL -> StudenteAFAM : getMatricola()`) e poi usa quella matricola nelle query
  (`CTRL -> DBMSBND : queryXxx(matricola)`).
- **Ordine delle lifeline** (da sinistra a destra):
  `Attore → Boundary/e schermate → Control → MessaggioBND → ErroreBND → Entity → DBMSBND → DBMS → Provider esterni`.

> **Nota sul DBMS (coerenza col materiale del corso).** Il DBMS è un **attore** esterno
> (come nell'esempio LOGIN delle slide). Poiché ogni attore dialoga col sistema tramite una
> boundary, nei **diagrammi** (sequence e class) si inserisce la boundary `DBMSBND` davanti al
> `DBMS` — coerente con l'esempio 30/30. Attenzione però: nel **testo del flusso degli eventi**
> dei casi d'uso NON si nomina il `DBMSBND`; si scrive il dialogo col DBMS-attore, ad es.
> «il sistema richiede al DBMS…» / «il DBMS restituisce…». Il `DBMSBND` vive quindi solo nei
> diagrammi, non nella descrizione testuale dei casi d'uso.

### 0.5 Frammenti combinati (Combined Fragments)
In Astah: seleziona i messaggi coinvolti → tasto destro → *Create Combined Fragment*,
oppure usa la toolbar del sequence (icona "frame").
- **FINCHÉ (ciclo)** → frammento **`loop`**. Scrivi la guardia nel formato `[condizione]`
  (es. `[credenziali != corrispondono]`). Tutti i messaggi che si ripetono vanno DENTRO il frame.
- **SE … ALTRIMENTI** → frammento **`alt`** con **due operandi** (due scomparti separati da una
  linea tratteggiata). Nel primo operando scrivi la prima guardia `[guardia]`, nel secondo `[else]`.
- **Invocazione di un altro caso d'uso** → frammento **`ref`** (*Interaction Use*): rettangolo
  con la scritta `ref` e il nome del caso d'uso richiamato.
- **Frammenti annidati**: un `alt` può stare dentro un `loop` (es. Login: dentro il loop di
  reinserimento c'è l'alt formato-errato / credenziali-errate). Crea prima il frame esterno,
  poi seleziona i messaggi interni e crea il frame annidato.

### 0.6 Messaggi: sincroni, di ritorno, di creazione
- Messaggio sincrono: freccia piena continua (default per chiamate di metodo).
- Ritorno (opzionale): freccia tratteggiata. Nei nostri diagrammi i ritorni sono per lo più
  impliciti; esplicitali solo dove serve chiarezza (es. `Provider --> CTRL : attributiIdentita`).
- **`<<create>>`**: messaggio tratteggiato con stereotipo `create`, dalla boundary al control
  (crea l'oggetto control). In Astah: crea il messaggio, poi imposta *Stereotype = create*.
- Auto-messaggio: freccia che rientra sulla stessa lifeline (es. `CTRL -> CTRL : verificaFormato()`).
  Nota di stile: per aderenza stretta all'euristica 2-3, il **primo** inoltro dopo il create può
  essere reso come `Boundary -> Control` (es. `LoginBND -> LoginCTRL : verificaCredenziali()`);
  gli auto-messaggi restano validi per i controlli interni.

### 0.7 Punto di partenza: la Bacheca Pubblica (index)
**Ogni** diagramma di sequenza parte dalla **Bacheca Pubblica** (`:BachecaPubblicaBND`), che è
l'index e l'unico punto d'ingresso del sistema. Vale sia per gli utenti esterni sia per gli utenti
già autenticati: l'utente si trova sulla bacheca e, da lì, naviga verso la schermata dell'azione.

- La **prima lifeline** dopo l'attore è sempre `:BachecaPubblicaBND`.
- Il **primo messaggio** è l'attore che, dalla bacheca, clicca la voce di navigazione (es.
  `clickDashboard()`, `clickArchivio()`, `clickPortfolio()`, `clickNotifiche()`, `clickAccedi()`).
- Se la schermata dell'azione si raggiunge attraverso schermate intermedie, si mostra l'intera
  catena di navigazione (percorso completo). Esempio *Modifica profilo*:
  Bacheca → Dashboard → Impostazioni profilo → (flusso dell'azione).
- Al termine, il sequence ritorna alla schermata coerente col contesto (tipicamente la schermata
  dell'azione appena aggiornata; per il Logout e per i link non validi si ritorna alla Bacheca).

Le catene di navigazione iniziali di ciascun sequence sono già riportate, messaggio per messaggio,
nella sezione 2.

---

## 1. DIAGRAMMI DEI CASI D'USO (7)

Per **ogni** sottosistema crea un nuovo *Use Case Diagram*:
1. Menu → *Diagram* → *Create UseCase Diagram*, nominalo come il sottosistema.
2. Disegna un rettangolo (confine del sistema/sottosistema): trascina un *Package* o usa il
   rettangolo di sistema; scrivi il nome del sottosistema in alto a sinistra.
3. Posiziona gli **attori a sinistra** (chi inizia) e **a destra** (DBMS, provider).
4. Inserisci gli **ovali dei casi d'uso** dentro il rettangolo.
5. Collega attori e casi d'uso con **associazioni** (linea continua).
6. Tra i casi d'uso usa frecce **tratteggiate** con stereotipo `<<include>>` dove indicato
   (freccia che punta al caso d'uso incluso).
7. La generalizzazione tra attori (Studente/Revisore → Utente) è una freccia con **triangolo
   vuoto** che punta all'attore generico.

Di seguito il contenuto esatto di ciascun diagramma.

### 1.A — Autenticazione
- **Attori sinistra:** Studente, Revisore, Utente. **Generalizzazioni:** Studente ▷ Utente, Revisore ▷ Utente.
- **Attori destra:** DBMS, Mail Provider, Provider SPID/eIDAS.
- **Casi d'uso:** Registrazione, Login, Verifica 2FA, Accesso con SPID/eIDAS, Recupero credenziali, Logout.
- **Associazioni attore→UC:** Studente→Registrazione; Utente→Login; Utente→Verifica 2FA; Studente→Accesso con SPID/eIDAS; Studente→Recupero credenziali; Utente→Logout.
- **`<<include>>`:** Login ⇢ Verifica 2FA; Recupero credenziali ⇢ Verifica 2FA; Registrazione ⇢ Verifica 2FA.
- **UC→attore destra:** tutti i sei UC → DBMS; Verifica 2FA e Recupero credenziali → Mail Provider; Accesso con SPID/eIDAS → Provider SPID/eIDAS.

### 1.B — Gestione Profilo
- **Attori sinistra:** Studente. **Attori destra:** DBMS, Provider AFAM.
- **Casi d'uso:** Visualizza profilo (Dashboard), Modifica profilo (dati e avatar), Federazione AFAM.
- **Associazioni:** Studente→tutti e tre. Tutti e tre → DBMS. Federazione AFAM → Provider AFAM.

### 1.C — Gestione Contenuti
- **Attori sinistra:** Studente. **Attori destra:** DBMS.
- **Casi d'uso:** Carica contenuto, Modifica contenuto, Elimina contenuto, Imposta thumbnail video.
- **Associazioni:** Studente→tutti; tutti→DBMS.

### 1.D — Gestione Portfolio
- **Attori sinistra:** Studente. **Attori destra:** DBMS.
- **Casi d'uso:** Crea portfolio, Modifica portfolio, Elimina portfolio.
- **Associazioni:** Studente→tutti; tutti→DBMS.

### 1.E — Gestione Condivisione
- **Attori sinistra:** Studente. **Attori destra:** DBMS.
- **Casi d'uso:** Genera link condivisione, Visualizza notifiche e storico.
- **Associazioni:** Studente→entrambi; entrambi→DBMS.

### 1.F — Consultazione Pubblica
- **Attori sinistra:** Utente Esterno. **Attori destra:** DBMS.
- **Casi d'uso:** Bacheca pubblica, Ricerca studenti, Profilo pubblico, Accesso tramite link condiviso.
- **Associazioni:** Utente Esterno→tutti; tutti→DBMS.

### 1.G — Segnalazioni
- **Attori sinistra:** Utente Esterno, Revisore. **Attori destra:** DBMS.
- **Casi d'uso:** Invia segnalazione, Revisione segnalazioni.
- **Associazioni:** Utente Esterno→Invia segnalazione; Revisore→Revisione segnalazioni; entrambi→DBMS.

---


---

## 2. SEQUENCE DIAGRAM — ISTRUZIONI PASSO-PASSO (24)

Per ogni caso d'uso: *Diagram → Create Sequence Diagram*, nominato come l'UC. Prima crea tutte le **lifeline** nell'ordine indicato (con lo stereotipo giusto), poi inserisci i **messaggi** UNO A UNO nell'ordine numerato. Le indicazioni «[apri frame …]» e «[chiudi frame]» dicono quando creare i *Combined Fragment*. Ricorda: i messaggi sono **sincroni** (freccia piena); i `<<create>>` sono messaggi tratteggiati con stereotipo *create*.

**Come leggere le indicazioni sui frame.** In Astah un *Combined Fragment* è un riquadro che
racchiude uno o più messaggi. Quando trovi «[apri frame `loop`]», seleziona i messaggi che
seguono fino al relativo «[chiudi frame]» e racchiudili in un frammento `loop` con quella guardia.
Un «[apri frame `alt`]» seguito più avanti da «[secondo operando dell'`alt`]» indica un unico
riquadro `alt` diviso in due scomparti (due guardie). **I frame possono essere annidati**: se un
«[apri frame `alt`]» compare dopo un «[apri frame `loop`]» ancora aperto, l'`alt` sta *dentro* il
`loop`; chiudi sempre prima il frame più interno. Ogni «[chiudi frame]» chiude il frammento aperto
più di recente (regola LIFO, come le parentesi).


### A. Autenticazione

#### Login
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:LoginBND` `<<boundary>>` · `:LoginCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**
1. `Utente` → `:BachecaPubblicaBND` : `clickAccedi()`
2. `:BachecaPubblicaBND` → `:LoginBND` : `mostraLogin()`
3. `Utente` → `:LoginBND` : `inserisci(email, password)`
4. `Utente` → `:LoginBND` : `clickAccedi()`
5. `:LoginBND` **crea** `:LoginCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
6. `:LoginCTRL` → *(auto-messaggio)* : `verificaFormato(email, password)`
7. `:LoginCTRL` → `:DBMSBND` : `queryCredenziali(email)`
8. `:DBMSBND` → `DBMS` : `queryCredenziali(email)`
- **[apri frame `loop`]** guardia: `[credenziali != corrispondono || formato errato]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
- **[apri frame `alt`]** primo operando, guardia: `[formato errato]`.
9. `:LoginCTRL` → `:ErroreBND` : `mostra("Formato non valido, riprova")`
10. `Utente` → `:ErroreBND` : `clickOK()`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
11. `:LoginCTRL` → `:ErroreBND` : `mostra("Le credenziali non corrispondono, riprova")`
12. `Utente` → `:ErroreBND` : `clickOK()`
- **[chiudi frame]**
13. `:LoginBND` → *(auto-messaggio)* : `mostraLogin()`
14. `Utente` → `:LoginBND` : `inserisci(email, password)`
15. `Utente` → `:LoginBND` : `clickAccedi()`
16. `:LoginCTRL` → *(auto-messaggio)* : `verificaFormato(email, password)`
17. `:LoginCTRL` → `:DBMSBND` : `queryCredenziali(email)`
18. `:DBMSBND` → `DBMS` : `queryCredenziali(email)`
- **[chiudi frame]**
- **[inserisci frame `ref`]** che copre Control e DBMSBND: richiama il caso d'uso «Verifica 2FA».
19. `:LoginCTRL` → `:MessaggioBND` : `mostra("Verifica riuscita")`
20. `Utente` → `:MessaggioBND` : `clickOK()`
21. `:LoginCTRL` → `:StudenteAFAM` : `getMatricola()`
22. `:LoginCTRL` → `:DBMSBND` : `insertSessione(matricola)`
23. `:DBMSBND` → `DBMS` : `insertSessione(matricola)`
24. `:LoginCTRL` → `:BachecaPubblicaBND` : `mostraDashboard()`

#### Verifica 2FA
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:LoginBND` `<<boundary>>` · `:Verifica2FABND` `<<boundary>>` · `:Verifica2FACTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Utente` → `:BachecaPubblicaBND` : `clickAccedi()` — *(l'utente è sulla bacheca e avvia l'accesso)*
2. `:BachecaPubblicaBND` → `:LoginBND` : `mostraLogin()`
3. `:LoginBND` → `:Verifica2FABND` : `mostraVerifica2FA()` — *(dopo l'inserimento delle credenziali il sistema mostra la verifica del secondo fattore)*
4. `Utente` → `:Verifica2FABND` : `inserisciCodice(codice)`
5. `Utente` → `:Verifica2FABND` : `clickVerifica()`
6. `:Verifica2FABND` **crea** `:Verifica2FACTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
7. `:Verifica2FACTRL` → `:StudenteAFAM` : `getMatricola()`
8. `:Verifica2FACTRL` → `:DBMSBND` : `querySegretoSecondoFattore(matricola)`
9. `:DBMSBND` → `DBMS` : `querySegretoSecondoFattore(matricola)`
10. `:Verifica2FACTRL` → *(auto-messaggio)* : `verificaCodice(codice, segreto)`
- **[apri frame `loop`]** guardia: `[codice non valido]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
11. `:Verifica2FACTRL` → `:ErroreBND` : `mostra("Codice errato")`
12. `Utente` → `:ErroreBND` : `clickOK()`
13. `Utente` → `:Verifica2FABND` : `inserisciCodice(codice)`
14. `Utente` → `:Verifica2FABND` : `clickVerifica()`
15. `:Verifica2FACTRL` → *(auto-messaggio)* : `verificaCodice(codice, segreto)`
- **[chiudi frame]**
16. `:Verifica2FACTRL` → `:MessaggioBND` : `mostra("Verifica riuscita")`

#### Registrazione
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:LoginBND` `<<boundary>>` · `:RegistrazioneBND` `<<boundary>>` · `:RegistrazioneCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**
1. `Studente` → `:BachecaPubblicaBND` : `clickAccedi()`
2. `:BachecaPubblicaBND` → `:LoginBND` : `mostraLogin()`
3. `Studente` → `:LoginBND` : `clickRegistrati()`
4. `:LoginBND` → `:RegistrazioneBND` : `mostraRegistrazione()`
5. `Studente` → `:RegistrazioneBND` : `compilaForm(matricola, nome, cognome, email, password)`
6. `Studente` → `:RegistrazioneBND` : `clickRegistrati()`
7. `:RegistrazioneBND` **crea** `:RegistrazioneCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
8. `:RegistrazioneCTRL` → *(auto-messaggio)* : `verificaFormatoPassword(password)`
9. `:RegistrazioneCTRL` → `:DBMSBND` : `checkRegistrato(matricola, email)`
10. `:DBMSBND` → `DBMS` : `checkRegistrato(matricola, email)`
- **[apri frame `alt`]** primo operando, guardia: `[utente non registrato]`.
- **[apri frame `loop`]** guardia: `[formato password non valido]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
11. `:RegistrazioneCTRL` → `:ErroreBND` : `mostra("Formato password non valido, riprova")`
12. `Studente` → `:ErroreBND` : `clickOK()`
13. `Studente` → `:RegistrazioneBND` : `inserisciPassword(password)`
14. `:RegistrazioneCTRL` → *(auto-messaggio)* : `verificaFormatoPassword(password)`
- **[chiudi frame]**
15. `:RegistrazioneCTRL` → `:StudenteAFAM` : `createStudente(matricola, nome, cognome, email)`
16. `:RegistrazioneCTRL` → `:DBMSBND` : `insertStudente(studente, credenzialeProtetta, segretoSecondoFattore)`
17. `:DBMSBND` → `DBMS` : `insertStudente(...)`
18. `:RegistrazioneCTRL` → `:MessaggioBND` : `mostra("Registrazione riuscita, configura la 2FA")`
19. `Studente` → `:MessaggioBND` : `clickOK()`
20. `:RegistrazioneCTRL` → `:LoginBND` : `mostraLogin()`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
21. `:RegistrazioneCTRL` → `:ErroreBND` : `mostra("Utente già registrato, effettua il login")`
22. `Studente` → `:ErroreBND` : `clickOK()`
23. `:RegistrazioneCTRL` → `:LoginBND` : `mostraLogin()`
- **[chiudi frame]**

#### Accesso con SPID/eIDAS
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:LoginBND` `<<boundary>>` · `:AccessoFederatoCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)* · `Provider SPID/eIDAS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickAccedi()` — *(lo studente è sulla bacheca e avvia l'accesso)*
2. `:BachecaPubblicaBND` → `:LoginBND` : `mostraLogin()`
3. `Studente` → `:LoginBND` : `clickAccediConSPID()`
4. `:LoginBND` **crea** `:AccessoFederatoCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
5. `:AccessoFederatoCTRL` → `Provider SPID/eIDAS` : `avviaAutenticazione()`
6. `:AccessoFederatoCTRL` ← `Provider SPID/eIDAS` : *ritorno* `attributiIdentita` (freccia tratteggiata).
7. `:AccessoFederatoCTRL` → `:DBMSBND` : `checkRegistrato(codiceFiscale)`
8. `:DBMSBND` → `DBMS` : `checkRegistrato(codiceFiscale)`
- **[apri frame `alt`]** primo operando, guardia: `[studente già registrato]`.
9. `:AccessoFederatoCTRL` → `:StudenteAFAM` : `getMatricola()`
10. `:AccessoFederatoCTRL` → `:DBMSBND` : `updateIdentitaFederata(matricola, attributi)`
11. `:DBMSBND` → `DBMS` : `updateIdentitaFederata(...)`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
12. `:AccessoFederatoCTRL` → `:StudenteAFAM` : `createStudente(attributi)`
13. `:AccessoFederatoCTRL` → `:DBMSBND` : `insertStudente(studente, identitaFederata)`
14. `:DBMSBND` → `DBMS` : `insertStudente(...)`
- **[chiudi frame]**
15. `:AccessoFederatoCTRL` → `:DBMSBND` : `insertSessione(matricola)`
16. `:DBMSBND` → `DBMS` : `insertSessione(matricola)`
17. `:AccessoFederatoCTRL` → `:MessaggioBND` : `mostra("Accesso effettuato con successo")`
18. `Studente` → `:MessaggioBND` : `clickOK()`
19. `:AccessoFederatoCTRL` → `:LoginBND` : `mostraDashboard()`

#### Recupero credenziali
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:LoginBND` `<<boundary>>` · `:RecuperoPasswordBND` `<<boundary>>` · `:RecuperoPasswordCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)* · `Mail Provider` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickAccedi()` — *(lo studente è sulla bacheca e avvia l'accesso)*
2. `:BachecaPubblicaBND` → `:LoginBND` : `mostraLogin()`
3. `Studente` → `:LoginBND` : `clickRecuperaPassword()`
4. `:LoginBND` → `:RecuperoPasswordBND` : `mostraRecupero()`
5. `Studente` → `:RecuperoPasswordBND` : `inserisci(email)`
6. `Studente` → `:RecuperoPasswordBND` : `clickInvia()`
7. `:RecuperoPasswordBND` **crea** `:RecuperoPasswordCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
8. `:RecuperoPasswordCTRL` → `:DBMSBND` : `queryStudente(email)`
9. `:DBMSBND` → `DBMS` : `queryStudente(email)`
- **[apri frame `alt`]** primo operando, guardia: `[account esistente]`.
10. `:RecuperoPasswordCTRL` → *(auto-messaggio)* : `generaTokenReset()`
11. `:RecuperoPasswordCTRL` → `:DBMSBND` : `insertTokenReset(token, matricola)`
12. `:DBMSBND` → `DBMS` : `insertTokenReset(...)`
13. `:RecuperoPasswordCTRL` → `Mail Provider` : `inviaEmail(linkReset)`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
- **[chiudi frame]**
14. `:RecuperoPasswordCTRL` → `:MessaggioBND` : `mostra("Se l'account esiste, riceverai un'email")`
15. `Studente` → `:MessaggioBND` : `clickOK()`
16. `:RecuperoPasswordCTRL` → `:LoginBND` : `mostraLogin()`

#### Logout
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:LogoutCTRL` `<<control>>`

**Messaggi (in ordine):**

1. `Utente` → `:BachecaPubblicaBND` : `clickEsci()` — *(l'utente, autenticato, clicca "Esci" nella navigazione della bacheca)*
2. `:BachecaPubblicaBND` **crea** `:LogoutCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
3. `:LogoutCTRL` → *(auto-messaggio)* : `invalidaSessione()`
4. `:LogoutCTRL` → `:BachecaPubblicaBND` : `mostraBachecaPubblica()`

#### Visualizza profilo
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:DashboardBND` `<<boundary>>` · `:ProfiloCTRL` `<<control>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickDashboard()` — *(lo studente, autenticato, è sulla bacheca e apre la Dashboard)*
2. `:BachecaPubblicaBND` → `:DashboardBND` : `mostraDashboard()`
3. `:DashboardBND` **crea** `:ProfiloCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
4. `:ProfiloCTRL` → `:StudenteAFAM` : `getMatricola()`
5. `:ProfiloCTRL` → `:DBMSBND` : `queryProfilo(matricola)`
6. `:DBMSBND` → `DBMS` : `queryProfilo(matricola)`
7. `:ProfiloCTRL` → `:DBMSBND` : `queryContenuti(matricola)`
8. `:DBMSBND` → `DBMS` : `queryContenuti(matricola)`
9. `:ProfiloCTRL` → `:DashboardBND` : `mostraDashboard(profilo, opere, portfolios)`

#### Modifica profilo
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:DashboardBND` `<<boundary>>` · `:ImpostazioniProfiloBND` `<<boundary>>` · `:ModificaProfiloCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickDashboard()` — *(lo studente, autenticato, apre la Dashboard dalla bacheca)*
2. `:BachecaPubblicaBND` → `:DashboardBND` : `mostraDashboard()`
3. `Studente` → `:DashboardBND` : `clickImpostazioni()`
4. `:DashboardBND` → `:ImpostazioniProfiloBND` : `mostraImpostazioni()`
5. `Studente` → `:ImpostazioniProfiloBND` : `modificaCampi(dati, avatar)`
6. `Studente` → `:ImpostazioniProfiloBND` : `clickSalva()`
7. `:ImpostazioniProfiloBND` **crea** `:ModificaProfiloCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
8. `:ModificaProfiloCTRL` → *(auto-messaggio)* : `verificaFormato(dati, avatar)`
- **[apri frame `loop`]** guardia: `[dati non validi]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
9. `:ModificaProfiloCTRL` → `:ErroreBND` : `mostra("Dati non validi, riprova")`
10. `Studente` → `:ErroreBND` : `clickOK()`
11. `Studente` → `:ImpostazioniProfiloBND` : `modificaCampi(dati, avatar)`
12. `Studente` → `:ImpostazioniProfiloBND` : `clickSalva()`
13. `:ModificaProfiloCTRL` → *(auto-messaggio)* : `verificaFormato(dati, avatar)`
- **[chiudi frame]**
14. `:ModificaProfiloCTRL` → `:StudenteAFAM` : `getMatricola()`
15. `:ModificaProfiloCTRL` → `:DBMSBND` : `updateProfilo(matricola, dati, avatar)`
16. `:DBMSBND` → `DBMS` : `updateProfilo(matricola, dati, avatar)`
17. `:ModificaProfiloCTRL` → `:MessaggioBND` : `mostra("Profilo aggiornato con successo")`
18. `Studente` → `:MessaggioBND` : `clickOK()`
19. `:ModificaProfiloCTRL` → `:ImpostazioniProfiloBND` : `mostraImpostazioni()`

#### Federazione AFAM
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:DashboardBND` `<<boundary>>` · `:ImpostazioniProfiloBND` `<<boundary>>` · `:FederazioneCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:ErroreBND` `<<boundary>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)* · `Provider AFAM` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickDashboard()` — *(lo studente, autenticato, apre la Dashboard dalla bacheca)*
2. `:BachecaPubblicaBND` → `:DashboardBND` : `mostraDashboard()`
3. `Studente` → `:DashboardBND` : `clickImpostazioni()`
4. `:DashboardBND` → `:ImpostazioniProfiloBND` : `mostraImpostazioni()`
5. `Studente` → `:ImpostazioniProfiloBND` : `clickCollegaIstituto()`
6. `Studente` → `:ImpostazioniProfiloBND` : `selezionaIstituto(idIstituto)`
7. `Studente` → `:ImpostazioniProfiloBND` : `clickConferma()`
8. `:ImpostazioniProfiloBND` **crea** `:FederazioneCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
9. `:FederazioneCTRL` → `:StudenteAFAM` : `getMatricola()`
10. `:FederazioneCTRL` → *(auto-messaggio)* : `generaIdFederato()`
11. `:FederazioneCTRL` → `Provider AFAM` : `trasmettiRiferimento(idFederato, metadati)`
12. `:FederazioneCTRL` ← `Provider AFAM` : *ritorno* `confermato` (freccia tratteggiata).
- **[apri frame `alt`]** primo operando, guardia: `[confermato]`.
13. `:FederazioneCTRL` → `:DBMSBND` : `insertIdentitaFederata(idFederato, matricola, idIstituto)`
14. `:DBMSBND` → `DBMS` : `insertIdentitaFederata(...)`
15. `:FederazioneCTRL` → `:MessaggioBND` : `mostra("Istituto collegato con successo")`
16. `Studente` → `:MessaggioBND` : `clickOK()`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
17. `:FederazioneCTRL` → `:ErroreBND` : `mostra("Servizio di federazione non disponibile")`
18. `Studente` → `:ErroreBND` : `clickOK()`
- **[chiudi frame]**
19. `:FederazioneCTRL` → `:ImpostazioniProfiloBND` : `mostraImpostazioni()`

#### Carica contenuto
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:ArchivioBND` `<<boundary>>` · `:CaricaContenutoCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:ContenutoMultimediale` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickArchivio()` — *(lo studente, autenticato, apre l'Archivio dalla navigazione della bacheca)*
2. `:BachecaPubblicaBND` → `:ArchivioBND` : `mostraArchivio()`
3. `Studente` → `:ArchivioBND` : `clickCaricaContenuto()`
4. `:ArchivioBND` **crea** `:CaricaContenutoCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
5. `Studente` → `:ArchivioBND` : `selezionaFile(file)`
6. `Studente` → `:ArchivioBND` : `inserisci(titolo, descrizione, visibilita)`
7. `Studente` → `:ArchivioBND` : `clickCarica()`
8. `:CaricaContenutoCTRL` → *(auto-messaggio)* : `verificaFile(file)`
- **[apri frame `loop`]** guardia: `[file non valido]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
9. `:CaricaContenutoCTRL` → `:ErroreBND` : `mostra("File non valido, riprova")`
10. `Studente` → `:ErroreBND` : `clickOK()`
11. `Studente` → `:ArchivioBND` : `selezionaFile(file)`
12. `:CaricaContenutoCTRL` → *(auto-messaggio)* : `verificaFile(file)`
- **[chiudi frame]**
13. `:CaricaContenutoCTRL` → `:StudenteAFAM` : `getMatricola()`
14. `:CaricaContenutoCTRL` → `:ContenutoMultimediale` : `createContenuto(titolo, descrizione, visibilita)`
15. `:CaricaContenutoCTRL` → `:DBMSBND` : `insertContenuto(contenuto, matricola)`
16. `:DBMSBND` → `DBMS` : `insertContenuto(contenuto, matricola)`
17. `:CaricaContenutoCTRL` → `:MessaggioBND` : `mostra("Contenuto caricato")`
18. `Studente` → `:MessaggioBND` : `clickOK()`
19. `:CaricaContenutoCTRL` → `:ArchivioBND` : `mostraArchivio()`

#### Modifica contenuto
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:ArchivioBND` `<<boundary>>` · `:ModificaContenutoCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:ContenutoMultimediale` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickArchivio()` — *(lo studente apre l'Archivio dalla bacheca)*
2. `:BachecaPubblicaBND` → `:ArchivioBND` : `mostraArchivio()`
3. `Studente` → `:ArchivioBND` : `clickModifica(idContenuto)`
4. `:ArchivioBND` **crea** `:ModificaContenutoCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
5. `Studente` → `:ArchivioBND` : `modificaCampi(titolo, descrizione, visibilita)`
6. `Studente` → `:ArchivioBND` : `clickSalva()`
7. `:ModificaContenutoCTRL` → *(auto-messaggio)* : `verificaFormato(campi)`
- **[apri frame `loop`]** guardia: `[campi non validi]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
8. `:ModificaContenutoCTRL` → `:ErroreBND` : `mostra("Dati non validi, riprova")`
9. `Studente` → `:ErroreBND` : `clickOK()`
10. `Studente` → `:ArchivioBND` : `modificaCampi(titolo, descrizione, visibilita)`
11. `:ModificaContenutoCTRL` → *(auto-messaggio)* : `verificaFormato(campi)`
- **[chiudi frame]**
12. `:ModificaContenutoCTRL` → `:StudenteAFAM` : `getMatricola()`
13. `:ModificaContenutoCTRL` → `:DBMSBND` : `queryContenuto(idContenuto, matricola)`
14. `:DBMSBND` → `DBMS` : `queryContenuto(idContenuto, matricola)`
15. `:ModificaContenutoCTRL` → `:ContenutoMultimediale` : `setTitolo() / setDescrizione() / setVisibilita()`
16. `:ModificaContenutoCTRL` → `:DBMSBND` : `updateContenuto(idContenuto, campi)`
17. `:DBMSBND` → `DBMS` : `updateContenuto(idContenuto, campi)`
18. `:ModificaContenutoCTRL` → `:MessaggioBND` : `mostra("Contenuto aggiornato con successo")`
19. `Studente` → `:MessaggioBND` : `clickOK()`
20. `:ModificaContenutoCTRL` → `:ArchivioBND` : `mostraArchivio()`

#### Elimina contenuto
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:ArchivioBND` `<<boundary>>` · `:EliminaContenutoCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:ContenutoMultimediale` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickArchivio()` — *(lo studente apre l'Archivio dalla bacheca)*
2. `:BachecaPubblicaBND` → `:ArchivioBND` : `mostraArchivio()`
3. `Studente` → `:ArchivioBND` : `clickElimina(idContenuto)`
4. `:ArchivioBND` **crea** `:EliminaContenutoCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
5. `:EliminaContenutoCTRL` → `:MessaggioBND` : `mostra("Sei sicuro di voler eliminare il contenuto?")`
6. `Studente` → `:MessaggioBND` : `clickConferma()`
7. `:EliminaContenutoCTRL` → `:StudenteAFAM` : `getMatricola()`
8. `:EliminaContenutoCTRL` → `:DBMSBND` : `queryContenuto(idContenuto, matricola)`
9. `:DBMSBND` → `DBMS` : `queryContenuto(idContenuto, matricola)`
10. `:EliminaContenutoCTRL` → `:ContenutoMultimediale` : `setVisibilita(RIMOSSO)`
11. `:EliminaContenutoCTRL` → `:DBMSBND` : `updateContenuto(idContenuto, RIMOSSO)`
12. `:DBMSBND` → `DBMS` : `updateContenuto(idContenuto, RIMOSSO)`
13. `:EliminaContenutoCTRL` → `:MessaggioBND` : `mostra("Contenuto eliminato")`
14. `Studente` → `:MessaggioBND` : `clickOK()`
15. `:EliminaContenutoCTRL` → `:ArchivioBND` : `mostraArchivio()`

#### Imposta thumbnail video
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:ArchivioBND` `<<boundary>>` · `:ThumbnailCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:ContenutoMultimediale` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickArchivio()` — *(lo studente apre l'Archivio dalla bacheca)*
2. `:BachecaPubblicaBND` → `:ArchivioBND` : `mostraArchivio()`
3. `Studente` → `:ArchivioBND` : `apriPannelloVideo(idContenuto)`
- **[apri frame `alt`]** primo operando, guardia: `[carica immagine]`.
4. `Studente` → `:ArchivioBND` : `selezionaImmagine(file)`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
5. `:ArchivioBND` → *(auto-messaggio)* : `catturaFotogramma()`
- **[chiudi frame]**
6. `:ArchivioBND` **crea** `:ThumbnailCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
7. `:ThumbnailCTRL` → *(auto-messaggio)* : `verificaImmagine(file)`
8. `:ThumbnailCTRL` → `:StudenteAFAM` : `getMatricola()`
9. `:ThumbnailCTRL` → `:DBMSBND` : `queryContenuto(idContenuto, matricola)`
10. `:DBMSBND` → `DBMS` : `queryContenuto(idContenuto, matricola)`
11. `:ThumbnailCTRL` → `:ContenutoMultimediale` : `setThumbnail(path)`
12. `:ThumbnailCTRL` → `:DBMSBND` : `updateThumbnail(idContenuto, path)`
13. `:DBMSBND` → `DBMS` : `updateThumbnail(idContenuto, path)`
14. `:ThumbnailCTRL` → `:MessaggioBND` : `mostra("Anteprima impostata")`
15. `Studente` → `:MessaggioBND` : `clickOK()`
16. `:ThumbnailCTRL` → `:ArchivioBND` : `mostraArchivio()`

#### Crea portfolio
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:GestionePortfolioBND` `<<boundary>>` · `:CreaPortfolioBND` `<<boundary>>` · `:CreaPortfolioCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:Portfolio` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickPortfolio()` — *(lo studente apre la Gestione portfolio dalla navigazione della bacheca)*
2. `:BachecaPubblicaBND` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`
3. `Studente` → `:GestionePortfolioBND` : `clickNuovoPortfolio()`
4. `:GestionePortfolioBND` → `:CreaPortfolioBND` : `mostraForm()`
5. `Studente` → `:CreaPortfolioBND` : `inserisci(titolo)`
6. `Studente` → `:CreaPortfolioBND` : `selezionaContenuti(ids, ordine)`
7. `Studente` → `:CreaPortfolioBND` : `selezionaVisibilita(visibilita)`
8. `Studente` → `:CreaPortfolioBND` : `clickCrea()`
9. `:CreaPortfolioBND` **crea** `:CreaPortfolioCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
10. `:CreaPortfolioCTRL` → *(auto-messaggio)* : `verificaDati(titolo, ids)`
- **[apri frame `loop`]** guardia: `[dati non validi]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
11. `:CreaPortfolioCTRL` → `:ErroreBND` : `mostra("Inserisci un titolo e almeno un contenuto")`
12. `Studente` → `:ErroreBND` : `clickOK()`
13. `Studente` → `:CreaPortfolioBND` : `inserisci(titolo)`
14. `Studente` → `:CreaPortfolioBND` : `selezionaContenuti(ids, ordine)`
15. `:CreaPortfolioCTRL` → *(auto-messaggio)* : `verificaDati(titolo, ids)`
- **[chiudi frame]**
16. `:CreaPortfolioCTRL` → `:StudenteAFAM` : `getMatricola()`
17. `:CreaPortfolioCTRL` → `:Portfolio` : `createPortfolio(titolo, ids, ordine, visibilita)`
18. `:CreaPortfolioCTRL` → `:DBMSBND` : `insertPortfolio(portfolio, matricola)`
19. `:DBMSBND` → `DBMS` : `insertPortfolio(portfolio, matricola)`
20. `:CreaPortfolioCTRL` → `:MessaggioBND` : `mostra("Portfolio creato con successo")`
21. `Studente` → `:MessaggioBND` : `clickOK()`
22. `:CreaPortfolioCTRL` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`

#### Modifica portfolio
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:GestionePortfolioBND` `<<boundary>>` · `:ModificaPortfolioBND` `<<boundary>>` · `:ModificaPortfolioCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:Portfolio` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickPortfolio()` — *(lo studente apre la Gestione portfolio dalla bacheca)*
2. `:BachecaPubblicaBND` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`
3. `Studente` → `:GestionePortfolioBND` : `clickModifica(idPortfolio)`
4. `:GestionePortfolioBND` **crea** `:ModificaPortfolioCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
5. `:ModificaPortfolioCTRL` → `:StudenteAFAM` : `getMatricola()`
6. `:ModificaPortfolioCTRL` → `:DBMSBND` : `queryPortfolio(idPortfolio, matricola)`
7. `:DBMSBND` → `DBMS` : `queryPortfolio(idPortfolio, matricola)`
8. `:ModificaPortfolioCTRL` → `:ModificaPortfolioBND` : `mostraForm(portfolio, contenuti)`
9. `Studente` → `:ModificaPortfolioBND` : `modifica(titolo, ids, ordine, visibilita)`
10. `Studente` → `:ModificaPortfolioBND` : `clickSalva()`
11. `:ModificaPortfolioCTRL` → *(auto-messaggio)* : `verificaDati(titolo, ids)`
- **[apri frame `loop`]** guardia: `[dati non validi]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
12. `:ModificaPortfolioCTRL` → `:ErroreBND` : `mostra("Inserisci un titolo e almeno un contenuto")`
13. `Studente` → `:ErroreBND` : `clickOK()`
14. `Studente` → `:ModificaPortfolioBND` : `modifica(titolo, ids, ordine, visibilita)`
15. `:ModificaPortfolioCTRL` → *(auto-messaggio)* : `verificaDati(titolo, ids)`
- **[chiudi frame]**
16. `:ModificaPortfolioCTRL` → `:Portfolio` : `setTitolo() / setContenuti() / setVisibilita()`
17. `:ModificaPortfolioCTRL` → `:DBMSBND` : `updatePortfolio(idPortfolio, titolo, ids, ordine, visibilita)`
18. `:DBMSBND` → `DBMS` : `updatePortfolio(...)`
19. `:ModificaPortfolioCTRL` → `:MessaggioBND` : `mostra("Portfolio aggiornato con successo")`
20. `Studente` → `:MessaggioBND` : `clickOK()`
21. `:ModificaPortfolioCTRL` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`

#### Elimina portfolio
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:GestionePortfolioBND` `<<boundary>>` · `:EliminaPortfolioCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:Portfolio` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickPortfolio()` — *(lo studente apre la Gestione portfolio dalla bacheca)*
2. `:BachecaPubblicaBND` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`
3. `Studente` → `:GestionePortfolioBND` : `clickElimina(idPortfolio)`
4. `:GestionePortfolioBND` **crea** `:EliminaPortfolioCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
5. `:EliminaPortfolioCTRL` → `:MessaggioBND` : `mostra("Sei sicuro di voler eliminare il portfolio?")`
6. `Studente` → `:MessaggioBND` : `clickConferma()`
7. `:EliminaPortfolioCTRL` → `:StudenteAFAM` : `getMatricola()`
8. `:EliminaPortfolioCTRL` → `:DBMSBND` : `queryPortfolio(idPortfolio, matricola)`
9. `:DBMSBND` → `DBMS` : `queryPortfolio(idPortfolio, matricola)`
10. `:EliminaPortfolioCTRL` → `:DBMSBND` : `deletePortfolio(idPortfolio)`
11. `:DBMSBND` → `DBMS` : `deletePortfolio(idPortfolio)`
12. `:EliminaPortfolioCTRL` → `:MessaggioBND` : `mostra("Portfolio eliminato")`
13. `Studente` → `:MessaggioBND` : `clickOK()`
14. `:EliminaPortfolioCTRL` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`

#### Genera link condivisione
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:GestionePortfolioBND` `<<boundary>>` · `:GeneraLinkCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:LinkCondivisione` `<<entity>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickPortfolio()` — *(lo studente apre la Gestione portfolio dalla bacheca)*
2. `:BachecaPubblicaBND` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`
3. `Studente` → `:GestionePortfolioBND` : `clickCondividi(idPortfolio)`
4. `:GestionePortfolioBND` **crea** `:GeneraLinkCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
5. `:GeneraLinkCTRL` → `:StudenteAFAM` : `getMatricola()`
6. `:GeneraLinkCTRL` → `:DBMSBND` : `queryPortfolio(idPortfolio, matricola)`
7. `:DBMSBND` → `DBMS` : `queryPortfolio(idPortfolio, matricola)`
8. `:GeneraLinkCTRL` → *(auto-messaggio)* : `generaTokenUnivoco()`
9. `:GeneraLinkCTRL` → `:LinkCondivisione` : `createLink(token, idPortfolio, dataScadenza)`
10. `:GeneraLinkCTRL` → `:DBMSBND` : `insertLink(link, matricola)`
11. `:DBMSBND` → `DBMS` : `insertLink(link, matricola)`
12. `:GeneraLinkCTRL` → `:MessaggioBND` : `mostra("Link generato", url)`
13. `Studente` → `:MessaggioBND` : `clickCopiaLink()`
14. `:GeneraLinkCTRL` → `:MessaggioBND` : `mostra("Link copiato")`
15. `Studente` → `:MessaggioBND` : `clickOK()`
16. `:GeneraLinkCTRL` → `:GestionePortfolioBND` : `mostraGestionePortfolio()`

#### Visualizza notifiche e storico
**Lifeline (in quest'ordine, da sinistra a destra):**
`Studente` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:NotificheBND` `<<boundary>>` · `:NotificheCTRL` `<<control>>` · `:StudenteAFAM` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Studente` → `:BachecaPubblicaBND` : `clickNotifiche()` — *(lo studente apre le notifiche dall'icona nella navigazione della bacheca)*
2. `:BachecaPubblicaBND` → `:NotificheBND` : `mostraNotifiche()`
3. `:NotificheBND` **crea** `:NotificheCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
4. `:NotificheCTRL` → `:StudenteAFAM` : `getMatricola()`
5. `:NotificheCTRL` → `:DBMSBND` : `queryNotificheNonLette(matricola)`
6. `:DBMSBND` → `DBMS` : `queryNotificheNonLette(matricola)`
7. `:NotificheCTRL` → `:NotificheBND` : `mostraNotifiche(elenco)`
- **[apri frame `alt`]** primo operando, guardia: `[notifiche non lette presenti]`.
8. `Studente` → `:NotificheBND` : `clickSegnaLette()`
9. `:NotificheCTRL` → `:DBMSBND` : `updateNotificheLette(matricola)`
10. `:DBMSBND` → `DBMS` : `updateNotificheLette(matricola)`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
- **[chiudi frame]**
11. `Studente` → `:NotificheBND` : `clickStoricoCondivisioni()`
12. `:NotificheCTRL` → `:DBMSBND` : `queryStoricoLink(matricola)`
13. `:DBMSBND` → `DBMS` : `queryStoricoLink(matricola)`
14. `:NotificheCTRL` → `:NotificheBND` : `mostraStorico(link)`

#### Bacheca pubblica
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente Esterno` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:BachecaCTRL` `<<control>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**
1. `Utente Esterno` → `:BachecaPubblicaBND` : `apriSistema()`
2. `:BachecaPubblicaBND` **crea** `:BachecaCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
3. `:BachecaCTRL` → `:DBMSBND` : `queryContenutiPubblici()`
4. `:DBMSBND` → `DBMS` : `queryContenutiPubblici()`
5. `:BachecaCTRL` → `:BachecaPubblicaBND` : `mostraBacheca(contenuti)`
6. `Utente Esterno` → `:BachecaPubblicaBND` : `clickContenuto(idContenuto)`
7. `:BachecaPubblicaBND` → *(auto-messaggio)* : `apriAnteprima(idContenuto)`

#### Ricerca studenti
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente Esterno` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:RicercaCTRL` `<<control>>` · `:ProfiloPubblicoBND` `<<boundary>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**
1. `Utente Esterno` → `:BachecaPubblicaBND` : `clickRicerca()`
2. `:BachecaPubblicaBND` **crea** `:RicercaCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
3. `Utente Esterno` → `:BachecaPubblicaBND` : `digita(query)`
- **[apri frame `loop`]** guardia: `[utente continua a digitare]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
4. `:RicercaCTRL` → `:DBMSBND` : `queryProfili(query)`
5. `:DBMSBND` → `DBMS` : `queryProfili(query)`
6. `:RicercaCTRL` → `:BachecaPubblicaBND` : `aggiornaRisultati(profili)`
7. `Utente Esterno` → `:BachecaPubblicaBND` : `digita(query)`
- **[chiudi frame]**
8. `Utente Esterno` → `:BachecaPubblicaBND` : `clickProfilo(matricola)`
9. `:RicercaCTRL` → `:ProfiloPubblicoBND` : `mostraProfiloPubblico(matricola)`

#### Profilo pubblico
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente Esterno` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:ProfiloPubblicoBND` `<<boundary>>` · `:ProfiloPubblicoCTRL` `<<control>>` · `:VisualizzatorePortfolioBND` `<<boundary>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Utente Esterno` → `:BachecaPubblicaBND` : `clickProfilo(matricola)` — *(l'utente esterno, dalla bacheca, apre il profilo pubblico di uno studente)*
2. `:BachecaPubblicaBND` → `:ProfiloPubblicoBND` : `mostraProfiloPubblico(matricola)`
3. `:ProfiloPubblicoBND` **crea** `:ProfiloPubblicoCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
4. `:ProfiloPubblicoCTRL` → `:DBMSBND` : `queryProfiloPubblico(matricola)`
5. `:DBMSBND` → `DBMS` : `queryProfiloPubblico(matricola)`
6. `:ProfiloPubblicoCTRL` → `:DBMSBND` : `queryOperePubbliche(matricola)`
7. `:DBMSBND` → `DBMS` : `queryOperePubbliche(matricola)`
8. `:ProfiloPubblicoCTRL` → `:DBMSBND` : `queryPortfolioPubblici(matricola)`
9. `:DBMSBND` → `DBMS` : `queryPortfolioPubblici(matricola)`
10. `:ProfiloPubblicoCTRL` → `:ProfiloPubblicoBND` : `mostraProfilo(profilo, opere, portfolios)`
11. `Utente Esterno` → `:ProfiloPubblicoBND` : `clickPortfolio(idPortfolio)`
12. `:ProfiloPubblicoCTRL` → `:DBMSBND` : `queryContenutiPortfolio(idPortfolio)`
13. `:DBMSBND` → `DBMS` : `queryContenutiPortfolio(idPortfolio)`
14. `:ProfiloPubblicoCTRL` → `:VisualizzatorePortfolioBND` : `mostraVisualizzatorePortfolio(contenuti)`

#### Accesso tramite link condiviso
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente Esterno` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:VisualizzatoreLinkBND` `<<boundary>>` · `:AccessoLinkCTRL` `<<control>>` · `:LinkCondivisione` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Utente Esterno` → `:BachecaPubblicaBND` : `apriLinkCondiviso(token)` — *(l'utente esterno raggiunge il sistema tramite il link ricevuto; il punto d'ingresso resta la bacheca)*
2. `:BachecaPubblicaBND` → `:VisualizzatoreLinkBND` : `mostraVisualizzatoreLink(token)`
3. `:VisualizzatoreLinkBND` **crea** `:AccessoLinkCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
4. `:AccessoLinkCTRL` → `:DBMSBND` : `queryLink(token)`
5. `:DBMSBND` → `DBMS` : `queryLink(token)`
6. `:AccessoLinkCTRL` → `:LinkCondivisione` : `validaToken()`
- **[apri frame `alt`]** primo operando, guardia: `[link valido e non scaduto]`.
7. `:AccessoLinkCTRL` → `:LinkCondivisione` : `incrementaVisualizzazioni()`
8. `:AccessoLinkCTRL` → `:DBMSBND` : `updateVisualizzazioni(token)`
9. `:DBMSBND` → `DBMS` : `updateVisualizzazioni(token)`
10. `:AccessoLinkCTRL` → `:DBMSBND` : `insertNotifica(matricolaProprietario)`
11. `:DBMSBND` → `DBMS` : `insertNotifica(matricolaProprietario)`
12. `:AccessoLinkCTRL` → `:DBMSBND` : `queryContenutiPortfolio(idPortfolio)`
13. `:DBMSBND` → `DBMS` : `queryContenutiPortfolio(idPortfolio)`
14. `:AccessoLinkCTRL` → `:VisualizzatoreLinkBND` : `mostraContenutiCondivisi(contenuti)`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
15. `:AccessoLinkCTRL` → `:VisualizzatoreLinkBND` : `mostra("Link non valido o scaduto")`
16. `:AccessoLinkCTRL` → `:BachecaPubblicaBND` : `mostraBachecaPubblica()`
- **[chiudi frame]**

#### Invia segnalazione
**Lifeline (in quest'ordine, da sinistra a destra):**
`Utente Esterno` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:ProfiloPubblicoBND` `<<boundary>>` · `:FormSegnalazioneBND` `<<boundary>>` · `:SegnalazioneCTRL` `<<control>>` · `:ErroreBND` `<<boundary>>` · `:MessaggioBND` `<<boundary>>` · `:Segnalazione` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Utente Esterno` → `:BachecaPubblicaBND` : `clickProfilo(matricola)` — *(l'utente esterno, dalla bacheca, apre il profilo pubblico dove si trova il contenuto)*
2. `:BachecaPubblicaBND` → `:ProfiloPubblicoBND` : `mostraProfiloPubblico(matricola)`
3. `Utente Esterno` → `:ProfiloPubblicoBND` : `clickSegnala(idContenuto)`
4. `:ProfiloPubblicoBND` → `:FormSegnalazioneBND` : `mostraForm()`
5. `Utente Esterno` → `:FormSegnalazioneBND` : `selezionaMotivazione(motivazione)`
6. `Utente Esterno` → `:FormSegnalazioneBND` : `inserisciDescrizione(descrizione)`
7. `Utente Esterno` → `:FormSegnalazioneBND` : `clickInvia()`
8. `:FormSegnalazioneBND` **crea** `:SegnalazioneCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
9. `:SegnalazioneCTRL` → *(auto-messaggio)* : `verificaMotivazione(motivazione)`
- **[apri frame `loop`]** guardia: `[motivazione non selezionata]` — tutti i messaggi seguenti fino a «chiudi frame» si ripetono.
10. `:SegnalazioneCTRL` → `:ErroreBND` : `mostra("Seleziona una motivazione")`
11. `Utente Esterno` → `:ErroreBND` : `clickOK()`
12. `Utente Esterno` → `:FormSegnalazioneBND` : `selezionaMotivazione(motivazione)`
13. `:SegnalazioneCTRL` → *(auto-messaggio)* : `verificaMotivazione(motivazione)`
- **[chiudi frame]**
14. `:SegnalazioneCTRL` → `:Segnalazione` : `createSegnalazione(idContenuto, motivazione, descrizione, IN_ATTESA)`
15. `:SegnalazioneCTRL` → `:DBMSBND` : `insertSegnalazione(segnalazione)`
16. `:DBMSBND` → `DBMS` : `insertSegnalazione(segnalazione)`
17. `:SegnalazioneCTRL` → `:MessaggioBND` : `mostra("Segnalazione inviata con successo")`
18. `Utente Esterno` → `:MessaggioBND` : `clickOK()`
19. `:SegnalazioneCTRL` → `:ProfiloPubblicoBND` : `mostraSchermataPrecedente()`

#### Revisione segnalazioni
**Lifeline (in quest'ordine, da sinistra a destra):**
`Revisore` *(attore)* · `:BachecaPubblicaBND` `<<boundary>>` · `:PannelloRevisioneBND` `<<boundary>>` · `:RevisioneCTRL` `<<control>>` · `:MessaggioBND` `<<boundary>>` · `:Segnalazione` `<<entity>>` · `:ContenutoMultimediale` `<<entity>>` · `:DBMSBND` `<<boundary>>` · `DBMS` *(attore)*

**Messaggi (in ordine):**

1. `Revisore` → `:BachecaPubblicaBND` : `clickPannelloRevisione()` — *(il revisore, autenticato, apre il pannello di revisione dalla navigazione della bacheca)*
2. `:BachecaPubblicaBND` → `:PannelloRevisioneBND` : `mostraPannelloRevisione()`
3. `:PannelloRevisioneBND` **crea** `:RevisioneCTRL` — messaggio `<<create>>` (tratteggiato, stereotipo *create*).
4. `:RevisioneCTRL` → `:DBMSBND` : `querySegnalazioniPendenti()`
5. `:DBMSBND` → `DBMS` : `querySegnalazioniPendenti()`
6. `:RevisioneCTRL` → `:PannelloRevisioneBND` : `mostraPendenti(segnalazioni)`
7. `Revisore` → `:PannelloRevisioneBND` : `selezionaSegnalazione(idSegnalazione)`
8. `:RevisioneCTRL` → `:PannelloRevisioneBND` : `mostraDettaglio(segnalazione, contenuto)`
- **[apri frame `alt`]** primo operando, guardia: `[segnalazione fondata]`.
9. `Revisore` → `:PannelloRevisioneBND` : `clickAccogli()`
10. `:RevisioneCTRL` → `:ContenutoMultimediale` : `setVisibilita(RIMOSSO)`
11. `:RevisioneCTRL` → `:DBMSBND` : `updateContenuto(idContenuto, RIMOSSO)`
12. `:DBMSBND` → `DBMS` : `updateContenuto(idContenuto, RIMOSSO)`
13. `:RevisioneCTRL` → `:Segnalazione` : `setStato(ACCOLTA)`
14. `:RevisioneCTRL` → `:DBMSBND` : `updateSegnalazione(idSegnalazione, ACCOLTA)`
15. `:DBMSBND` → `DBMS` : `updateSegnalazione(idSegnalazione, ACCOLTA)`
- **[secondo operando dell'`alt`]** guardia: `[else]`.
16. `Revisore` → `:PannelloRevisioneBND` : `clickArchivia()`
17. `:RevisioneCTRL` → `:Segnalazione` : `setStato(ARCHIVIATA)`
18. `:RevisioneCTRL` → `:DBMSBND` : `updateSegnalazione(idSegnalazione, ARCHIVIATA)`
19. `:DBMSBND` → `DBMS` : `updateSegnalazione(idSegnalazione, ARCHIVIATA)`
- **[chiudi frame]**
20. `:RevisioneCTRL` → `:PannelloRevisioneBND` : `aggiornaPendenti()`
21. `Revisore` → `:PannelloRevisioneBND` : `clickStorico()`
22. `:RevisioneCTRL` → `:DBMSBND` : `querySegnalazioniRisolte()`
23. `:DBMSBND` → `DBMS` : `querySegnalazioniRisolte()`
24. `:RevisioneCTRL` → `:PannelloRevisioneBND` : `mostraStorico(segnalazioni)`

## 3. DIAGRAMMI DELLE CLASSI (8)

Per ogni sottosistema crea un *Class Diagram* e uno complessivo per le Entities. Ogni classe porta lo stereotipo (`<<boundary>>`, `<<control>>`, `<<entity>>`) e l'insieme COMPLETO delle funzioni usate nel sottosistema.

**Procedura passo-passo (per ogni diagramma):**
1. *Diagram → Create Class Diagram*, nominalo come il sottosistema.
2. Per **ogni classe** elencata sotto: trascina una *Class* dalla toolbar, scrivi il nome, poi applica lo stereotipo (tasto destro → *Stereotype* → `boundary`/`control`/`entity`).
3. Per le **entity**, aggiungi gli **attributi** (tasto destro sulla classe → *Add Attribute*, uno per riga).
4. Per **ogni classe**, aggiungi le **funzioni** elencate (tasto destro → *Add Operation*, una per riga, esattamente come scritte).
5. Traccia le **dipendenze** (freccia tratteggiata a punta aperta, tipo *Dependency* `..>`) nell'ordine: ogni Boundary → il suo Control; ogni Control → le Entity che usa; ogni Control → `DBMSBND`. L'elenco esatto è in fondo a ciascun sottosistema («Dipendenze»).
6. Nel diagramma **Entities**, invece delle dipendenze traccia le **associazioni** con le molteplicità indicate («Relazioni»).


### 3.A — Autenticazione

**BachecaPubblicaBND** `<<boundary>>`
- Funzioni: mostraBachecaPubblica(), clickAccedi(), clickRicercaProfili()

**LoginBND** `<<boundary>>`
- Funzioni: mostraLogin(), inserisci(email, password), clickAccedi(), clickAccediConSPID(), clickRecuperaPassword(), clickRegistrati(), mostraDashboard()

**RegistrazioneBND** `<<boundary>>`
- Funzioni: mostraRegistrazione(), compilaForm(), inserisciPassword(password), clickRegistrati()

**Verifica2FABND** `<<boundary>>`
- Funzioni: mostraVerifica2FA(), inserisciCodice(codice), clickVerifica()

**RecuperoPasswordBND** `<<boundary>>`
- Funzioni: mostraRecupero(), inserisci(email), clickInvia()

**MessaggioBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**ErroreBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**LoginCTRL** `<<control>>`
- Funzioni: createLogin(), verificaFormato(), checkRuolo()

**Verifica2FACTRL** `<<control>>`
- Funzioni: createVerifica2FA(), verificaCodice()

**RegistrazioneCTRL** `<<control>>`
- Funzioni: createRegistrazione(), verificaFormatoPassword()

**AccessoFederatoCTRL** `<<control>>`
- Funzioni: createAccessoFederato(), avviaAutenticazione()

**RecuperoPasswordCTRL** `<<control>>`
- Funzioni: createRecuperoPassword(), generaTokenReset()

**LogoutCTRL** `<<control>>`
- Funzioni: createLogout(), invalidaTokenLocale()

**StudenteAFAM** `<<entity>>`
- Attributi: matricola, nome, cognome, emailIstituzionale, avatarPath, visibilitaProfilo
- Funzioni: createStudente(), getMatricola(), getDatiStudente(), getEmail(), setEmail(), setAvatar()

**DBMSBND** `<<boundary>>`
- Funzioni: queryCredenziali(), queryStudente(), querySegretoSecondoFattore(), checkRegistrato(), insertStudente(), insertSessione(), insertTokenReset(), updateIdentitaFederata(), updatePassword()

- **Dipendenze:** LoginBND ⇢ LoginCTRL; Verifica2FABND ⇢ Verifica2FACTRL; RegistrazioneBND ⇢ RegistrazioneCTRL; RecuperoPasswordBND ⇢ RecuperoPasswordCTRL; LoginCTRL ⇢ StudenteAFAM; LoginCTRL ⇢ DBMSBND; Verifica2FACTRL ⇢ StudenteAFAM; Verifica2FACTRL ⇢ DBMSBND; RegistrazioneCTRL ⇢ StudenteAFAM; RegistrazioneCTRL ⇢ DBMSBND; AccessoFederatoCTRL ⇢ StudenteAFAM; AccessoFederatoCTRL ⇢ DBMSBND; RecuperoPasswordCTRL ⇢ DBMSBND


### 3.B — Gestione Profilo

**DashboardBND** `<<boundary>>`
- Funzioni: mostraDashboard(), clickDashboard(), clickImpostazioni()

**ImpostazioniProfiloBND** `<<boundary>>`
- Funzioni: mostraImpostazioni(), modificaCampi(dati, avatar), selezionaIstituto(idIstituto), clickSalva(), clickCollegaIstituto(), clickConferma()

**MessaggioBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**ErroreBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**ProfiloCTRL** `<<control>>`
- Funzioni: createProfilo()

**ModificaProfiloCTRL** `<<control>>`
- Funzioni: createModificaProfilo(), verificaFormato()

**FederazioneCTRL** `<<control>>`
- Funzioni: createFederazione(), generaIdFederato()

**StudenteAFAM** `<<entity>>`
- Attributi: matricola, nome, cognome, emailIstituzionale, bio, avatarPath, visibilitaProfilo
- Funzioni: getMatricola(), getDatiStudente(), setNome(), setBio(), setAvatar(), setVisibilita()

**IdentitaFederata** `<<entity>>`
- Attributi: idFederato, matricola, idIstitutoEsterno, dataAttivazione
- Funzioni: createIdentitaFederata(), getIdFederato()

**DBMSBND** `<<boundary>>`
- Funzioni: queryProfilo(), queryContenuti(), updateProfilo(), insertIdentitaFederata()

- **Dipendenze:** DashboardBND ⇢ ProfiloCTRL; ImpostazioniProfiloBND ⇢ ModificaProfiloCTRL; ImpostazioniProfiloBND ⇢ FederazioneCTRL; ProfiloCTRL ⇢ StudenteAFAM; ProfiloCTRL ⇢ DBMSBND; ModificaProfiloCTRL ⇢ StudenteAFAM; ModificaProfiloCTRL ⇢ DBMSBND; FederazioneCTRL ⇢ StudenteAFAM; FederazioneCTRL ⇢ IdentitaFederata; FederazioneCTRL ⇢ DBMSBND


### 3.C — Gestione Contenuti

**ArchivioBND** `<<boundary>>`
- Funzioni: mostraArchivio(), clickCaricaContenuto(), selezionaFile(file), inserisci(titolo, descrizione, visibilita), clickCarica(), clickModifica(idContenuto), modificaCampi(titolo, descrizione, visibilita), clickSalva(), clickElimina(idContenuto), apriPannelloVideo(idContenuto), selezionaImmagine(file), catturaFotogramma()

**MessaggioBND** `<<boundary>>`
- Funzioni: mostra(), clickOK(), clickConferma()

**ErroreBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**CaricaContenutoCTRL** `<<control>>`
- Funzioni: createCaricaContenuto(), verificaFile()

**ModificaContenutoCTRL** `<<control>>`
- Funzioni: createModificaContenuto(), verificaFormato()

**EliminaContenutoCTRL** `<<control>>`
- Funzioni: createEliminaContenuto()

**ThumbnailCTRL** `<<control>>`
- Funzioni: createThumbnail(), verificaImmagine()

**ContenutoMultimediale** `<<entity>>`
- Attributi: idContenuto, titolo, descrizione, tipo, percorsoFile, thumbnailPath, visibilita, matricola
- Funzioni: createContenuto(), getIdContenuto(), setTitolo(), setDescrizione(), setVisibilita(), setThumbnail()

**StudenteAFAM** `<<entity>>`
- Attributi: matricola
- Funzioni: getMatricola()

**DBMSBND** `<<boundary>>`
- Funzioni: queryContenuto(), queryContenuti(), insertContenuto(), updateContenuto(), updateThumbnail()

- **Dipendenze:** ArchivioBND ⇢ CaricaContenutoCTRL; ArchivioBND ⇢ ModificaContenutoCTRL; ArchivioBND ⇢ EliminaContenutoCTRL; ArchivioBND ⇢ ThumbnailCTRL; CaricaContenutoCTRL ⇢ ContenutoMultimediale; CaricaContenutoCTRL ⇢ StudenteAFAM; CaricaContenutoCTRL ⇢ DBMSBND; ModificaContenutoCTRL ⇢ ContenutoMultimediale; ModificaContenutoCTRL ⇢ StudenteAFAM; ModificaContenutoCTRL ⇢ DBMSBND; EliminaContenutoCTRL ⇢ ContenutoMultimediale; EliminaContenutoCTRL ⇢ StudenteAFAM; EliminaContenutoCTRL ⇢ DBMSBND; ThumbnailCTRL ⇢ ContenutoMultimediale; ThumbnailCTRL ⇢ StudenteAFAM; ThumbnailCTRL ⇢ DBMSBND


### 3.D — Gestione Portfolio

**GestionePortfolioBND** `<<boundary>>`
- Funzioni: mostraGestionePortfolio(), clickNuovoPortfolio(), clickModifica(idPortfolio), clickElimina(idPortfolio)

**CreaPortfolioBND** `<<boundary>>`
- Funzioni: mostraForm(), inserisci(titolo), selezionaContenuti(ids, ordine), selezionaVisibilita(visibilita), clickCrea()

**ModificaPortfolioBND** `<<boundary>>`
- Funzioni: mostraForm(portfolio, contenuti), modifica(titolo, ids, ordine, visibilita), clickSalva()

**MessaggioBND** `<<boundary>>`
- Funzioni: mostra(), clickOK(), clickConferma()

**ErroreBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**CreaPortfolioCTRL** `<<control>>`
- Funzioni: createCreaPortfolio(), verificaDati()

**ModificaPortfolioCTRL** `<<control>>`
- Funzioni: createModificaPortfolio(), verificaDati()

**EliminaPortfolioCTRL** `<<control>>`
- Funzioni: createEliminaPortfolio()

**Portfolio** `<<entity>>`
- Attributi: idPortfolio, titolo, visibilita, matricola, contenuti
- Funzioni: createPortfolio(), getIdPortfolio(), setTitolo(), setContenuti(), setVisibilita()

**StudenteAFAM** `<<entity>>`
- Attributi: matricola
- Funzioni: getMatricola()

**DBMSBND** `<<boundary>>`
- Funzioni: queryPortfolio(), insertPortfolio(), updatePortfolio(), deletePortfolio()

- **Dipendenze:** GestionePortfolioBND ⇢ CreaPortfolioCTRL; GestionePortfolioBND ⇢ ModificaPortfolioCTRL; GestionePortfolioBND ⇢ EliminaPortfolioCTRL; CreaPortfolioCTRL ⇢ Portfolio; CreaPortfolioCTRL ⇢ StudenteAFAM; CreaPortfolioCTRL ⇢ DBMSBND; ModificaPortfolioCTRL ⇢ Portfolio; ModificaPortfolioCTRL ⇢ StudenteAFAM; ModificaPortfolioCTRL ⇢ DBMSBND; EliminaPortfolioCTRL ⇢ Portfolio; EliminaPortfolioCTRL ⇢ StudenteAFAM; EliminaPortfolioCTRL ⇢ DBMSBND


### 3.E — Gestione Condivisione

**GestionePortfolioBND** `<<boundary>>`
- Funzioni: mostraGestionePortfolio(), clickCondividi(idPortfolio)

**NotificheBND** `<<boundary>>`
- Funzioni: clickNotifiche(), mostraNotifiche(elenco), clickSegnaLette(), clickStoricoCondivisioni(), mostraStorico(link)

**MessaggioBND** `<<boundary>>`
- Funzioni: mostra(), clickOK(), clickCopiaLink()

**GeneraLinkCTRL** `<<control>>`
- Funzioni: createGeneraLink(), generaTokenUnivoco()

**NotificheCTRL** `<<control>>`
- Funzioni: createNotifiche()

**LinkCondivisione** `<<entity>>`
- Attributi: tokenUnivoco, idPortfolio, matricola, dataScadenza, numeroVisualizzazioni, stato
- Funzioni: createLink(), getToken(), getDataScadenza(), incrementaVisualizzazioni(), setStato()

**NotificaSistema** `<<entity>>`
- Attributi: idNotifica, matricola, messaggio, letta
- Funzioni: createNotifica(), setLetta()

**StudenteAFAM** `<<entity>>`
- Attributi: matricola
- Funzioni: getMatricola()

**DBMSBND** `<<boundary>>`
- Funzioni: queryPortfolio(), insertLink(), queryNotificheNonLette(), updateNotificheLette(), queryStoricoLink()

- **Dipendenze:** GestionePortfolioBND ⇢ GeneraLinkCTRL; NotificheBND ⇢ NotificheCTRL; GeneraLinkCTRL ⇢ LinkCondivisione; GeneraLinkCTRL ⇢ StudenteAFAM; GeneraLinkCTRL ⇢ DBMSBND; NotificheCTRL ⇢ NotificaSistema; NotificheCTRL ⇢ StudenteAFAM; NotificheCTRL ⇢ DBMSBND


### 3.F — Consultazione Pubblica

**BachecaPubblicaBND** `<<boundary>>`
- Funzioni: apriSistema(), mostraBacheca(contenuti), clickContenuto(idContenuto), apriAnteprima(idContenuto), clickRicerca(), digita(query), aggiornaRisultati(profili), clickProfilo(matricola), mostraBachecaPubblica()

**ProfiloPubblicoBND** `<<boundary>>`
- Funzioni: mostraProfiloPubblico(matricola), mostraProfilo(profilo, opere, portfolios), clickPortfolio(idPortfolio)

**VisualizzatorePortfolioBND** `<<boundary>>`
- Funzioni: mostraVisualizzatorePortfolio(contenuti)

**VisualizzatoreLinkBND** `<<boundary>>`
- Funzioni: apriLink(token), mostraContenutiCondivisi(contenuti), mostra(messaggio)

**BachecaCTRL** `<<control>>`
- Funzioni: createBacheca()

**RicercaCTRL** `<<control>>`
- Funzioni: createRicerca()

**ProfiloPubblicoCTRL** `<<control>>`
- Funzioni: createProfiloPubblico()

**AccessoLinkCTRL** `<<control>>`
- Funzioni: createAccessoLink()

**ContenutoMultimediale** `<<entity>>`
- Attributi: idContenuto, titolo, tipo, percorsoFile, thumbnailPath, visibilita
- Funzioni: getIdContenuto(), getVisibilita()

**Portfolio** `<<entity>>`
- Attributi: idPortfolio, titolo, visibilita, contenuti
- Funzioni: getIdPortfolio(), getContenuti()

**LinkCondivisione** `<<entity>>`
- Attributi: tokenUnivoco, idPortfolio, matricola, dataScadenza, numeroVisualizzazioni
- Funzioni: validaToken(), incrementaVisualizzazioni()

**StudenteAFAM** `<<entity>>`
- Attributi: matricola, nome, cognome, avatarPath, visibilitaProfilo
- Funzioni: getMatricola(), getDatiStudente()

**DBMSBND** `<<boundary>>`
- Funzioni: queryContenutiPubblici(), queryProfili(), queryProfiloPubblico(), queryOperePubbliche(), queryPortfolioPubblici(), queryContenutiPortfolio(), queryLink(), updateVisualizzazioni(), insertNotifica()

- **Dipendenze:** BachecaPubblicaBND ⇢ BachecaCTRL; BachecaPubblicaBND ⇢ RicercaCTRL; ProfiloPubblicoBND ⇢ ProfiloPubblicoCTRL; VisualizzatoreLinkBND ⇢ AccessoLinkCTRL; BachecaCTRL ⇢ ContenutoMultimediale; BachecaCTRL ⇢ DBMSBND; RicercaCTRL ⇢ StudenteAFAM; RicercaCTRL ⇢ DBMSBND; ProfiloPubblicoCTRL ⇢ StudenteAFAM; ProfiloPubblicoCTRL ⇢ ContenutoMultimediale; ProfiloPubblicoCTRL ⇢ Portfolio; ProfiloPubblicoCTRL ⇢ DBMSBND; AccessoLinkCTRL ⇢ LinkCondivisione; AccessoLinkCTRL ⇢ Portfolio; AccessoLinkCTRL ⇢ DBMSBND


### 3.G — Segnalazioni

**ProfiloPubblicoBND** `<<boundary>>`
- Funzioni: clickSegnala(idContenuto), mostraSchermataPrecedente()

**FormSegnalazioneBND** `<<boundary>>`
- Funzioni: mostraForm(), selezionaMotivazione(motivazione), inserisciDescrizione(descrizione), clickInvia()

**PannelloRevisioneBND** `<<boundary>>`
- Funzioni: apriPannelloRevisione(), mostraPendenti(segnalazioni), selezionaSegnalazione(idSegnalazione), mostraDettaglio(segnalazione, contenuto), clickAccogli(), clickArchivia(), aggiornaPendenti(), clickStorico(), mostraStorico(segnalazioni)

**MessaggioBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**ErroreBND** `<<boundary>>`
- Funzioni: mostra(), clickOK()

**SegnalazioneCTRL** `<<control>>`
- Funzioni: createSegnalazione(), verificaMotivazione()

**RevisioneCTRL** `<<control>>`
- Funzioni: createRevisione()

**Segnalazione** `<<entity>>`
- Attributi: idSegnalazione, idContenuto, matricolaAutore, motivazione, descrizioneLibera, stato, dataInvio
- Funzioni: createSegnalazione(), getIdSegnalazione(), getIdContenuto(), setStato()

**ContenutoMultimediale** `<<entity>>`
- Attributi: idContenuto, visibilita
- Funzioni: getIdContenuto(), setVisibilita()

**DBMSBND** `<<boundary>>`
- Funzioni: insertSegnalazione(), querySegnalazioniPendenti(), querySegnalazioniRisolte(), updateSegnalazione(), updateContenuto()

- **Dipendenze:** ProfiloPubblicoBND ⇢ SegnalazioneCTRL; FormSegnalazioneBND ⇢ SegnalazioneCTRL; PannelloRevisioneBND ⇢ RevisioneCTRL; SegnalazioneCTRL ⇢ Segnalazione; SegnalazioneCTRL ⇢ DBMSBND; RevisioneCTRL ⇢ Segnalazione; RevisioneCTRL ⇢ ContenutoMultimediale; RevisioneCTRL ⇢ DBMSBND


### 3.H — Diagramma complessivo Entities

**StudenteAFAM** `<<entity>>`
- Attributi: matricola, nome, cognome, emailIstituzionale, bio, avatarPath, visibilitaProfilo
- Funzioni: createStudente(), getMatricola(), getDatiStudente(), getEmail(), setEmail(), setNome(), setBio(), setAvatar(), setVisibilita()

**CredenzialiSicurezza** `<<entity>>`
- Attributi: matricola, credenzialeProtetta, segretoSecondoFattore, ruolo
- Funzioni: getCredenzialeProtetta(), getSegretoSecondoFattore(), getRuolo(), setCredenzialeProtetta()

**ContenutoMultimediale** `<<entity>>`
- Attributi: idContenuto, titolo, descrizione, tipo, percorsoFile, thumbnailPath, visibilita, matricola
- Funzioni: createContenuto(), getIdContenuto(), setTitolo(), setDescrizione(), setVisibilita(), setThumbnail()

**Portfolio** `<<entity>>`
- Attributi: idPortfolio, titolo, visibilita, matricola, contenuti
- Funzioni: createPortfolio(), getIdPortfolio(), setTitolo(), setContenuti(), setVisibilita()

**LinkCondivisione** `<<entity>>`
- Attributi: tokenUnivoco, idPortfolio, matricola, dataScadenza, numeroVisualizzazioni, stato
- Funzioni: createLink(), getToken(), getDataScadenza(), incrementaVisualizzazioni(), setStato()

**Segnalazione** `<<entity>>`
- Attributi: idSegnalazione, idContenuto, matricolaAutore, motivazione, descrizioneLibera, stato, dataInvio
- Funzioni: createSegnalazione(), getIdSegnalazione(), getIdContenuto(), setStato()

**NotificaSistema** `<<entity>>`
- Attributi: idNotifica, matricola, messaggio, letta, dataCreazione
- Funzioni: createNotifica(), setLetta()

**IdentitaFederata** `<<entity>>`
- Attributi: idFederato, matricola, idIstitutoEsterno, dataAttivazione
- Funzioni: createIdentitaFederata(), getIdFederato()

**TokenResetPassword** `<<entity>>`
- Attributi: token, matricola, dataScadenza
- Funzioni: createToken(), getToken(), isScaduto()

- **Relazioni:** StudenteAFAM 1—1 CredenzialiSicurezza; StudenteAFAM 1—0..* ContenutoMultimediale; StudenteAFAM 1—0..* Portfolio; StudenteAFAM 1—0..* NotificaSistema; StudenteAFAM 1—0..1 IdentitaFederata; StudenteAFAM 1—0..* TokenResetPassword; Portfolio 1—0..* ContenutoMultimediale; Portfolio 1—0..* LinkCondivisione; ContenutoMultimediale 1—0..* Segnalazione

---

## 4. EXPORT DELLE IMMAGINI (coerenza con il LaTeX)

Il file `RAD.tex` si aspetta le immagini in queste cartelle (rispetta ESATTAMENTE nomi e
sottocartelle, altrimenti le figure non compaiono):

| Tipo | Percorso atteso dal LaTeX | Esempio |
|---|---|---|
| Diagrammi UC | `immagini/UC/<Chiave>.png` | `immagini/UC/Autenticazione.png` |
| Sequence | `immagini/SD/<NomeSottosistema>/<NomeUC>.png` | `immagini/SD/Autenticazione/Login.png` |
| Class diagram | `immagini/CD/<Chiave>.png` | `immagini/CD/Autenticazione.png` |
| Entities | `immagini/CD/Entities.png` | `immagini/CD/Entities.png` |
| Mockup | `immagini/Mockup/<Nome>.png` | `immagini/Mockup/Dashboard.png` |

Chiavi UC/CD dei sottosistemi: `Autenticazione`, `GestioneProfilo`, `GestioneContenuti`,
`GestionePortfolio`, `GestioneCondivisione`, `ConsultazionePubblica`, `Segnalazioni`.

Nomi cartelle sequence (con spazi): `Autenticazione`, `Gestione Profilo`, `Gestione Contenuti`,
`Gestione Portfolio`, `Gestione Condivisione`, `Consultazione Pubblica`, `Segnalazioni`.

**Export in Astah:** menu *File → Export Image* (o tasto destro sul diagramma → *Export Image*),
formato PNG, e salva con il nome/percorso della tabella.

## 5. ORDINE DI LAVORO CONSIGLIATO
1. Crea prima il **diagramma Entities** (3.H): definisci le entity una sola volta, le riuserai.
2. Crea i **7 class diagram** dei sottosistemi (riusa le entity già definite).
3. Crea i **7 UC diagram** (veloci, servono da indice).
4. Crea i **24 sequence**, un sottosistema alla volta, seguendo le euristiche.
5. Esporta tutto in PNG nelle cartelle indicate e compila il LaTeX.
