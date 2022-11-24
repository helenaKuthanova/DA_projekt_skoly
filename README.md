# Dostupnost středních škol v České republice

# blog

## O projektu
1.	Důležitost všeobecného vzdělání pro prosperitu společnosti (příprava na VŠ)


Téma si nás našlo samo. Prvotní impulz - obě máme dite v 5. třídě, které by mohlo dělat přijímačky na osmileté gymnázium, a obě máme pocit, že v našem okolí žádná nejsou a všichni, kdo bydlí kdekoli jinde než my dvě, je na tom líp.
V projektu jsme se rozhody zaměřit se na čtyřleté střední školy:
+ jejich nabídka je pestřejší, nejde jen o gymnázia
+ v současné době je téma velice aktuální, protože 9. třídy dokončují nejsilnější populační ročníky posledních let
+ mluví se o tom, že by nějaké nové střední školy mohly vzniknout (na rozdíl od osmiletých gymnázií, u nichž se mluví o tom, že by měla zaniknout), tudíž bychom se mohly na základě dat pokusit vytipovat nějaký blank spot, kde škola očividně chybí a náš projekt by mohl mít i praktický dopad. 


2.	Cílem našeho projektu bylo zmapovat, jak se liší možnosti studentů v různých regionech ČR dostat se na maturitní obor a jaká je nabídka oborů v krajích. V první části projektu jsme z dostupných dat porovnaly kapacitu škol v jednotlivých regionech s počtem obyvatel ve věku 15-19 let. Ve druhé části jsme provedly analýzu vzorku obcí různých velikostí ze všech krajů s cílem zjistit, jaký vliv má bydliště na možnosti výběru střední školy. 

-----------------------------------------DONE---------------------------------------------------

## Zdroje dat
Ministerstvo školství poskytuje Rejstřík škol a školských zařízení pro jednotlivé kraje (xml). Z Českého statistického úřadu jsme použily tabulky Obyvatelstvo podle pětiletých věkových skupin a pohlaví, Počet obyvatel v obcích České republiky k 1. 1. 2021 (cvs), číselníky krajů, okresů, obcí a příslušnosti obcí ke krajům. Použily jsme číselník České pošty spojující poštovní směrovací číslo a obec. Při členění Prahy na obvody jsme vycházely z Wikipedie.

## Zpracování dat a výpočty
Data jsme zpracovávaly v Jupyter Notebooku, k vizualizacím jsme používaly Tableau.

### Počty teenagerů v jednotlivých krajích a okresech
V datech z ČSÚ jsme vyfiltrovaly počty osob ve věkové kategorii 15–19 let (dále teenageři), která odpovídá věku studentů středních škol. Vytvořily jsme jednu tabulku s počty teenagerů pro 14 krajů, druhou jsme vytvořily pro 76 okresů a k okresům jsme napojily 10 pražských obvodů (zvolily jsme obvody, protože podle zákona o územně správním členění státu odpovídají úrovni okresů).

### Střední školy - zařazení k okresu, získání souřadnic, rozdělení do kategorií
Pomocí modulu xml.etree.ElementTree jsme z rejstříku škol a školských zařízení extrahovaly střední školy (kód C00), pro každou jsme uložily adresu, kapacitu čtyřletého oboru a jeho kód. Pomocí modulu re a regulárního výrazu jsme vyčlenily název obce a PSČ. Dále jsme s využitím číselníku České pošty na základě názvu obce a PSČ ke školám připojily okres. Tam, kde se okres přiřadit nepodařilo (škola uváděla jiné PSČ, než by odpovídalo podle ČP), jsme ho doplnily ručně. Tím se školy rozdělily do skupin podle okresů. Postupně jsme zpracovaly všech 14 krajů a nakonec jsme je spojily do DataFramu s 805 středními školami.

**XXX** obrázek xml_tree

Od Jirky Macourka jsme dostaly skript s API na získání souřadnic. Jako vstup vyžadoval zvlášť hodnoty pro obec, ulici a číslo popisné, které jsme izolovaly opět pomocí regulárního výrazu. Výstupem byl DataFrame obohacený o hodnoty zeměpisné šířky a délky pro každou školu.
Za účelem porovnání různých typů škol jsme je rozdělily do kategorií. Oficiální dělení je na gymnázia, úplné střední odborné vzdělání s maturitou (bez vyučení) a úplné střední odborné vzdělání s odborným výcvikem a maturitou. Rozhodly jsme se vyčlenit další dvě skupiny z kategorie úplné střední odborné vzdělání s maturitou (bez vyučení) - informační technologie a obchodní akademie a veřejnosprávní činnost.

### Čas dojezdu z obce do školy
V datech z ČSÚ jsme vyfiltrovaly obce v daném kraji (kromě Prahy) a celkový počet obyvatel. Metodou sample jsme pro každý kraj vybraly vzorek 10 obcí s méně než 2000 obyvateli, 10 obcí s 2000-10.000 obyvateli a až 10 obcí s 10.000-50.000 obyvateli (ve většině krajů méně než 10). Zvolily jsme tyto kategorie, protože předpokládáme, že ve větších městech jsou problémy s dopravní dostupností méně výrazné. V Keboole jsme k obcím dohledaly souřadnice pomocí komponenty Geocoding Augmentation, která bere jako vstup název obce. Zkontrolovaly jsme, zda ke všem obcím byly přiřazeny souřadnice a zda jsou obce ve správném kraji - chyby jsme opravily ručně podle Google Maps. Výsledek jsme opět napojily na tabulku počtů obyvatel v obci. Cross joinem jsme spojily vzorek obcí se všemi školami. Pomocí modulu haversine jsme naměřily pro každou dvojici obec–škola vzdálenost vzdušnou čarou a následně vyfiltrovaly dvojice, kde je vzdálenost menší než 30 km. Tím jsme získaly tzv. blízké školy.
Původní plán byl měřit čas dojezdu z obce do školy veřejnou dopravou, ale v době, kdy jsme potřebovaly, se nám nepodařilo najít vhodné API. Od mentorů jsme dostaly dva tipy – Public Transit Routing API v8 z více než 400 kombinací obec–škola vyhledalo spoje pouze pro 10 %, nepomohlo nastavit maximální vzdálenost pro pěší chůzi (6000 m). Pokus s <a href='https://apidocs.geoapify.com/playground/routing/'>Routing API Playground</a> také nevyšel.
Nakonec jsme s použitím <a href='https://transit.router.hereapi.comAPI změřily dojezd autem a získaly čas v sekundách pro všechny vyhledávané kombinace.

**XXX** obr vzdalenost_api

Podle velikosti obcí jsme přiřadily kategorie, 1 – nejmenší, 2 – střední a 3 – největší obce a vytvořily jsme tabulku se všemi kraji, kterou jsme použily k vizualizacím v Tableau.

--------------------------------------------DONE-----------------------------------------------------

Dopsat: výběr 5 nejbližších škol

Výsledky analýzy vzorku obcí jsme take statisticky zpracovaly. 


## Statistická analýza
### Data: 

Používáme dva vzorky: 

1.	celý vzorek získaný v poslední fázi sběru dat: 10 malých, 10 středních a až 10 větších obcí v každém kraji (kromě Prahy). V tomto vzorku využíváme tato data: 
+ název obce, ze které se dopravní dostupnost sleduje
+ kraj, ve kterém se obec bydliště nachází
+ všechny školy dostupné z každé zkoumané obce: název obce a ID školy
+ čas cesty z obce bydliště do obce školy
Tento vzorek má celkem 357 obcí, a 10351 řádků (párování každé obce se všemi školami do vzdálenosti 30 km).

2.	redukovaný vzorek založený na předchozím datasetu: pro každou obec je vybráno jen 5 nejbližších škol.
 
Motivací pro použití obou vzorků je zjistit, jak může zvolená metoda ovlivnit výsledky analýzy. Domníváme se, že plná data (vzorek 1) mohou ukázat plnou škálu možností v dané obci, zatímco redukovaná data (vzorek 2) jsou vhodnější při porovnávání času dojezdu z jednotlivých obcí. 

### Hypotézy:
1. čas dojezdu do škol v jednotlivých krajích:
+ Nulová hypotéza: Mezi kraji není v dojezdovém čase statisticky významný rozdíl.
+ Alternativní hypotéza: Mezi kraji je v dojezdovém čase statisticky významný rozdíl.

2.	čas dojezdu do škol podle velikosti obce: 
+ Nulová hypotéza: Dojezdová vzdálenost není lineárně závislá na velikosti města.
+ Alternativní hypotéza: Dojezdová vzdálenost je lineárně závislá na velikosti města.

### Metody a statistické testy: 
Nejprve jsme prozkoumaly vzorek dat a zjistily, že data o dojezdových vzdálenostech nemají normální rozdělení ani v jednom vzorku (test_normality_dat.ipynb)
Proto jsme pro statistické ověřování zvolily následující testy (statisticke_testy.ipynb): 

1.	rozdíly mezi kraji: 
+ Kruskal-Wallis H-test: test ukázal, že rozdíly mezi kraji jsou statisticky signifikantní. Můžeme tedy zamítnout nulovou hypotézu a předpokládat, že dojezdové časy se v krajích liší. 
+ Test však neříká, kde přesně se vzorky liší a je tedy potřeba provést sérii dalších testů, při kterých jsou zkoumány kraje po dvojicích, ve kterých zjišťujeme, kde jsou rozdíly signifikantní. Tento test naznačil, že rozdíly mezi kraji s blízkými hodnotami mediánu nejsou statisticky signifikantní, ale rozdíly mezi kraji vzdálenějšími v hodnotě mediánu (3 místa od sebe) už často signifikantní jsou. Nejedná se tedy o jeden nebo dva kraje vybočující z řady, ale rozdíly napříč většinou krajů. Na základě této části můžeme tvrdit, že existují rozdíly v dostupnosti středních škol v závislosti na kraji bydliště.

2.	test závislosti času dojezdu na velikosti obce: 
+ jako test korelace jsme použily Spearmanův koeficient. Výsledek testu (koeficient -0.08, p < .001) vypovídá o velmi slabé (téměř nulové) korelaci mezi veličinami, statisticky je ale výsledek signifikantní. To znamená, že díky velkému množství dat je výsledek výpočtu velmi přesně odpovídající distribuci v populaci, ale protože je korelace extrémně slabá, nemá velikost obce téměř žádný vliv na délku cesty do školy.
+ Korelaci jsme dále zkoušely měřit (stejným testem) s mediánem za každou obci, korelace byla stále slabá negativní, a výsledek signifikantní
+ Když se však v testu použil vzorek 2 (tedy pouze 5 nejbližších škol na obec), tak byla korelace středně silná negativní, statisticky signifikantní (koeficient -0.38, p < 0.001). 
+ Na základě dat 5 obcí, jsme si v pythonu vyzkoušely vykreslit i regresní model. 

Data ze vzorku 1 pravděpodobně reflektují relativně husté osídlení v ČR (obce nejsou dostupností automobilem vázány jen na jedno město v okolí). Vzorek 2 ale ukazuje, že ve větších obcích jsou nejbližší školy dostupné za kratší dobu než v menších obcích.


-------------------------
# VYNECHÁME, JEN ZATÍM NEMAŽU
## Mezery
+ nevěnujeme se Praze z hlediska dopravy - měřit dojezd autem nemá už vůbec smysl
+ Bereme celkový počet 15-19 let, přitom část jich odchází už na 8leté nebo 6leté gymnázium, část na SŠ bez maturity nebo na učňák, část jde pracovat. => vycházíme z toho, že poměry k porovnání se nemění.

### Proč ne veřejná doprava?

Tohle je určitě možný směr jak pokračovat - najít api na měření dojezdu veřejnou dopravou, protože u teenagerů je pravděpodobně primární volbou.

### Proč tyto velikosti obcí??
+   

XXX Výsledné dělení a počty zařazených škol:
1. gymnázium   255
2. informační technologie   129
3. obchodní akademie a veřejnosprávní činnost   122
4. odborné vzdělání s maturitou   260
5. odborné vzdělání s odborným výcvikem a maturitou   39

Zvažovaly jsme 22 městských částí, ale pak bylo v jednotlivých částech relativně málo osob v dané věkové kategorii ve srovnání s okresy.