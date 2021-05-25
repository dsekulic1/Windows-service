**1.2 MWA Windows service** 

**1.2.1 Opis**



Windows servis predstavlja obični program koji se od standardnih aplikacija razlikuje po dva osnova kriterijuma:

- Ne zahtijeva korisničku intervenciju da bi se pokrenuo, dizajniran je da se "tiho" izvršava u pozadini.
- Podešavanje da li će servis biti pokrenut sa operativnim sistemom ili ne te eventualno zaustavljanje servisa od strane korisnika vrši se preko alatke Services u Windowsu.



Najbrži način da pristupite servisima jeste preko Start menija. 

- Kliknite na start dugme i kucajte Services (alternativni način pristupa je Win+R te u dato polje ukucati services.msc) .
- U samom vrhu liste će vam se pojaviti prečica ka već više puta spominjanoj alatki. 
- Kliknite na nju i tada će vam se otvoriti prozor kao na donjoj slici (Slika 1.6).



![](https://github.com/dsekulic1/Windows-service/blob/master/Readme%20foto/Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.001.png)

*Slika 1.6: Lista svih servisa koji su instalirani na računaru*



**1.2.2 Funkcionalnosti**



MWA je Windows aplikacija koja se na IWM-u izvršava u dva načina rada: 



- kao windows service, pod ograničenim userom koji nema Admin prava 
- kao desktop aplikacija, pod ograničenim userom koji nema Admin prava 



Realizovane funkcionalnosti u okviru windows servisa pružaju mogućnost da:



- prikuplja informacije sa IWM-a i šalje na Monitor Server (MS) 
- prima naredbe od korisnika, putem MS-a, i izvršava ih na IWM-u 
- povremeno šalje „keepalive“ signal na MS da se zna da je IWM mrežno dostupan - (dati signal se može pratiti preko wireshark-a)
- periodično šalje signale o statusu računara (trenutno vrijeme, korištenje resursa računara kao što su CPUUsage, RAMUsage, HDDUsage i GPUUsage te određena poruka.)
- periodično šalje kreirani config file na server sa svim relevantnim podacima za računar na kojem je instalirana aplikacija
- periodično šalje sve fileove iz odabranog foldera sa računara
- pruža  mogućnost gašenja i resetovanja računara na zahtjev
- pruža mogućnost slanja screenshota 
- automatsko slanje svih detektovanih errora prema serveru i spašavanja istih u bazu u cilju evidentiranja kod kojeg agenta je došlo do problema u radu
- prikuplja infromacije iz Windows Event Log-a 
- detektovane zahtjeve upisuje u Windows Event Log
- slanje system informations prema serveru
- real time komunikacija sa web i mobile aplikacijama





Važne značajke: 



- Komunikacija MWA-MS je sigurna. Realizovana je putem https enkripcije. 
- MWA moze da radi na bilo kom windowsu (win7+), x86 ili x64 oba trebaju biti podržana, podržati i server verzije Windows Server 2008+. Ukoliko ima neki prerequisite (win update, MS redist ili .Net, staviti ga da se automatski instalira prije MWA instalacije). 
- Monitor Windows servis se automatski pokreće prilikom startanja računara na kojem je instaliran


<img src="https://github.com/dsekulic1/Windows-service/blob/master/Readme%20foto/Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.002.png" alt="drawing" width="200"/>
![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.002.png)

*Slika 1.7:  Prikaz MWA Agent servisa u Services*

![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.003.png)

`                 `*Slika 1.8:  MWA Windows Service Properties*.

**1.2.3 Pregled config faile-a na Monitor Windows Servisu**

Konfiguracijski file u sklopu aplikacije omogućava pregled svih relevantnih podataka za dati računar. Realizovan je u JSON formatu. Prilikom instaliranja servisa kreira se predefinisani config filea. Prije prvog pokretanja servisa korisnik ne smije mijenjati konfiguracijski file. Nakon što se pokrene servis, šalje se zahtjev na glavni server, kojim se dobije povratna informacija u vidu novog config filea koji se prepisuje preko postojećeg. U nastavku rada aplikacije koristi se config filea sa servera koji sadrži sve jedinstvene podatke kreirane isključivo za dati računar.



Config file sadrži sljedeće podatke :



- naziv računara - name 
- lokaciju računara -location
- device id - deviceUid
- keep alive signal -keepAlive 
- web socket - webSocketUri
- pingUri - link putem kojeg servis vrši “javljanje” na server da je aktivan
- errorUri - link putem kojeg servis šalje prema serveru errore kako bi se evidentiralo u bazi kod kojeg agenta je došlo do errora prilikom njegovog rada
- main uri - link putem kojeg servis dobiva početni config file na osnovu installation code-a
- file uri - link putem kojeg servis svakih Time(1-5) minuta šalje file ili cijeli sadržaj foldera zadanih putanjama file(1-5) na server koji ih ih spašava u bazu  
- instalacijski kod - installationCode
- lokacije fileova koji se šalju na server - fileLocations

` `![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.004.png)

























*Slika 1.9: Prikaz config file-a* 

**1.2.4 Komunikacija sa serverom** 



Komunikacija sa serverom je realizovana u vidu nekoliko funkcionalnosti :



- Primanje config file-a sa servera 
- Periodično slanje odabranih file-ova ka serveru 
- Periodično šalje signale o statusu računara
- Slanje error-a nastalih prilikom rada servisa



Svi navedeni fileovi koji se šalju ka serveru se spremaju u bazu podataka na nivou cijelog sistema te služe za pregled značajnih podataka za dati računar te proračune vezane za statistiku korištenja aplikacije na računaru.



**1.2.4.1 Primanje config file-a sa servera**



Prilikom prvog pokretanja aplikacije unosi se dodijeljeni installation kod (slika 1.10). Servis se zatim spaja na server koji mu na osnovu validnog installation koda šalje sve potrebne podatke koji se spremaju u dati config file (slika 1.11), na osnovu kojih će servis komunicirati u daljem radu. 

![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.005.png)

*Slika 1.10:  Izgled config file-a prije pokretanja aplikacije*

![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.006.png)

*Slika 1.11:  Izgled config file-a poslije pokretanja aplikacije*

**1.2.4.2 Periodično slanje file-ova prema serveru svakih Time1-Time5 minuta** 

Jedna u nizu mogućnosti već pomenutog servisa je slanje filea ili cijelog foldera ka serveru svakih Time1-5 minuta (vremena navedena u config filea). Broj minuta zadaje korisnik prilikom prvog podešavanja config file. Ako je zadana nula, onda se slanje ne vrši nikako. U primjeru na slici šalje se sadržaj cijelog foldera navedenog putanjom file1 u config file-u. Slanje se vrši svako Time1 minuta (u datom primjeru 1 minuta), što se na osnovu TimeStamp-a na slici i može vidjeti.



![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.007.png)

*Slika 1.12:  Prikaz file-ova koji se šalju prema serveru* 

**1.2.4.3 Periodično slanje signala na server sa informacijom o statusu kompjutera** 



Monitor Windows Agent čita podatake vezane za CPU, RAM, HDD i GPU sa računara, te iste šalje prema serveru. Navedeni podaci se šalju svako keepAlive sekundi navedenih u config file-u (slika 1.13).



![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.008.png)

*Slika 1.13: Prikaz podataka o iskoristivosti resursa računara*

**1.2.4.4 Slanje error-a nastalih prilikom rada servisa**



Moguće je da tokom rada servisa dođe do određenih poteškoća pri primanju ili slanju zahtjeva na server. Korisnik servisa ne mora biti upućen u prirodu grešaka niti znati da se greška dogodila jer iako dođe do neke greške, servis nastavlja sa radom. Svaki error do kojeg dođe se automatski šalje na servis i evidentira se u bazu.



**1.2.5 Real time komunikacija sa web i mobile aplikacijom** 



Komunikacija sa web i mobile aplikacijom je realizovana u vidu nekoliko funkcionalnosti :



- Slanje screenshot-a na zahtjev sa web ili mobilne aplikacije
- Slanje system info sa računara na kojem je instalirana aplikacija
- Remote terminal
- Mogućnost gašenja ili resetovanja računara



**1.2.5.1 Screenshot** 



Dodatna mogućnost MWA servisa je sposobnost slanja screenshot-a. Ova funkcionalnost se realizuje kada dođe zahtjev sa web ili mobilne aplikacije. Sve je to usklađeno u realnom vremenu kako bi zadovoljilo da se stvarni snimci ekrana prikazuju u tačno određenim intervalima kada to krajnji korisnik želi. Naravno to povećava mogućnost pronalska nekih errora ili bugova koji bi se eventualno mogli desiti pri samom radu MWA servisa. Zbog ograničenja u mogućnostima Windows servisa, snimak ekrana ne može napraviti sami servis. Iz tog razloga, koristi se pomoćna aplikacija ScreenshotApp koja pravi snimak ekrana svakih 5 sekundi. Aplikacija se pokreće automatski pri svakom paljenju kompjutera.

![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.009.png)

*Slika 1.14: Screenshot ekrana koji se šalje real-time*

**1.2.5.2 Slanje systems information prema serveru**



Još jedna mogućnost MWA servisa je sposobnost slanja system information ka web ili mobile aplikaciji. Ova funkcionalnost se također realizuje na osnovu zahtjeva sa već pomenutih aplikacija. Servis pokupi sve sistemske informacije, spremi ih u json objekat te ih proslijedi datim aplikacijama koje ih iščitaju i analiziraju.



![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.010.png)

*Slika 1.15:  Izgled sistemskih informacija koje su poslane ka serveru*

**1.2.5.3 Remote terminal (web i mobile)**



Naš servis podržava takozvani remote terminal, tj. komande sa web i mobilne aplikacije trebaju da se izvršavaju na zadanom računaru ali sa neke udaljene lokacije. Naš servis omogućava izvršavanje tj. upravljanje komandi terminala npr. komande tipa :  mkdir , cd, ls i slično. Kao najveća prednost remote terminala je ta što korisnik može raditi sa udaljene lokacije ali da se čini kao da se radi na tom računaru.



![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.011.png)

*Slika 1.16:  Terminal na web aplikaciji konektovan na računar preko servisa* 



**1.2.5.4 Mogućnost gašenja i resetovanja računara na zahtjev**



U sklopu servisu je provedena mogućnost da se računar na kojem je servis pokrenut može ugasiti na osnovu zahtjeva sa web ili mobile aplikacije. Date aplikacije komandama kroz terminal te unošenjem odgovarajućih podataka o imenu i šifri računara gase ili resetuju dati računar. Prilikom ponovnog pokretanja računara servis se pokreće automatski što je prikazano na slici ispod:



![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.012.png)



*Slika 1.17:  Prikaz automatskog startanja servisa* 



**1.2.6 Event Viewer** 



Windows Event Viewer prikazuje logove aplikacije i informacije o aktivnosti servisa. Preciznije, prikazuje informacije o radnjama koje je preduzeo krajnji korisnik. Do Event Viewera se dolazi upisom “Event Viewer” u Start pretragu. Nakon otvaranja aplikacije odabere se Monitor Windows Service iz sekcije Applications and Services Logs.

U logu se nalazi naziv klijenta koji je uputio zahtjev prema agentu, kao i podatak o tipu zahtjeva koji je klijent uputio. Iz logova se jasno može vidjeti u koje vrijeme je koji zahtjev upućen. Tip Loga je “Information”. Klikom na neki od logova prikažu se specifični detalji.



![](Aspose.Words.b3255f80-6125-453d-b5b2-84e27e189ed5.013.png)

*Slika 1.17:  Prikaz svih zahtjeva u event view-eru*



**1.2.7 Uputstvo za instalaciju i korištenje**

\1. Install Notepad++

\2. Install ScreenshotApp

\3. Open CMD (Win+R and type cmd, press enter)

\4. Type cd "C:\Program Files (x86)\Grupa2" (copy all)

\5. Type mkdir "Monitor Service" (copy all)

\6. Open File Explorer (Windows search and type File Explorer)

\7. Open folder: "C:\Program Files (x86)\Grupa2\Monitor Service"

\8. Copy config file in folder

\9. Open config file with notepad++ and type installationcode 

\10. Input File1-File2 directory(file) path and Time1-5 int minute

\11. Install MonitorWindowsAgentService

\12. Open File Explorer (Windows search and type File Explorer)

\13. Open folder: "C:\Program Files (x86)\Grupa2\ScreenshotApp" and start Screenshot app



Link za download: <https://drive.google.com/drive/folders/1fbKkZdho65K7S7QS6_iX0FP6gUGjZx_h?fbclid=IwAR0epxKke462EhwMRyHnjVcKRf5qlF_fztYbmTP3ozPBZ8lsFmd_F-vfoXE>



**1.2.9 Upotrebljivost**

Bilo kojem korisniku koji je zainteresovan za rad našeg MWA Windows servisa i njegovih funkcionalnosti je sve objašnjeno u Uputama za instalaciju i korištenje.

Još jedna od odlika Usability-a u FURPS-ovom modelu je dokumentacija. Za naš projekat je odrađena dokumentacija i cijeli proces implementacije je praćen i bilježen po sprintovima.
**


**1.2.10 Pouzdanost**

Komunikacija između servisa i servera je veoma sigurna i za nju se koristi https protokol. Prilikom svakog poziva servera za određenu rutu vrši se autentifikacija pomoću jwt tokena koji je dostupan nakon autorizacije korisnika. Ovaj token važi 30 minuta. Pored toga određene operacije dodavanja i ažuriranja informacija su dostupne samo korisnicima sa određenim pravima koja su unaprijed definisana u okviru njihovih rola. Server s kojim je povezan naš servis je platforma DigitalOcean koja se ističe u neprekidnom radu, isporučujući u prosjeku> 99,99% vremena tokom posljednje godine praćenja. Handliranje errora je urađeno na način da  se svi errori  automatski šalju prema serveru kako bi se evidentiralo u bazi  kod kojeg agenta je došlo do errora u radu applikacije. U slučaju error-a šalje se poruka ka serveru, u kojem se šalje poruka error-a i kod greške.





**1.2.11 Performanse**

Za sva navedena vremena keepAlive sekundi, te Time1-5 minuta se šalju signali serveru. Naš servis nema ograničenje broja korisnika koji mogu istovremeno koristiti servis, to jeste više korisnika ga može istovremeno koristiti. 





` `**1.2.12 Podrška**

Namijenjen za windows operativni sistem.

