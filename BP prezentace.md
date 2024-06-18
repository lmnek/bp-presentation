
[Dobrý rady](https://www.herout.net/blog/2012/04/co-chci-slyset-u-statnic/)

Měření přednesu: 7 min 43 s - rychlé čtení bez videa 

# Scénář

## Úvodní slide

Vážený pane předsedo, vážená komise,

Jmenuji se Kryštof Gärtner a v rámci své BP jsem se věnoval vývoji běžecké mobilní aplikace, která využívá umělou inteligenci ke generování hlasového doprovodu k běhům.

## Motivace

*Nápad* na tento projekt ke mně přišel, že jsem v minulosti používal aplikaci na běhaní Nike Run Club, která obsahuje možnost zapnout si k běhu audio doprovod kouče. 

Funguje to tak, že si člověk vybere nějaké téma běhu nebo cíl a je následně *doprovázen nahrávkami profesionálního kouče*, které byly vytvořeny jen k tomuto účelu. 

Kouč dává běžci různé užitečné *rady*, zvyšuje *motivaci* když je potřeba a celkově přidává takový *zábavný element* k aktivitě. 

Problém tohoto doprovodu je, že nahrávky jsou opravdu *statické* a každý jednotlivý běh je při opakovaném přehrání vždy stejný, nezávisle na různém kontextu běhu. 

Tudíž mě napadlo, že *obsah* doprovodu by mohl být *dynamicky generovaný* pomocí umělé inteligence v reálném čase během běhu, aby mohl reagovat na změny jako jsou například na rapidní fluktuace v tempu běžce.

A poté, co jsem *prozkoumal* ostatní možnosti na trhu, tak jsem došel k závěru, že *žádná* aplikace momentálně nenabízí jakkoliv podobnou funkcionalitu. Dokonce neexistuje téměř žádný *výzkum* na využití umělé inteligence v koučovacím kontextu. 

Takže i přes to, že se s integrováním umělé inteligence do různých aplikací poslední dobou roztrhl pytel, tak se v tomto případě jedná o nový koncept a zatím neprozkoumanou oblast.

## Cíle

Moje práce se zabývá dvěmi hlavními tématy.

Za prvé vytvořit funkční prototyp běžecké aplikace, která bude mít kompletní základní funkcionalitu jako ostatní běžecké alternativy a zároveň bude integrovat již zmíněného AI kouče.

Druhým aspekt je provedení analýzy toho, jak se dají momentálně Velké jazykové modely integrovat do softwarových aplikacích. 

V tomto jsem se zaměřil na různá specifika, jako je porovnání různých modelů na trhu dle kvality, ceny, výkonu a zaměřil jsem se i na soukromí při využívání podobných technologií. Poté možnosti využít open source proti proprietary modelům a také self-hosting proti využívání API. Nakonec jsem rozebral integrační stránku, která se týkala validace a testování výstupů a taky různých prompt engineering technik pro zvyšování kvality výstupů.

## Postup

Aplikace byla vytvářena docela standartním způsobem pro vývoj softwarových projektů. Postupně jsem tedy prošel fázemi: 
- *Rešerše* a tvorby *požadavků*
- *Návrhu*, kde jsem vytvořil low fidelity wireframy aplikace a několik diagramů (jak je vidět zde)
- Poté nastala samotná *implementace*, při které jsem zároveň aplikaci *nasazoval*
- A nakonec jsem provedl *uživatelské testování* 

Architektonicky, projekt nejvíce připomíná strukturu *Client-Server*. Přičemž client je naše mobilní aplikace a server se stará o velkou část business logiky.

## Technologie

Zde jsou ty nejdůležitější technologie, které jsem využil k postavení celého systému.

*Mobilní aplikace* využívá frameworku React Native ve spojení s meta-frameworkem Expo a tím pádem cílí na obě platformy Android i iOS. 

*Server* běží v Node.js runtimu. *Perzistence* je zajištěna relační databází Postgres, s kterou server komunikuje přes Drizzle ORM. Dále pro například držení uživatelských relací je využívána instance Redis databáze.

Protože i na klientu a serveru jsem zvolil programovací jazyk *TypeScript*, tak jsem pro komunikaci mezi nimi využil tRPC (neboli TypeScript remote procedure call). To ještě ve spojení s Drizzle ORM zajišťuje, že projekt je typově ošetřen od databáze až na frontend. A jsem si jistý, že právě tohle mi to během vývoje ušetřilo hodně potu a slz, a umožnilo mi se pohybovat mnohem rychleji.

Systém komunikuje s několika *externími službami* nebo APIs, jako je autentifikační služba Clerk (JWT), samotné velké jazykové modely, text-to-speech služba a taky například API na získávání momentálního počasí dle polohy.

Pro server, databáze a ostatní služby jsem pro *hostování* využíval převážně serverless platformy, u kteréch jsou nízké vstupní náklady a dokáží se velmi jednoduše škálovat.  

## LLM

Teď přiblížím, jakým způsobem jsem integroval velký jazykový model.

Kouč vstoupí několikrát během aktivity a vždy zareaguje na určité změny, které se v běhu udály, a má monolog ohledně centrálního tématu daného běhu. 

1. Když se tedy nastal bod na přehrání audia, tak o něj mobilní aplikace požádá server.
2. Ten si uchovává data o běžci a o momentálním běhu, z kterých vytvoří prompt a zašle jej na API velkého jazykového modelu.
3. Využívám služby od OpenAI a také OpenRouter, který umožňuje připojení na libovolný model od velké skupiny providerů, takže zaručuje obrovskou flexibilitu. 
4. Vygenerovaný text pro momentální koučův vstup se poté přepošle do Text-to-speech služby, kde využívám AWS Polly. Služba už jenom navrátí odkaz na vygenerované audio, které mobilní aplikace přehraje. 

*Kontext* ke kterému má přístup model je například jméno běžce nebo počasí v místě, kde zrovna běží. 
*Průběžný kontext*, který se aktualizuje během aktivity obsahuje data o tempu, uběhnuté vzdálenosti a času rozdělené na segmenty.

(Přidal jsem také možnost generovaný obsah přizpůsobovat různými možnostmi v nastavení.
 Pro optimalizaci LLM dotazů jsem využil různé Prompt engineering techniky, například model nejprve na začátku nageneruje osnovu pro daný běh, které se při generování jednotlivých vstupů drží. )

Generovaný text také obsahuje SSML tagy, které upravují parametry pro text-to-speech službu, jako je hlasitost a tempo.

## Demo

Nyní ukážu, jak aplikace funguje na demonstračním videu. *Demo* bylo nahrávano na Iphone 13 mini při reálném běhu venku. Některé části budou z časových důvodů zrychlené.

Nejprve je potřeba se *přihlásit* nebo zaregistrovat. Samozřejmě aplikace funguje i pro více uživatelů zároveň.

Poté jdeme do *nastavení*, kde je možné poupravit nějaké parametry pro všechny běhy a zároveň si aplikaci přizpůsobit k větší personalizaci.
Například poslední možnost privátní režim, když je zapnuta, tak se AI modelu nezasílají žádná citlivá uživatelská data.

Tady je *hlavní obrazovka* pro zapnutí běhu, kde si nastavíme
- cíl běhu - v km nebo minutách
- typ běhu a potom téma, o kterém bude kouč primárně hovořit - v tomhle případě je to běhání a ekologie

Teď už běh *začíná* a nechám puštěný první a poslední koučův vstup.

Tady je *detail běhu* s naměřenými statistikami a mapou, která se sestavila z bodů sesbíráných GPS polohou

*Historii* všech předchozích si můžeme zobrazit na této obrazovce.

A nakonec už jen vymažeme nějaký běh a odhlásíme se z aplikace. 

## Výsledky

Celkově se mi povedlo splnit všechny požadavky a cíle, které byly definovány na začátku.
- V *textu* bakalářské práce jsem popsal každý krok vývoje.  
- Finální *aplikace* má kompletní sadu základních funkcionalit pro běžeckou aplikaci. Zároveň je uživatelské rozhraní čisté, jednoduché a intuitivní na používání.
- A hlavně jsem dokázal úspěšně integrovat generovaného kouče, což byl hlavní cíl tohoto projektu.

Kód jsem psal a strukturoval takovým způsobem, aby aplikace byla udržitelná a jednoduše *modifikovatelná* v budoucnu a volil jsem řešení, které podporují *škálování* pro větší množství uživatelů.

Během vývoje jsem aplikaci několikrát sám *testoval* v reálných podmínkách a nakonec jsem také provedl uživatelské testování s několika běžci.
Testeři nereportovali téměř žádné problémy s fungováním aplikace a veškerá jejich jejich zpětná vazba se týkala pouze nápadů, co by šlo do aplikace přidat a tím jsem si ověřil, že aplikace v momentálním stavu je kvalitně zpracovaná a příjemná na používání.

Možné *rozšíření* zahrnují například využití více vstupů od uživatele jako je hlas nebo kamera a kladení většího důrazu na personalizace pro jednotlivé uživatele.

Ale hlavní kroky s momentálním stavem aplikace v budoucnu by byly
- vymyslet *business model* pro financování aplikace, protože výdaje na provoz nejsou nulové (Mimo ceny za hostování, tak tady v tabulce můžeme vidět přibližné ceny pro jednotlivé modely za jeden 10 minutový běh, které vypočítal z několika měření spotřeby tokenů modelem)
- provést hlubší otestování, aby se vychytaly chybky
- a nakonec aplikaci vydat na app storech pro Android i iOS


## Last slide 

Abych to vše shrnul, tak se jednalo o nejkomplexnější projekt, na kterém jsem kdy sám pracoval a taky jsem během vývoje musel projít každou fází práce softwarovém projektu.

Výsledná aplikace je plně funkční a kompletní řešení běžecké aplikace s unikátním vlastností AI hlasového doprovodu a rád bych na tomto základu budoval i v budoucnu a dotáhl aplikaci do stavu, kdy jí budou moct používat normální uživatelé. 

Děkuji za pozornost a to je z mojí prezentace vše :)).


## Otázky

### 1
Proč 3 testeři?

Tady bych chtěl říct a zmiňuji to i v samostné práci, že tři testeři rozhodně *nejsou dostatečně* velký vzorek, aby dokázaly dělat nějaké konkrétní *závěry*. 

Nebudu tady předstírat, že jsem měl nějaké specifické důvody, proč testovaná skupina nebyla větší. Spíše se je kvůli tomu, že jsem testování prováděl až *po dokončení* aplikace a nezbývalo mi tolik času.

A právě kvůli tomu, že jsem takhle omezen, tak jsem se snažil, aby testování bylo co nejefektivnější. Takže

1. Vybíral jsem testery, aby každý z nich byl různorodý - jinou věkovou kategorii a zkušenost s běháním
2. Potom jsem místo, abych rozeslal třeba video nebo demo aplikaci několika lidem neosobně, tak jsem spíš chtěl provést hlubší testování. Takže jsem se s každým sešel osobně a provedli jsme jeden nebo dva normální běhy a nechal jsem je, aby nahlas říkaly svoje myšlenky a nakonec jsme měli konverzaci o tom, co by šlo přidat nebo zlepšit.
3. No a hlavní cíl testování bylo ji otestovat celkově jako finální produkt a ověřit, že tam nejsou žádné zásadní chyby a vše funguje, na což nebyla potřeba větší skupina.

### 2

Rozhodně to může ovlivňovat, jakým způsobem testeři aplikaci vnímají, protože už mají nějaké očekávání a osobní preference, které jsou ovlivněné aplikací co už používaly. 

Dělo se to i při mém testování, že veškerý feedback jednoho z testerů byl vždy propojen s aplikací Strava, i co se týče chybějících featur.

S tím se pojí i to, že uživatelé aplikací, kde neexistuje vůbec žádný hlasový doprovod, tak by byly z této aplikaci ještě víc unešeni 
ale na druhou stranu ti kteří, s tím mají už zkušenosti tak by mohly být více kritičtí, protože AI hlas částečně postrádá lidskost a obsah může být více průměrný a nezajímavý než od reálného kouče.

Naštestí všechny běžecké aplikace sdílí velmi podobné rozhraní pro ovládání a já jsem se toho držel, takže všichni by měli být schopni aplikaci používat bez problémů.

Hlavní nedostatky, které můžou vytknout obecně všichni uživatelé jiných aplikací, tak jsou:
- Podpora pro uživatelské rozhraní na hodinkách, protože více než polovina běžců kouká do mobilu jen na začátku a na konci
- Propojení účtů mezi aplikacemi, protože nikdo nechce odcházet z aplikace, kde už má uložené všechny data svých běhů a statisky, takže by bylo lepší zajistit propojení mezi aplikacemi, aby se každý běh automaticky exportoval

### Low vs high fidelity

Nejprve jsem udělal tyhle low fidelity wireframy 
-> prošel jsem Nielsenovy heuristiky a snažil jsem se, aby to Wireframy splňovaly
(např:
- Minimalstický design - žádné zbytečné informace
- Pomocné chyby, když něco nefunguje (jít do nastavení pro zapnutí polohy)
- Pomocné vysvětlivky pro jednotlivé nastavení
- Viditelný system status (menu + navigace + taby + stack layout - zpět
- Vystoupení z momentální akce + potvrzení)
)

-> konečná high fidelity prototyp vznikal až během vývoje ale základem byly wireframy

---

# Vypracování


Nezminovat osnovu !!! Neni potreba, vsichni studenti stejni

Cca 1 slide na minutu
Hesla misto souvetich
Vyzkouset doma a zmerit cas

1. Uvodni slide
2. Motivace
	- Obrazek / video s finalnim vysledkem?
	- Proc resime ulohu
	- Cim je unikatni, jaky je prinos
	- Co, jak
	- Nejdulezitejsi!!!!
3. Cile prace / Zadani prace
	- Co se v praci resi
	- Metodologie / postup
	- Nekopirovat zadani
- Postup
	- Jak reseno a proc?
	- Hlavni myslenky
	- Obrazky, videa, vzorce, ...
	- Nezabihat az moc de detailu!!! Neztracet cas, zadna teorie
5. Vysledky
	- Nejdulezitejsi!!!!
	- Jak jsem jich dosahl
	- Hlavni vysledky - strucne, prehledne, citelne
6. Shrnuti
	- Strucne shrnuti zavery prace
	- Dosazeny cile?
	- Pochvaleni
	- Smer dalsiho vyvoje

-> Video na konci? / Hezky obrazek behem cteni posudku


Otazky od oponenta 
- Citace vytky
- Prepsana reakce
- Klidne si priznat chybu / jak bych to delal lepe priste
- Argumentovat vecne, nehadat se
