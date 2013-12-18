PDE - Patient Documentation Exchange
===

# Team #
- Petr Onderka (gsvick (at) gmail (dot) com)
- Filip Řepka (repik (dot) x (at) seznam.cz)
- Ondra Heřmánek (ondra (dot) hermanek (at) gmail.com)

# Situace #
Doktor při každé návštěvě (nového) pacienta zjišťuje jeho anamnézu - zdravotní stav, předchozí nemoci, vyšetření, operace, rodinnou anamnézu a podobně. Pacient navíc nemusí všechny informace vědět, nebo může něco úmyslně zatajit. Navíc tento proces doktorovi zabírá nemalé množství času.

Současný stav výměny dokumentace mezi doktory je následující: když doktor posílá pacienta za jiným doktorem na nějaké vyšetření, vytiskne mu jeho kartu, kterou musí pacient donést tomu jinému doktorovi. Tak karta ale obsahuje jen informace od prvního doktora. Po vyšetření by pacient kartu měl správně odevzdat svému obvodnímu doktorovi, ale často se tak neděje a pacienti si kartu nechávají, ztrácí nebo vyhazují.

O řešení tohoto problému se již v minulosti snažil projekt **IZIP**, který neuspěl především z důvodu, že systém byl centralizovaný v rukou pojišťoven, které kontrolovali, jestli doktoři dělají, co vykazují, že dělají (což někdy kvůli přísným limitům donktoři obcházeli v dobré víře lépe pomoci více lidem).

# Motivace #
Cílem tohoto projektu je navrhnout systém, který umožní doktorům získat celkovou dokumentaci pacienta a obohacovat ji. Systém bude o každém pacientovi obsahovat seznam všech návštěv u doktorů, popis vyšetření a nějakou výstupní zprávu. Tato dokumentace bude přístupná na vyžádání - doktor při vyšetření, běhěm výjezdu sanitního vozu.

# Ideální svět #
Všichni dokotři a zdravotnická zařízení mají telefon a přístup k počítači se zabezpečeným internetovým připojením, díky kterému budou moci pacientovu dokumentaci sdílet. Všichni doktoři se sdílením dokumentace souhlasí, všichni pacienti také. 

# První návrh řešení #
## Centralizované / distribuované řešení ##
Systém může uchovávat dokumentaci na jednom místě, to má ale mnoho nevýhod - dostupnost, komunikační náročnost, ochrana osobních údajů, veliké množství dat. Tato centra se dají duplikovat - ochrana proti výpadku, ale tím dostáváme distribuovaný systém, kterému roste náročnost na synchronizaci mezi jednotlivými uzly.

Proto navrhujeme kompromisní řešení - centralizovat dokumentaci v daných uzlech a distribuovaně šířit pouze **Index** - seznam pacientů a jejich navštěv u doktorů a kódu vyšetření. Tento index bude určovat, na kterém centrálním uzlu leží jaká konkrétní **Dokumentace**.

## Index ##
Pro **Index** očekáváme řádově desítky milionů záznamů, proto bude kladen důraz na minimální velikost jednoho záznamu. Ideálně tedy: 

<pre>ID pacienta | ID vyšetřujícího doktora | Datum vyšetření | ID zákroku | ID zařízení, kde je **Dokumentace** uložena</pre>

Jako ID pacienta by šlo použít jeho rodné číslo, to ale není unikátní, proto jako ID bude použijeme rodné číslo kombinované s celým jménem pacienta.

ID doktora bude bráno jako kombinace názvu jeho informačního systému, který používá, a přihlašovacího jména, kterým se přihlašuje. Tyto informační systémy si unikátnost přihlašovacích údajů řeší samy, tedy unikátnost takto vytvořených ID je zaručena. Případné kolize se dají řešit při registraci doktora do PDE systému.

## Dokumentace pacientů ##
**Dokumentace** se bude uchovávat centralizovaně v daných uzlech - zdravotnická zařízení nebo zdravotní pojišťovny nebo obvodní lékaři. Doktoři budou dokumentaci na tyto uzly nahrávat - doktoři v nemocnicích přímo v rámci nemocničního systému, soukromí doktoři přes nějaké webové rozhraní nebo pomocí svých informačních systémů, které budou s tímto celým systémem komunikovat. Jednotlivé uzly se budou starat o zabezpečení osobních údajů, o dostupnost dokumentace a o aktualizaci **Index**u.

Logicky nejsprávnější by bylo uchovávat dokumentaci paciantů u jejich obvodních lékařů, kteří by o svých pacientech měli vědět vše (tedy mít veškerou jejich dokumentaci). To by znamenalo vybavit každého doktora serverem, který bude idálně 24/7/356 dostupný a zabezpečený proti zneužití. 

Proto budou zřízeny centralizované uzly v nemocnicích a každému zdravotníkovi bude určen (přiřazen pri registraci) jeden uzel, který bude uchovávat dokumentaci jeho paicentů.

## Zdravotnická dokumentace ##
Dokumentace se bude uchovávat centralizovaně v daných uzlech - zdravotnická zařízení nebo zdravotní pojišťovny nebo obvodní lékaři. Doktoři budou dokumentaci na tyto uzly nahrávat - doktoři v nemocnicích přímo v rámci nemocničního systému, soukromí doktoři přes nějaké webové rozhraní nebo pomocí svých informačních systémů, které budou s tímto celým systémem komunikovat. Jednotlivé uzly se budou starat o zabezpečení osobních údajů, o dostupnost dokumentace a o aktualizaci **Index**u.

## Slovník ##
Zde je uveden význam používaných pojmů

- **Dokumentace** - Záznam o vyšetření pacienta
- **Index** - Záznam o tom, na kterém uzlu ze nachází jaká **Dokumentace**
- **Centrální uzel** - Uzel, kde se ukládá **Dokumentace**, **Index** a odkud se distribuují okolním uzlům
	- *lokální* - uzel, kam doktor nahrává  **Dokumentaci** (myšleno v kontextu doktora)
	- *vzdálený* - ostatní uzly, kde se nachází dokumentace, doktor na něj dokumentaci nenahrává, pouze ji může získat (myšleno v kontextu doktora)

## Přístupnost dokumentace ##
### Jak bude dokumentace přístupná? ###
- Všichni vidí vše - není ideální, doktoři by neměli nahlížet do dokumentace cizích pacientů, pokud to není potřeba
- Nikdo nevidí nic - nebylo by co sdílet
- Přístupné jen něco - relevantní data mohou být zrovna nepřístupná
- Výchozí dostupnost?

### Kdo bude určovat viditelnost informací? ###
- Pacient určuje, co bude dostupné a co ne - jak to změní?
- Doktor určuje, co bude dostupné a co ne - měl by mít pacientův souhlas?

### Zvolené řešení ###
Veškeré **Dokumentace** budou přístupné všem doktorům. Pro ochranu osobních údajů proti zneužití bude každá akce - vytvoření/úprava/mazání/vyžádání **Dokumentace** - zalogována. Bude tedy možné dohledat, který doktor přistoupil k jaké **Dokumentaci**.

Toto řešení bylo zvoleno, protože omezování přístupu doktorů k dokumentaci by vedlo k problémům (např. doktor nemůže vyšetřit pacienta, protože mu omylem nebylo dáno právo přístupu k jeho dokumentaci). Na druhou stranu, do systému mají přístup jen autentizovaní doktoři, takže riziko zneužití zřejmě není tak velké (ve srovnání se systémy, které jsou veřejně přístupné).

## Registrace doktorů ##
Každý doktor se zaregistruje do systému PDE, dostane pevné ID a náhodný token, pod kterým bude identifikován v rámci systému, rovněž mu bude přiřazen centrální uzel, se kterým bude jeho zdravotnický IS komunikovat. Při registraci uvede rovněž telefonní číslo a další údaje. 

Jako jeho ID bude považována kombinace jeho zdravotnického IS a jeho loginu do jeho systému. Token mu bude vygenerován náhodně a bude sloužit k validaci požadavků vyslaných zdravotnikým IS doktora. 

# Případy použití #

## Vyšetření pacienta ##
Pacient přijde k doktorovi, ten si pomocí **Index**u zjistí, kde všude byl na vyšetření a jaké to bylo (jen podle kódu vyšetření). Pokud nějaké vyšetření bude doktora zajímat, vyžádá si celý záznam o vyšetření od zdravotního střediska, kde bylo vyšetření provedeno a které tedy den záznam vyšetření shraňuje. 

Odpovědí na vyžádání tohoto záznamu bude buď samotný záznam, který si doktoru bude moci přečíst, nebo informace, že záznam není k dispozici a telefonní číslo na doktora, který záznam pořídil. To je nutné, aby v případě potřeby se dalo aspoň telefonicky informace o vyšetření předat.

## Nahrání dokumentace do centralizovaného úložiště ##
Vyšetřující doktor provede vyšetření pacientovi. Vyplní informace o vyšetření do svého systému. Systém může **Dokumentaci** rovnou nahrát do určeného úložistě (online). Nebo být schopen vyexportovat data v takovém formátu, aby se tato data dala nahrát do určených uzlů (offline)

Zdravotnické informační systémy: Medicus, ...

## Zprávy ##
- Zadost o Index
	- <pre>datum | doktorId | pacientId</pre>
- Odpoved Indexu
   - <pre>[ documentId | uzelId | pacientId | doktorId | doktorTelefon | doktorInfo|  typVysetreni | datumZmeny |  pacientInfo ]</pre>
- Zadost o dokumentaci
	- <pre>datum | doktorId | pacientId | uzelId | document</pre>
- Odpoved s  dokumentaci
		- vlasntí **Dokumentace**
- Vlozeni/Update/Delete **Dokumentace**:
    - <pre>datumVytvoreni | datumAkce | doktorId | pacientId | typVysetreni | typAkce | vlastni dokumentace | pacientInfo </pre>
- Synchronizace indexu:
   <pre>datumVytvoreni | datumAkce | doktorId | pacientId | typVysetreni | typAkce | dokumentaceId | pacientInfo</pre>

# DÚ 1#
- Modularizace - schéma základního rozdělení na moduly a jejich vazby
![Component Model](doc/ComponentModel.png)

- [Kvalitativní atributy](http://msdn.microsoft.com/en-us/library/ee658094.aspx)
	- ***Design Qualities***
		- **Conceptual Integrity**
			- Takto robustní systém je potřeba dobře rozmyslet a navrhnout, ustanovit jednotný coding style, navrhnout základní postupy při vývoji, strategii a organizaci týmu, ... 
		- Maintainability
			- Nebude snadné jednoduše provést aktualizaci distribuovaného systému, takže by se na možná rozšíření mělo myslet už při návrhu (například počítat s podporou obrázků v dokumentaci pacienta).  
		- Reusability
			- Systém bude nabízet API pro komunikaci s ostatnímy systémy
	- ***RunTime Qualities***
		- **Availability**
			- Měl by být kladen důraz na dostupnost API, která sbírají data od doktorů a ukládají je do nemocničních IS (99%).
			- Dostupnost dokumentace z nemocničního IS už nemusí být tak ostře hlídána (95%). **TODO: upřesnit,jak jsme došli k těm číslům**
		- **Performance**
			- Distribuovaný systém vyžaduje režii na synchronizaci uzlů, proto je potřeba minimalizovat velikost a počet zpráv určených na synchronizaci **Indexu**. Na rychlost přenesení nových informací pomocí synchronizace není kladen zásadní důraz. Za rozumnou horní hranici potřebného času je považováno 30 minut.
			- Od systému je požadováno, aby byl schopen v rozumném čase přenášet dokumentaci na uzel, který si ji vyžádá. *V rozumném čase* znaméná řádově desítky sekund, horní hranice je jedna minuta.
		- **Reliability**
			- Kladen větší důraz a dostupnost WEB API než na systém přednosu dokumentace. 
		- **Interoperability**
			- Kladen důraz na jednoduchou změnu implementaci komunikace s centrálním WEB API u doktorských IS. Jednoduché zabezpečené API (https), jednoduché zprávy ve standardizovaném formátu (XML).
			- Možnost parsovat interní formáty dokotrských IS na straně cenrálních IS.
		- **Security**
			- Systém bude manipulovat s citlivými daty a uchovávat je. Vymyslet způsob, jak se bránit DOS útokům, falešným zprávám, falešným doktorům, logovat přístupy pro ochranu osobních údajů.
		- Scalability
			- Už při návrhu je nutné počítat s velkým množstvím přenášených a uchovávaných dat. Velikost přenášených zpráv kolísat nebude, ale velikost ukládaných dat stále poroste.
			- Tedy spíše *horizontální škálování* (úložné kapacity) než *vertikální škálování* (přenosové kapacity)
		- Manageability
			- Systém se principiálně jen úložiště citlivých dat, moc lidí by jej spravovat nemělo.
	- ***User Qualities***
		- Usability 
			- Uživatelé přijdou do styku se systémem pouze prostřednictvím svých IS, které nejsou součástí toho systému. 

## DÚ 2 (do 29.11.2013)##
![Component Model](doc/ComponentModel.png)

Tento diagram komponent zachycuje pohled na architekturu jednoho centralizovaného úložiště a jeho komunikace s různýmy zdravotnickýmy informačnímy systémy. 

Každý zdravotní informační systém komunikuje s centralizovaným uzlem přes zabezpečené Web API. Každý požavek na centralizovaný uzel je před zpracováním verifikován - proběhne autorizace a autentizace zdravotnického systému, který požadavek vytovořil, zároveň proběhne kontrola proti zahlcení požadavky. Zdravotnický systém může vyžádat **Index** se záznamy pacienta, ktérého zrovna léčí, nebo přímo zvolenou **Dokumentace**.,

Centralizovaný uzel po verifikaci požadavku 
- na **Index**: vrátí **Index** se záznamy pacienta, který má u sebe uložen v lokální databázi
- na konkrétní záznam: se podle **Indexu** rozhodne, kde záznam vyzvednout
	- buď je uložen v lokální databázi uzlu
	- nebo je uložen v lokální cache uzlu (je potřeba ověřit, že není zastaralý)
	- nebo ví, na jakém vzdáleném uzlu je uložena, tak si o záznam řekne
		- požadavek na vzdáleném uzlu je verifikován stejným mechanizmem jako požadavek zdravotnického systému. 
		- poté, co obdrží záznam, si jej uloží do své cache	

Centralizované uzly zároveň v daném intervalu synchronizují **Index** mezi ostatnímy uzly. Každý tento požadavek s novýmy/aktualizovanými záznamy je verifikován na uzlu, na který požadavek dorazí, stejným verifikačním procesem, jako ostatní (dříve zmíněné) požadavky. 

#### Verifikace ####
Verifikační proces bude před každou akcí zavolanou přes Webové API kontrolovat, že ten, kdo akci volá, má správný login a token (pokud volající je zdravotnický IS) a nebo ověří, že volá jiný centrální uzel. Zároveň v rámci verifikace požadavku proběhne i kontrola před zahlcením. Prioritu budou mít požadavky, které přicházejí z adres, které komunikují méně, tedy minimálně zahlcují systém.

### Nahrání dokumentace ###
![Uploading Documentation](doc/DF_UploadDocumentation.png)

Tento diagram znázorňuje tok dat při nahrání dokumentace
- Doktor vytvoří/upravé ve svém IS, ten se pošle přes Web API určeného centralizovaného uzlu, kde se verifikuje.
- Pokud verifikace proběhne úspěšně, záznam se uloží a aktualizuje se **Index**.
 
![Uploading Documentation](doc/SQ_UploadDocumentation.png)

Tento diagram detailněji popisuje průběh vytvoření/aktualizace dokumentace pacienta.

### Vyžádání dokumentace ###
![Obtaining documentation](doc/DF_ObtainDocumentation.png)

Tento diagram ukazuje, jak probíhá získání dané dokumentace
- Doktor si nejprve vyžádá **Index** dokumentace o pacientovi
- Poté vybere konkrétní dokument, který ho zajímá, a centralizovaný uzel mu jej vyhledá a najde. Každý požadavek je verifikován.

![Obtaining documentation](doc/SQ_ObtainDocumentation.png)

Tento diagram detailněji popisuje průběh vyžádání dokumentace pacienta. 

### Distribuce Indexu ###
![Distributing Index](doc/SQ_DistributeIndex.png)

Tento diagram popisuje průběh synchronizace **Index**u mezi všemy uzly.
- Každý uzel zjistí ze své lokální databáze, které záznamy přibyly/byly změněny od poslední synchronizace a tyto záznamy (jako **Index**) rozešle na všechny ostatní uzly
- Zároveň si kontroluje potvrzení, kterými mu ostatní uzly dávají vědět, že zpracovaly nový obsah **Indexu**. V případě nedodržení potvrzení je **Index** na daný uzel posílán opakovaně, dokud se operace nepodaří. 

### Performance ###

Nejdůležitější výkonostní požadavky na systém jsou:

- Přenesení dokumentace na uzel, který si ji vyžádal, by mělo trvat desítky sekund, maximálně jednu minutu.
- Synchronizace indexů by měla mít zpoždění do 30 minut.

Zdůvodnění: Předpokládá se, že lékaři budou ochotní na přenesení dokumentace počkat, ale ne příliš dlouho. U synchronizace indexů není kladen důraz na rychlost, protože se nepředpokládá, že by se pacient rychle přesouval mezi lékaři náležejícími pod různé nemocnice.

Největšími potenciálními problémy s výkonen systému by mohly být nároky přenášení dokumentů mezi nemocnicemi a synchronizace indexu na připojení nemocnic. Následuje velmi hrubý odhad těchto nároků:

Předpoklady výpočtu (mnohé velmi nereálné, ale pro rámcový výpočet by měly postačovat):

- za rok proběhne v ČR 140 milionů ambulantních ošetření ([Zdravotnická ročenka ČR 2012](http://www.uzis.cz/publikace/zdravotnicka-rocenka-ceske-republiky-2012): 135 786 630)
- každé ošetření bude vyžadovat přenesení jednoho dokumentu mezi nemocnicemi
- všechna ošetření probíhají během 8 pracovních hodin 250 pracovních dnů roku (simulace špičky)
- v ČR se nachází zhruba 200 nemocnic ([Nemocnice v České republice v roce 2012](http://www.uzis.cz/rychle-informace/nemocnice-ceske-republice-roce-2012): 188)
- všechny nemocnine jsou rovnoměrně zatížené
- přenášené dokumenty mají velikost 1 MB
- záznam v indexu má 100 B

Z těchto předpokldů vychází, že v ČR probíhá 70 000 ošetření za hodinu = cca 20 ošetření za sekundu = cca 0,1 ošetření za sekundu v obvodu každé nemocnice. To znamená, že každá nemocnice bude potřebovat kapacitu 800 kb/s jak na upload, tak na download, což v dnešních podmínkách není sebemenší problém.

Vzhledem k velmi malé velikosti položek indexu bude potřeba k synchronizaci jedné nemocnice se všemi ostaními nemocnicemi cca 16 kb/s, což taktéž není problém.

Průběh zpracování požadavku na přenesení dokumentu, který není umístěný v lokální nemocnici, probáhá následovně:

1. přenos požadavku do lokální nemocnice
2. zpracování požadavku v lokální nemocnici
3. přenos požadavku do vzdálené nemocnince
4. zpracování požadavku ve vzdálené nemocnici
5. přenos dokumentu ze vzdálené nemocnince
6. zpracování dokumentu v lokální nemocnici
7. přenos dokumentu z lokální nemocnince

Pokud předpokládáme, že na přenos tohoto dokumentu je k dispozici 1 Mb/s ve všech zúčastněných síťových uzlech, tak kroky 5 a 7 každý budou trvat 8 s. Všechny ostatní kroky by měly proběhnout rychle, předpokládejme do 1 s. Dohromady na celý proces dostáváme 21 s, což je v rámci požadavků.

### Security ###

K zajištění bezpečnosti je potřeba určit, proti jakým druhům útoků se budeme bránit. Předpokládané druhy útoků jsou:

- útočník bude předstírat, že je nemocnicí, při komunikaci s jinou nemocnicí nebo s lékařem
- utočník bude předstírat, že je lékařem, při komunikaci s nemocnicí
- útočník je lékař a bude přistupovat k datům, která nepotřebuje

Pro zabránění prvního případu budou všechny nemocnice vybaveny certifikátem, který se bude ověřovat při každé komunikaci s jinou nemocnicí nebo s lékařem. Všechny certifikáty nemocnic budou podepsány centrální autoritou. Bude také existovat revokační seznam, pro případ kompromitace privátního klíče nemocnice.

Pro ochranu před druhým případem se bude používat kombinace přihlašovacího jména a hesla lékaře. Toto heslo bude platné jenom v nemocnici, pod jejíž obvod lékař patří. Nemocnice budou používat standartní praktiky pro ukládání hesel (ukládá se jenom hash hesla, používá se salt).

Předpokládá se, že lékař se už přihlašuje do svého systému heslem, takže heslo pro komunikaci s nemocnicí může být uloženo na jeho disku zašifrované heslem pro přístup k jeho systému.

Lékař nikdy nebude přímo komunikovat se vzdálenými nemocnicemi, takže tento případ není potřeba řešit.

Veškerá komunikace bude zašifrovaná pomocí TLS, v případě nemocnice pomocí klíče z jejího certifikátu.

Případ, že útočník je lékař (případně někdo, kdo získal jeho heslo) se bude řešit jen logováním: každý přístup do systému se bude logovat, podezřelé přístupy se budou vyšetřovat. Pokud se tímto způsobem zjistí neoprávněný přístup k dokumentům, danému doktorovi se zakáže přistupovat do systému.

Tady se počítá s tím, že přístup k datům může být důležitý. To znamená, že jakékoliv omezování přístupu je nevhodné a že zvýšené riziko neoprávněného přístupu je akceptovatelné.
 
### Availability ###

Snaha bude minimalizovat dobu neplánovaných výpadku i jejich četnost. Počítá se s pravidlnými údržbami, které se budou provádět mimo špičku, ideálně po půlnoci, kdy se předpokládá minimální počet požadavků na systém.

#### Výpadky ####
- plánované výpadky
	- hodina měsíčně na údržbu (kolem 1:00 ráno)
- neplnánované výpadky
	- 30 minut měsíčně

#### Výpočet dostupnosti ####
**MTBF** = předpokládáme polovinu měsíce = 360 hodin

**MTTR** = předpokládáme 3/4 hodiny (1 hodina plánovaný výpadek(údržba), 0,5 hodiny neplánovaný výpadek; průměr = 0.75 hodiny) = 0.75 hodiny

```Availability = MTBF / (MTBF + MTTR) = 360 / (360 + 0,75) = 0.9979```

Tedy celková dostupnost systému bude činit zhruba 99,8% procent. Tímto se náš systém bude řadit do třídy spolehlivosti 2 - Managed ([slidy RNDr. Jakub Lokoč, Ph.D.](https://docs.google.com/viewer?url=http://siret.ms.mff.cuni.cz/lokoc/UK/Transakce.pdf)).

#### Ochrana proti DoS ####
Systém bude obsahovat modul ```Traffic Control```, který bude kontrolovat zatížení serverů. V případě přetížení nebude zpracovávat některé požadavky.
