# Dostupnost středních škol v České republice

# blog

## O projektu
Tématem našeho projektu je středoškolské vzdělávání v Česku. Vzdělání považujeme za jeden ze základních pilířů prosperity společnosti. Rovnost v přístupu ke vzdělání je považována za důležitou podmínku kvalitního školského systému. Toto téma je velmi aktuální, protože v příštím roce se bude na střední školy hlásit nejsilnější populační ročník za 29 let, a jak varuje např. analytik Daniel Hůle, školy nejsou na tento nápor připravené. Problémů s regionálními nerovnosti si všímá i MŠMT. Vzniká také pnutí mezi Středočeským krajem a Prahou, která podle některých supluje potřeby studentů ze Středočeského kraje. (**do blogu přidat odkazy**)

V projektu se zaměřujeme na čtyřleté maturitní obory, které představují přímou cestu k terciálnímu vzdělávání. Naším cílem je zmapovat, jak se liší možnosti studentů v různých regionech ČR dostat se na maturitní obor a jaká je nabídka oborů v krajích. Projekt má dvě části: v první části z dostupných dat porovnáme kapacitu škol s počtem obyvatel ve věku 15-19 let v dané oblasti. Ve druhé části analyzujeme vzorek obcí různých velikostí ze všech krajů s cílem zjistit, jaký vliv má bydliště na možnosti výběru střední školy. 

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

Podle velikosti obcí jsme přiřadily kategorie, 1 – nejmenší, 2 – střední a 3 – největší obce a vytvořily jsme dataset se všemi kraji, obsahující celkem 357 obcí, a 10351 řádků. V něm jsme dále označily pro každou obec pět škol s nejkratším časem dojezdu (pokud nebylo k dostupných 5 škol, vybraly jsme všechny školy) a vytvořily jsme si tak vzorek, který nám umožnil analyzovat dostupnost nejbližších škol v každé obci, celkem obsahuje tento dataset 357 obcí, a 1817 řádků. Tento dataset jsme použily ke statistické analýze a k vizualizacím v Tableau.


--------------------------------------------DONE-----------------------------------------------------

## Statistická analýza
### Data: 
Ke statistické analýze jsme tedy používaly tyto dva datasety:
1.	celý vzorek vybraných obcí z každého kraje kromě Prahy 
2.	vzorek pěti nejbližších škol pro každou obec bydliště 

V obou vzorcích jsme využily tato data: 
+ název obce bydliště
+ kraj, ve kterém se obec bydliště nachází
+ všechny školy dostupné z každé obce bydliště: obec a ID školy
+ čas cesty z obce bydliště do obce školy

### Hypotézy:
V první části jsme zkoumaly rozdíly mezi jednotlivými kraji a ověřovaly následující hypotézu:
+ Mezi kraji je v dojezdovém čase statisticky významný rozdíl.

V druhé části jsme analyzovaly vztah mezi časem dojezdu a velikostí obce a naše hypotéza byla tato:
+ Dojezdová vzdálenost je lineárně závislá na velikosti města.

### Metody a statistické testy: 
Nejprve jsme zjistily, že data o dojezdových vzdálenostech nemají normální rozdělení ani v jednom vzorku.
(**vložit test_normality_dat.ipynb**)
Proto jsme pro statistické ověřování zvolily testy nepředpokládající normální rozdělení.

#### Rozdíly mezi kraji
Kruskal-Wallis H-test ukázal, že rozdíly mezi kraji jsou statisticky signifikantní. Můžeme tedy zamítnout nulovou hypotézu a předpokládat, že dojezdové časy se v krajích liší. 
Test však neříká, kde přesně se vzorky liší, a je tedy potřeba provést sérii dalších testů (Mann–Whitney U), při kterých jsou zkoumány kraje po dvojicích a my zjišťujeme, zda jsou rozdíly signifikantní jen v určité části vzorku, nebo napříč celým vzorkem. Zjistily jsme, že se nejedná o jeden nebo dva kraje vybočující z řady, ale rozdíly jsou signifikantní napříč většinou krajů. Na základě této části můžeme tvrdit, že existují rozdíly v dostupnosti středních škol v závislosti na kraji bydliště.

#### Rozdíly dané velikostí obce: 
Jako test korelace jsme použily Spearmanův koeficient. Výsledek testu datasetu všech dostupných škol (koeficient -0.08, p < .001) vypovídá o velmi slabé (téměř nulové) korelaci mezi veličinami a velikost obce nemá podle testu téměř žádný vliv na délku cesty do školy.
Korelaci jsme otestovaly i na druhém datasetu (pět nejbližších obcí), a zde byla korelace středně silná negativní, statisticky signifikantní (koeficient -0.38, p < 0.001). V pythonu jsme si také vykreslily jednoduchý lineární regresní model. 
(**vložit plot s trendline**)

Téměř nulový vztah zkoumaných veličin v případě prvního datasetu nás překvapil, protože jsme očekávaly, že velikost obce bude mít na dojezdovou vzdálenost vliv. Domníváme se, že je to způsobeno hustým osídlením v ČR (nejen z menších obcí, ale i z měst jsou dostupná další města v okruhu 30 km). Druhý test už ale v souladu s očekáváním ukazuje, že pokud se zaměříme pouze na pět nejbližších škol, ve větších obcích se do nich studenti zvládnou dostat za kratší dobu.

------------------------- asi víceméně DONE :-) -----------------------------------------------------
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