---
id: version-4.0.0-beta.10-best
title: Migliori Pratiche
original_id: best
---

La guida seguente è un elenco delle migliore pratiche raccolte e che raccomandiamo solitamente a tutti gli utenti. Non considerare questa guida come obbligatoria, puoi selezionare qualcuna di esse a seconda delle tue esigenze.

**Suggerisci le tue migliori pratiche alla community di Verdaccio**.

## Registro Privato

È possibile aggiungere utenti e gestire quali utenti possono accedere a quali pacchetti.

È raccomandabile definire un prefisso per i tuoi pacchetti privati, per esempio `local-*` o `@my-company/*` con scope, così che tutti i tuoi elementi privati appariranno così: `local-foo`. In questo modo si possono separare chiaramente i pacchetti pubblici da quelli privati.

```yaml
  packages:
    '@my-company/*':
      access: $all
      publish: $authenticated
     'local-*':
      access: $all
      publish: $authenticated
    '@*/*':
      access: $all
      publish: $authenticated
    '**':
      access: $all
      publish: $authenticated
   ```

Ricorda sempre, **l'ordine di accesso ai pacchetti è importante**, i pacchetti vengono combinati sempre dall'alto verso il basso.

### Utilizzo di pacchetti pubblici da npmjs.org

Se un pacchetto non esiste nell'archivio, il server proverà a recuperarlo da npmjs.org. Se npmjs.org non funziona, fornirà solo i pacchetti presenti nella cache come se non ne esistessero altri. **Verdaccio scaricherà solo ciò che è necessario (= richiesto dai client)** e questa informazione verrà memorizzata nella cache, così che se il client chiederà la stessa cosa una seconda volta, potrà essere soddisfatto senza dover chiedere a npmjs.org.

**Esempio:**

Se si fa una richiesta `express@4.0.1` che va a buon fine da questo server una volta, sarà possibile farla un'altra volta (con tutte le sue dipendenze) in ogni momento, anche con npmjs.org non funzionante. Però diciamo che `express@4.0.0` non verrà scaricato fino a che non sia effettivamente necessario per qualcuno. E se npmjs.org è offline, questo server direbbe che solo `express@4.0.1` (= solo quello che è nella cache) viene pubblicato, ma nient'altro.

### Override su pacchetti pubblici 

Se si desidera utilizzare una versione modificata di un pacchetto pubblico `foo`, si può pubblicarlo solamente sul server locale, così scrivendo `npm install foo`, **valuterà di installare questa versione**.

Ci sono due opzioni qui:

1. Si desidera creare un **fork** separato e interrompere la sincronizzazione con la versione pubblica.

   Se si vuole fare ciò, si dovrebbe modificare il file di configurazione affinché verdaccio non faccia più richieste a npmjs riguardo a questi pacchetti. Aggiungere un'entrata separata per questo pacchetto a` config.yaml`, rimuovere `npmjs` dalla lista `proxy` e riavviare il server.

   ```yaml
    packages:
      '@my-company/*':
        access: $all
        publish: $authenticated
        # commentare o lasciare vuoto
        # proxy:
   ```

   Quando si pubblica localmente il pacchetto, **si dovrebbe iniziare con una stringa di versione superiore a quella esistente**, così non entrerà in conflitto con il pacchetto esistente nella cache.

2. Si vuole temporaneamente utilizzare la propria versione, ma tornare alla pubblica appena questa sia aggiorna,.

   Per evitare qualsiasi conflitto delle versioni, **si dovrebbe usare un suffisso personalizzato rilasciato prima della successiva versione della patch**. Per esempio, se un pacchetto pubblico ha la versione 0.1.2, si può caricare `0.1.3-my-temp-fix`.

   ```bash
    npm version 0.1.3-my-temp-fix
    npm --publish --tag fix --registry http://localhost:4873
   ```

   In questo modo il pacchetto verrà utilizzato fino a che il suo maintainer originale aggiorna 
 il suo pacchetto pubblico a `0.1.3`.




## Sicurezza

La sicurezza comincia nel proprio ambiente, perciò invitiamo a leggere assolutamente **[10 npm Security Best Practices](https://snyk.io/blog/ten-npm-security-best-practices/)** e a seguire le raccomandazioni.

### Accesso ai Pacchetti

Di default tutti i pacchetti che pubblichi su Verdaccio sono accessibili a chiunque, ti raccomandiamo vivamente di proteggere il tuo registro da utenti esterni non autorizzati che aggiornano la proprietà `access` a `$authenticated`.

```packages:
    '@my-company/*':
      access: $authenticated
      publish: $authenticated
    '@*/*':
      access: $authenticated
      publish: $authenticated
    '**':
      access: $authenticated
      publish: $authenticated
   ```

In questo modo, **nessuno sarà in grado di utilizzare il tuo registro a meno che non sia autorizzato e i pacchetti privati non verranno visualizzati nell'Interfaccia Utente**.

## Server

### Connessioni Assicurate

L'utilizzo di **HTTPS** è una raccomandazione comune, per questa ragione raccomandiamo di leggere la sezione [SSL](ssl.md) per rendere Verdaccio sicuro o di utilizzare un HTTPS [reverse proxy](reverse-proxy.md) oltre a Verdaccio.

### Token in Scadenza

In `verdaccio@3.x` i token non hanno data di scadenza. Per questo motivo, nel prossimo `verdaccio@4.x`, abbiamo introdotto la funzionalità JWT [PR#896] (https://github.com/verdaccio/verdaccio/pull/896)

```yaml
security:
  api:
    jwt:
      sign:
        expiresIn: 15d
        notBefore: 0
  web:
    sign:
      expiresIn: 7d
```

**L'utilizzo di questa configurazione sovrascriverà il sistema corrente e sarai in grado di controllare per quanto tempo il token sarà valido**.

Utilizzare JWT migliora inoltre la prestazione con i plugin di autenticazione, il vecchio sistema realizzerà una decompressione e convaliderà le credenziali in ciascuna richiesta, mentre JWT dipenderà dalla firma del token evitando l'overhead per il plugin.

Come nota a margine, in **npmjs il token non scade mai**.