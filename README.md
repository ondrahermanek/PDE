PDE - Patient Documentation Exchange
===

# Team #
- Petr Onderka (gsvick (at) gmail (dot) com)
- Filip Řepka (repik (dot) x (at) seznam.cz)
- Ondra Heřmánek (ondra (dot) hermanek (at) gmail.com)

# Situace #
Doktor při každé návštěvě (nového) pacienta zjišťuje jeho anamnézu - zdravotní stav, předchozí nemoci, vyšetření, operace, rodinnou anamnézu a podobně. Pacient navíc nemusí všechny informace vědět, nebo může něco úmyslně zatajit. Navíc tento proces doktorovi zabírá nemalé množství času.

Současný stav výměny dokumentace mezi doktory je následující: když doktor posílá pacienta za jiným doktorem na nějaké vyšetření, vytiskne mu jeho kartu, kterou musí pacient donést tomu jinému doktorovi. Tak karta ale obsahuje jen informace od prvního doktora. Po vyšetření by pacient kartu měl správně odevzdat svému obvodnímu doktorovi, ale často se tak neděje a pacienti si kartu nechávají, ztrácí nebo vyhazují.

O řešení tohoto problému se již v minulosti snažil projekt **IZIP**, který neuspěl především z důvodu, že systém byl centralizovaný v rukou pojišťoven, které kontrolovali, jestli doktoři dělají, co vykazují, že dělají.

# Motivace #
Cílem tohoto projektu je navrhnout systém, který umožní doktorům získat celkovou dokumentaci pacienta a obohacovat ji. Systém bude o každém pacientovi obsahovat seznam všech návštěv u doktorů, popis vyšetření a nějakou výstupní zprávu. Tato dokumentace bude přístupná na vyžádání - doktor při vyšetření, běhěm výjezdu sanitního vozu.

# Ideální svět #
Všichni dokotři a zdravotnická zařízení mají telefon a přístup k počítači se zabezpečeným internetovým připojením, díky kterému budou moci pacientovu dokumentaci sdílet. Všichni doktoři se sdílením dokumentace souhlasí, všichni pacienti také. 

# První návrh řešení #
## Centralizované / distribuované řešení ##
Systém může uchovávat dokumentaci na jednom místě, to má ale mnoho nevýhod - dostupnost, komunikační náročnost, ochrana osobních údajů, veliké množství dat. Tato centra se dají duplikovat - ochrana proti výpadku, ale tím dostáváme distribuovaný systém, kterému roste náročnost na synchronizaci mezi jednotlivými uzly.

Proto navrhujeme udělt kompromisní řešení - centralizovat dokumentaci v daných uzlech a distribuovaně šířit pouze **Index** - seznam pacientů a jejich navštěv u doktorů a kódu vyšetření. 

## Index ##
Pro **Index** očekáváme řádově desítky milionů záznamů, proto bude kladen důraz na minimální velikost jednoho záznamu. Ideálně tedy: 

<pre>ID pacienta | ID vyšetřujícího doktora | Datum vyšetření | ID zákroku | ID zařízení, kde je dokumentace uložena</pre>

Jako ID pacienta by šlo použít rodné číslo, pokud by se dalo předpokládat, že je unikátní. Protože to ale neplatí, použijeme rodné číslo kombinované s celým jménem pacienta.

Index lze šířit dvěma způsoby:

- Synchronizovat mezi uzly automaticky, což je komunikačně náročné.
- Poskytovat až na výžádání dotazem na centralizovaný uzel typu "Dej mi všechna vyšetření tohoto pacienta, které máš u sebe uchované". Tento dotaz bude odeslán na všechny (doktorem vybrané) uzly až když pacient přijde na vyšetření. Na odpověď tohoto dotazu bude potřeba počkat.

## Dokumentace ##
Dokumentace se bude uchovávat centralizovaně v daných uzlech - zdravotnická zařízení nebo zdravotní pojišťovny nebo obvodní lékaři. Doktoři budou dokumentaci na tyto uzly nahrávat - doktoři v nemocnicích přímo v rámci nemocničního systému, soukromí doktoři přes nějaké webové rozhraní nebo pomocí svých informačních systémů, které budou s tímto celým systémem komunikovat. Jednotlivé uzly se budou starat o zabezpečení osobních údajů, o dostupnost dokumentace a o aktualizaci **Index**u.

Logicky nejsprávnější by bylo uchovávat dokumentaci paciantů u jejich obvodních lékařů, kteří by o svých pacientech měli vědět vše (tedy mít veškerou jejich dokumentaci). To by znamenalo vybavit každého doktora serverem, který bude idálně 24/7/356 dostupný a zabezpečený proti zneužití.

## Přístupnost dokumentace ##
### Jak bude dokumentace přístupná? ###
- Všichni vidí vše - porušení ochrany osobních údajů
- Nikdo nevidí nic - nebylo by co sdílet
- Přístupné jen něco - relevantní data mohou být zrovna nepřístupná
- Výchozí dostupnost?

Bude potřeba, aby někdo řekl, co je dovoleno sdílet, co není dovoleno sdílet, zároveň by měl existovat způsob, jak se dostat k relevantním informacim, které mouhou být nepřístupné jen díky špatnému nastavení.

### Kdo bude určovat, kdo co má vidět a kdo ne? ###
- Pacient určuje, co bude dostupné a co ne - jak to změní?
- Doktor určuje, co bude dostupné a co ne - měl by mít pacientův souhlas?

Asi by mělo záležet na domluvě doktora a pacienta, co se může sdílet a co ne. Doktor by se měl snažit sdílet maximum relevantních informací (on ví, co je relevantní lépe, než pacient).

### Jak dočasně zpřístupnit dokumentaci? ###
Případ "Pacient si přinese kartu od jednoho doktora k druhému pro vyšetření" se dá nahradit jako vyměnění elektronického povolení mezi doktory pro zobrazení dokumentace pacienta.

# Případy použití #

## Vyšetření pacienta ##
Pacient přijde k doktorovi, ten si pomocí **Index**u zjistí, kde všude byl na vyšetření a jaké to bylo (jen podle kódu vyšetření). Pokud nějaké vyšetření bude doktora zajímat, vyžádá si celý záznam o vyšetření od zdravotního střediska, kde bylo vyšetření provedeno a které tedy den záznam vyšetření shraňuje. 

Odpovědí na vyžádání tohoto záznamu bude buď samotný záznam, který si doktoru bude moci přečíst, nebo informace, že záznam není k dispozici a telefonní číslo na doktora, který záznam pořídil. To je nutné, aby v případě potřeby se dalo aspoň telefonicky informace o vyšetření předat.

## Nahrání dokumentace do centralizovaného úložiště ##
Vyšetřující doktor provede vyšetření pacientovi. Vyplní informace o vyšetření do svého systému. Systém může dokumentaci rovnou nahrát do určeného úložistě (online). Nebo být schopen vyexportovat data v takovém formátu, aby se tato data dala nahrát do určených uzlů (offline)

Zdravotnické informační systémy: Medicus, ...

# DÚ 1#
- Modularizace - schéma základního rozdělení na moduly a jejich vazby
![](https://raw.github.com/onashackem/PDE/master/doc/ComponentModel.png?login=onashackem&token=bf9fed6fcded562c4f64a474d4f98640)

- [Kvalitativní atributy](http://msdn.microsoft.com/en-us/library/ee658094.aspx)
	- *Design Qualities*
		- **Conceptual Integrity**
			- Takto robustní systém je potřeba dobře rozmyslet a navrhnout, usnanovit jednotný coding style, navrhnout základní postupy při vývoji, strategii a organizaci týmu, ... 
		- Maintainability
			- Nebude snadné jednoduše provést aktualizaci distribuovaného systému, takže by se na možná rozšíření mělo myslet už při návrhu (například počítat s podporou obrázků v dokumentaci pacienta).  
		- Reusability
			- Systém bude nabízet API pro komunikaci s ostatnímy systémy
	- *RunTime Qualities*
		- **Availability**
			- Měl by být kladen důraz na dostupnost API, která sbírají data od doktorů a ukládají je do nemocničních IS (99%).
			- Dostupnost dokumentace z nemocničního IS už nemusí být tak ostře hlídána (95%).  
		- **Performance**
			- Distribuovaný systém vyžaduje už nějakou režii na synchronizaci uzlů, je potřeba minimalizovat velikost a počet zpráv pro synchronizaci (**Index**), aby systém zvládl v rozumném čase přenášet i dokumentaci na uzel, který si ji vyžádá. *V rozumném čase* znaméná řádově desítky sekund. Horní hranice je minuta až dvě.
		- **Reliability**
			- Kladen větší důraz a dostupnost WEB API než na systém přednosu dokumentace. 
		- **Interoperability**
			- Kladen důraz na jednoduchou změnu implementaci komunikace s centrálním WEB API u doktorských IS. Jednoduché zabezpečené API (https), jednoduché zprávy ve standardizovaném formátu (XML).
			- Možnost parsovat interní formáty dokotrských IS na straně cenrálních IS.
		- **Security**
			- Systém bude manipulovat s citlivými daty a uchovávat je. Je potřeba
		- Scalability
			- Už při návrhu je nutné počítat s velkým množstvím přenášených a uchovávaných dat. Velikost přenášených zpráv kolísat nebude, ale velikost ukládaných dat stále poroste.
			- Tedy spíše *horizontální škálování* (úložné kapacity) než *vertikální škálování* (přenosové kapacity)
		- Manageability
			- Systém se pripiálně jen úložiště citlivých dat, moc lidí by jej spravovat nemělo.
	- *System Qualities*
		- Supportability
			- Moc ne
		- Testability
			- Moc ne 
	- *User Qualities*
		- Usability 
			- Uživatelé přijdou do styku se systémem pouze prostřednictvím svých IS, které nejsou součástí toho systému. 
 
