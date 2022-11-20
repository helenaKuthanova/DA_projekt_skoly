# Dostupnost středních škol v České republice

# to do: Statistické zpracování a Tablo
Dostupnost autem:
+ proměnné: kraj, velikost obce, čas dojezdu, vzdálenost, počet škol (limitovaný vzdáleností)
+ průměr (avg), možná medián – vzdálenost, čas vzhledem k velikosti obce a kraje
1.	kraj vs. Dojezd
2.	kraj vs. Vzdálenost
3.	velikost obce vs. Dojezd
4.	velikost obce vs. Vzdálenost
zkusíme boxplot

typy škol (napojení přes ID školy na tu velkou tabulku

# blog

## Východiska a výzkumné otázky
1.	Důležitost všeobecného vzdělání pro prosperitu společnosti (příprava na VŠ)
2.	Jak se liší možnosti studentů v různých oblastech ČR dostat se na gymnázium (maturitní obor)

## Metody 
a.	porovnání administrativních celků (kraje, okresy) – procento umístitelných studentů
b.	statistická analýza dostupnosti středního vzdělávání podle bydliště

## Volba tématu
Téma si nás našlo samo. Prvotní impulz - obě máme dite v 5. třídě, které by mohlo dělat přijímačky na osmileté gymnázium, a obě máme pocit, že v našem okolí žádná nejsou a všichni, kdo bydlí kdekoli jinde než my dvě, je na tom líp.
V projektu jsme se rozhody zaměřit se na čtyřleté střední školy:
+ jejich nabídka je pestřejší, nejde jen o gymnázia
+ v současné době je téma velice aktuální, protože 9. třídy dokončují nejsilnější populační ročníky posledních let
+ mluví se o tom, že by nějaké nové střední školy mohly vzniknout (na rozdíl od osmiletých gymnázií, u nichž se mluví o tom, že by měla zaniknout), tudíž bychom se mohly na základě dat pokusit vytipovat nějaký blank spot, kde škola očividně chybí a náš projekt by mohl mít i praktický dopad. 

## Zdroje dat
Ministerstvo školství (MŠMT) poskytuje Rejstřík škol a školských zařízení pro jednotlivé kraje ve formátu xml na https://data.gov.cz/ - souhrn veškerých vzdělávacích institucí od mateřských škol přes základní a střední až po základní umělecké školy.
Data o obyvatelstvu jsme čerpaly z Českého statistického úřadu (ČSÚ) - Obyvatelstvo podle pětiletých věkových skupin a pohlaví a Počet obyvatel v obcích České republiky k 1. 1. 2021, oboje ve formátu CSV. 
Z ČSÚ jsme používaly také číselníky s kódy krajů, okresů, obcí a příslušnost obcí ke krajům. Při členění Prahy na obvody jsme vycházely z Wikipedie (Administrativní dělení Prahy). 

## Příprava a čištění dat
Data jsme zpracovávaly v Pythonu, používaly jsme Jupyter Notebook.

### Počty teenagerů v jednotlivých krajích a okresech
+ skript: okresy.ipynb

V datech z ČSÚ jsme vyfiltrovaly věkovou kategorii 15–19 let (dále teenageři), která odpovídá věku studentů na středních školách, a celkové počty pro obě pohlaví.

Vytvořily jsme jednu tabulku s počty teenagerů ve 14 českých krajích. Druhou tabulku jsme vytvořily pro 76 okresů a k okresům jsme připojily hodnoty pro 10 pražských částí (obvodů). Za Prahu jsme zvolily obvody, protože podle zákona o územně správním členění státu 10 pražských obvodů odpovídá členění na úrovni okresů. Zvažovaly jsme i 22 městských částí, ale pak bylo v jednotlivých částech relativně málo osob v dané věkové kategorii ve srovnání s okresy. Hodnoty pro pražské obvody v tabulkách z ČSÚ nebyly, takže jsme je posčítaly z hodnot pro městské části.

### Střední školy a jejich zařazení ke kraji a okresu
+ skripty: skoly.ipynb, spojeni_skol.ipynb, roztrideni_skol_dle_oboru_podrobnejsi.ipynb, deleni_adresy_ulice_cp.ipynb

Pomocí modulu xml.etree.ElementTree jsme z rejstříku škol a školských zařízení extrahovaly pouze střední školy (kód C00), pro každou jsme si uložily kompletní adresu, kapacitu čtyřletého oboru a jeho kód. Pomocí modulu re a regulárního výrazu jsme vyčlenily samostatně název obce a samostatně poštovní směrovací číslo (PSČ). Dále jsme použily číselník České pošty a na základě názvu obce a PSČ jsme ke školám připojily příslušný okres. Tam, kde se okres přiřadit nepodařilo (většinou z důvodu, že škola uváděla jiné PSČ, než by odpovídalo podle České pošty), jsme ho doplnily ručně. Tím se nám školy rozdělily do skupin podle okresů. Postupně jsme takto zpracovaly všech 14 krajů a nakonec jsme je spojily do DataFramu s 805 středními školami.

### Získání souřadnic škol
+ deleni_adresy_ulice_cp.ipynb, GETrequestOSMapi.ipynb

Od jednoho z mentorů jsme dostaly skript s API na získání souřadnic. Jako vstup vyžadoval zvlášť hodnoty pro obec, ulici a číslo popisné, takže opět za pomoci regulárního výrazu jsme oddělily ulici a číslo popisné. Výstupem byl DataFrame obohacený o hodnoty zeměpisné šířky a délky pro každou školu.

### Rozdělení škol do kategorií
+ roztrideni_skol_dle_oboru_podrobnejsi.ipynb

Abychom mohly mezi sebou porovnávat různé typy škol, chtěly jsme je rozdělit do kategorií. Oficiální dělení je na gymnázia (K, 255 škol), úplné střední odborné vzdělání s maturitou (bez vyučení) (M, 511 škol) a úplné střední odborné vzdělání s odborným výcvikem a maturitou (L, 39 škol). Rozhodly jsme se pro vyčlenění dalších dvou skupin z kategorie "úplné střední odborné vzdělání s maturitou (bez vyučení)" -  informační technologie a obchodní akademie a veřejnosprávní činnost. Cýsledné použité dělení a počty zařazených škol jsou:
1. gymnázium   255
2. informační technologie   129
3. obchodní akademie a veřejnosprávní činnost   122
4. odborné vzdělání s maturitou   260
5. odborné vzdělání s odborným výcvikem a maturitou   39

## Zjišťování času dojezdu z obce do školy
+ skripty: vzdalenosti_univerzalni.ipynb, doprava_univerzalni.ipynb
+ Vycházíme ze seznamu obcí v daném kraji s celkovým počtem obyvatel (kód LAU 1).
+ Vytváříme sample:
++ 10 obcí v rozmezí 0-2000 obyvatel, 
++ 10 obcí 2000-10.000 obyvatel,
++ až 10 obcí s 10.000-50.000 obyvateli (ve většině krajů jich je méně než 10).
+ V Keboole dohledáváme k obcím souřadnice pomocí komponenty Geocoding Augmentation - jako vstup bere tabulku se samotnými názvy obcí.
+ Opět v Jupyteru kontrolujeme, zda ke všem obcím byly přiřazeny souřadnice a zda jsou všechny obce ve zpracovávaném kraji (Keboola vrací mj. název kraje) - chyby opravujeme ručně, souřadnice dohledáváme v Google Maps (v různých krajích 1-7 chyb).
+ Výstup z Kebooly spojujeme nazpět s tabulkou počtů obyvatel v obci.
+ Cross joinem spojujeme obce ze vzorku se všemi školami.
+ Funkcí haversine naměříme pro každou dvojici obec-škola vzdálenost vzdušnou čarou a následně vyfiltrujeme dvojice, kde vzdálenost je menší než 30 km (stanoveno arbitrárně, potřebovaly jsme nějak omezit množství dvojic).
+ S použitím https://transit.router.hereapi.com měříme pro tyto kombinace čas dojezdu autem, získáváme čas v sekundách pro všechny vyhledávané kombinace.
+ Abychom neztratily informaci o velikosti obcí, přiřazujeme kategorie - 1 pro nejmenší obce (do 2000 obyvatel), 2 pro střední (2000-10.000 obyvatel) a 3 pro obce s 10.000 až 50.000 obyvateli.
+ Výstup pro každý kraj ukládáme do tabulky a následně zpracováváme v Tableau.

### Proč ne veřejná doprava?
+ Plánovaly jsme měřit čas dojezdu veřejnou dopravou, což by pro naši cílovou skupinu středoškoláků dávalo větší smysl), ale v té době, kdy jsme potřebovaly, se nám nepodařilo najít vhodné API. Vyzkoušely jsme "Public Transit Routing API v8", ale z více než 400 kombinací obec-škola se nám vyhledaly spoje pouze pro cca 30 z nich. I když jsme nastavily maximální vzdálenost pro pěší chůzi (= 6000 m, přestože to už je spíš neakceptovatelné), pomohlo to jen málo - vyhledal se spoj asi pro 70 kombinací obec-škola.
++ pokusné měření mezi lokalitami Blatov (Praha 21) - Český Brod vyšlo správně.
API však zřejmě nepokrývá kompletní veřejnou dopravu v ČR.
++ Náhodným ověřením na jizdnirady.idos.cz jsme zjistily, že z některých obcí se člověk ve všední den ráno do dané školy prostě nedostane. => nezbytnost auta v malých obcích.
++ Další pokus byl s Routing API Playground (https://apidocs.geoapify.com/playground/routing/), ale nevyšel už pokus mezi zkušebními lokalitami, API zřejmě nepracuje s vlaky?
++ Tohle je určitě možný směr jak pokračovat - najít api na měření dojezdu veřejnou dopravou, protože u teenagerů je pravděpodobně primární volbou.

### Proč tyto velikosti obcí??
+ vynechaly jsme největší obce (kvůli způsobu zpracování - redukujeme obec na jeden bod)
+ rozhodly jsme se pro porovnání nejmenších vesnic, větších vesnic a menších měst

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




## Vizualizace
K vizualizacím jsme používaly Tableau.

## Mezery
+ nevěnujeme se Praze z hlediska dopravy - měřit dojezd autem nemá už vůbec smysl
+ Bereme celkový počet 15-19 let, přitom část jich odchází už na 8leté nebo 6leté gymnázium, část na SŠ bez maturity nebo na učňák, část jde pracovat. => vycházíme z toho, že poměry k porovnání se nemění.
