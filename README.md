# Apple notarization

When a user runs an app for the first time, macOS Gatekeeper check
Quando un utente esegue un'app er la prima volta, macOS Gatekeeper si connette ai server Apple per verificare che un'app, pkg, dmg, bundle provenga da uno sviluppatore attendibile.

![](https://www.davidebarranca.com/wp-content/uploads/2019/05/gatekeeper.gif)

Per consentire il processo anche offline si raccomanda di associare la conferma di avvenuta notarizzazione all'app con il comando `staple`.

La notarizzazione è necessaria per app, plugin, dmg e pkg.
Se si intende distribuire tramite **pkg** o **dmg** (che può anche conterere pkg) basta notarizzare il pkg o dmg e tutto ciò che contiene verrà notarizzato.  
In questo caso assicurarsi che tutti gli elementi all'interno abbiano una firma valida.

## Sources

- https://www.davidebarranca.com/2019/04/notarizing-installers-for-macos-catalina/
- https://www.kvraudio.com/forum/viewtopic.php?t=531663

## Prerequisiti

- XCode 10 o superiore (meglio se ultima versione scaricabile dall'App Store)
- Accesso a internet
- Apple Developer ID (99€ all'anno)
- [Password specifica](https://support.apple.com/it-it/HT204397)

## Ottenere un certificato

Dal proprio [portale Apple Developer](https://developer.apple.com/) generare, scaricare e installare un certificato. Esistono vari tipi di certificati, scegliere quello corretto tra:
- Developer ID Installer
- Developer ID Application 

![](https://www.davidebarranca.com/wp-content/uploads/2019/05/DeveloperID.png)

Verificare la corretta installazione con il comando:

> security find-identity -v

## Firma

Affinchè la notarizzazione vada a buon fine, tutti gli elementi devono avere una firma valida.

### Firmare un'app tramite XCode

- Aggiungere un account da sviluppatore nelle impostazioni.
- Nelle impostazioni del progetto selezionare il proprio account sviluppatore e il certificato.

### Firmare un'app fuori da XCode

Usare `codesign` per app, bundle, plugin e `productsign` per pkg.

> codesign -s "Developer ID Application: FATAR SRL (9L4T4JL6GX)" "/path_to_app.app" --timestamp  
productsign --sign "eveloper ID Installer: FATAR SRL (9L4T4JL6GX)" ./installer.pkg ./installer_signed.pkg

### Verifica della firma

Usare `codesign` per app, bundle, plugin e `pkgutil` per pkg.  
Assicurarsi che venga specificato il Developer ID altrimenti la notarizzazione non andrà a buon fine.

> codesign — verify — verbose /Applications/AppName.app  
pkgutil --check-signature ./installer_signed.pkg

## Inviare la richiesta di notarizzazione

### Tramite XCode

Dopo aver *Archiviato* un build, premere validate e seguire le istruzioni in base al metodo di distribuzione scelto.

### Manualmente

> xcrun altool --notarize-app --primary-bundle-id "com.yourcompany.app" -u "info@fatar.com -p "tozc-tzoo-smqj-wybr" -f "/full/path/to/the/installer_signed.pkg"

Se non ci sono errori durante l'upload verrà restituito l'ID della richiesta.  
E' possibile monitorare lo stato della singola richiesta oppure mostrare l'intera cronologia:

> xcrun altool --notarization-info 85b5e831-3fa0-4082-8ec8-d564d69869ef -u "info@fatar.com" -p "tozc-tzoo-smqj-wybr"  
xcrun altool --notarization-history 0 -u "info@fatar.com" -p "tozc-tzoo-smqj-wybr"

### Appicciare l'esito della notarizzazione al file

Questo passaggio è opzionare ma raccomandato perchè contente la verifica di Gatekeeper anche offline.

> xcrun stapler staple "/full/path/to/myapp.app"
> Processing: /full/path/to/myapp.app
The staple and validate action worked!

## Controllare notarization in file

E possibile controllare se un file é notorizzato in diversi modi.

### spctl

> $ spctl -a -vvv -t exec /Path/To/Notarised.app  
/Path/To/Notarised.app: accepted  
source=Notarized Developer ID  
origin=Developer ID Application: ***

Sostituire `exec` con `install` per i **pkg**.

(`spctl` consente anche di aggiungere, rimuovere, visualizzare gli sviluppatori autorizzati nel sistema)

### stapler

> $ stapler validate myfile.pkg  
Processing: myfile.pkg  
The validate action worked!

### Extra 1: Packages

[WhiteBox Packages](http://s.sudre.free.fr/Software/Packages/about.htmls) è un strumento grafico per la creazione di pkg.  
E possibile associare il certificato in Packages per evitare di farlo manualmente con `productsign`.

![](https://i.imgur.com/xyvfSyK.png)
![](https://i.imgur.com/aEkKzOK.png)

### Extra 2: DMG Canvas

[Araelium DMG Canvas](https://www.araelium.com/dmgcanvas) is GUI tool for making DMG 
