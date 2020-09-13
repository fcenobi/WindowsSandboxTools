# Windows Sandbox Tools

![sandbox](images/sandbox.jpg)

Questo repository è una raccolta di strumenti e script di PowerShell che utilizzo per eseguire e configurare la funzionalità Sandbox di Windows che fa parte di Windows 10 2004. Molti dei comandi in questo repository sono stati dimostrati per la prima volta sul mio blog . Ti consiglio vivamente di leggere il post del blog prima di provare qualsiasi codice. Come ho detto nel post del blog, la maggior parte del codice qui ridurrà la sicurezza dell'applicazione Windows Sandbox. Questo è un compromesso che sono disposto a fare per il bene della funzionalità che soddisfa le mie esigenze. Devi decidere quanto codice desideri utilizzare.

Tutto il codice viene offerto così com'è senza garanzie. Niente in questo repository deve essere considerato pronto per la produzione o utilizzato in ambienti critici.

Installazione della sandbox di Windows
Devi avere la versione 2004 di Windows 10. Non so se è supportata su Windows 10 Home. Altrimenti, dovresti essere in grado di eseguire questi comandi di PowerShell:

Get-WindowsOptionalFeature  - online - FeatureName Containers - DisposableClientVM
 Enable-WindowsOptionalFeature  - Online - FeatureName Containers - DisposableClientVM
I miei strumenti
Il completamento dello script di configurazione predefinito richiede circa 4 minuti. Uso il modulo BurntToast per mostrare una notifica del Centro operativo di Windows quando è completo. Questo progetto non è un modulo di PowerShell o un set di file che puoi eseguire così com'è. Puoi clonare, scaricare o copiare se necessario.

Start-WindowsSandbox
La Start-WindowsSandboxfunzione è il mio strumento principale. Ha un alias di wsb. È possibile specificare il percorso del file WSB.

Start-WindowsSandbox

Se utilizzi il NoSetupparametro, verrà avviato il Sandbox di Windows predefinito. In entrambi gli utilizzi, è possibile specificare le dimensioni di visualizzazione per la sandbox. Il WindowSizeparametro prevede un array di larghezza e altezza, come 1024,768. Il mio valore predefinito è 1920,1080. Potrebbe essere necessario trascinare leggermente la finestra per forzare la sandbox a ridisegnare lo schermo e rimuovere la barra di scorrimento orizzontale. L'impostazione del display è complicata e non so se quello che sto usando funzionerà per tutti, quindi se non ottieni i risultati che ti aspetti, pubblica un problema.

Funzioni WSB
Ho creato un semplice modulo chiamato wsbFunctions.psm1. In questo modulo sono presenti funzioni progettate per rendere più semplice visualizzare una configurazione WSB, creare una nuova configurazione ed esportare una configurazione in un file. Le funzioni utilizzano una serie di definizioni di classe.

PS C:\> Get-WsbConfiguration d:\wsb\simple.wsb
WARNING: No value detected for LogonCommand. This may be intentional on your part.


   Name: Simple

vGPU                 : Enable
MemoryInMB           : 8192
AudioInput           : Default
VideoInput           : Default
ClipboardRedirection : Default
PrinterRedirection   : Default
Networking           : Default
ProtectedClient      : Default
LogonCommand         :
MappedFolders        : C:\scripts -> C:\scripts [RO:False]
Il comando utilizza un file di formato personalizzato per visualizzare la configurazione. Ho anche trovato un modo per inserire metadati nel file WSB che (finora) non interferisce con l'applicazione Sandbox di Windows.

PS C:\> Get-WsbConfiguration d:\wsb\simple.wsb -MetadataOnly

Author     Name   Description                                       Updated
------     ----   -----------                                       -------
Jeff Hicks Simple a simple configuration with mapping to C:\Scripts 9/10/2020 4:47:10 AM
Se lo desideri, puoi creare una nuova configurazione.

$ params  =  @ {
  Networking  =  " Default "
  LogonCommand  =  " c: \ data \ demo.cmd "
  MemoryInMB  =  2048 
 PrinterRedirection  =  " Disable "
  MappedFolder  = ( New-WsbMappedFolder  - HostFolder d: \ data  - SandboxFolder c: \ data  - ReadOnly True)
  Name  =  " MyDemo "
  Description  =  " Una configurazione WSB demo "
}
$ new  =  New-WsbConfiguration   @params
Il LogonCommandvalore è relativo a WindowsSandbox. Questo codice creerà un wsbConfigurationoggetto.


   Name: MyDemo

vGPU                 : Default
MemoryInMB           : 2048
AudioInput           : Default
VideoInput           : Default
ClipboardRedirection : Default
PrinterRedirection   : Disable
Networking           : Default
ProtectedClient      : Default
LogonCommand         : c:\data\demo.cmd
MappedFolders        : d:\data -> c:\data [RO:True]
È possibile modificare questo oggetto se necessario.

$ new .vGPU  =  " Abilita "
 $ new .Metadata.Updated  =  Get-Date
L'ultimo passaggio consiste nell'esportare la configurazione in un wsbfile.

$ nuovo  |  Export-WsbConfiguration  : percorso d: \ wsb \ demo.wsb
Che creerà questo file:

< Configurazione >
  < Metadati >
    < Nome > MyDemo </ Nome >
    < Autore > Jeff </ Autore >
    < Description > Una configurazione WSB demo </ Description >
    < Aggiornato > 10/09/2020 15:40:54 </ Aggiornato >
  </ Metadati >
  < vGPU > Abilita </ vGPU >
  < MemoryInMB > 2048 </ MemoryInMB >
  < AudioInput > Predefinito </ AudioInput >
  < VideoInput > Predefinito </ VideoInput >
  < ClipboardRedirection > Default </ ClipboardRedirection >
  < PrinterRedirection > Disabilita </ PrinterRedirection >
  < Networking > Default </ Networking >
  < ProtectedClient > Predefinito </ ProtectedClient >
  < LogonCommand >
    < Comando > c: \ data \ demo.cmd </ Comando >
  </ LogonCommand >
  < MappedFolders >
    < MappedFolder >
      < HostFolder > d: \ data </ HostFolder >
      < SandboxFolder > c: \ data </ SandboxFolder >
      < ReadOnly > True </ ReadOnly >
    </ MappedFolder >
  </ MappedFolders >
</ Configurazione >
Posso avviare facilmente questa configurazione.

Start-WindowsSandbox  - Configurazione D: \ wsb \ demo.wsb
RoadMap
Ora che ho aggiunto più funzionalità, posso creare un singolo modulo di PowerShell per includere tutte le funzioni di gestione di Windows Sandbox. Potrei anche cercare un modo per organizzare i componenti di script utilizzati per le impostazioni di LogonCommand e una soluzione migliore per l'organizzazione dei wsbfile.

Ultimo aggiornamento 10 settembre 2020 .
