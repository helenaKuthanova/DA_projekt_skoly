# DA_projekt_skoly

1.	Důležitost všeobecného vzdělání pro prosperitu společnosti (příprava na VŠ)
2.	Jak se liší možnosti studentů v různých oblastech ČR dostat se na gymnázium (maturitní obor)
3.	Metody: 
a.	porovnání administrativních celků (kraje, okresy) – procento umístitelných studentů
b.	statistická analýza dostupnosti středního vzdělávání podle bydliště

dostupnost autem:
proměnné: kraj, velikost obce, čas dojezdu, vzdálenost, počet škol (limitovaný vzdáleností)
průměr (avg), možná medián – vzdálenost, čas vzhledem k velikosti obce a kraje
1.	kraj vs. Dojezd
2.	kraj vs. Vzdálenost
3.	velikost obce vs. Dojezd
4.	velikost obce vs. Vzdálenost
zkusíme boxplot

typy škol (napojení přes ID školy na tu velkou tabulku


## Zjišťování času dojezdu z obce do školy
## Příprava dat pro statistické zpracování
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


+ Plánovaly jsme měřit čas dojezdu veřejnou dopravou, což by pro naši cílovou skupinu středoškoláků dávalo větší smysl), ale v té době, kdy jsme potřebovaly, se nám nepodařilo najít vhodné API. Vyzkoušely jsme "Public Transit Routing API v8", ale z více než 400 kombinací obec-škola se nám vyhledaly spoje pouze pro cca 30 z nich. I když jsme nastavily maximální vzdálenost pro pěší chůzi (= 6000 m, přestože to už je spíš neakceptovatelné), pomohlo to jen málo - vyhledal se spoj asi pro 70 kombinací obec-škola.
++ pokusné měření mezi lokalitami Blatov (Praha 21) - Český Brod vyšlo správně.
++ API však zřejmě nepokrývá kompletní veřejnou dopravu v ČR.
++ Náhodným ověřením na jizdnirady.idos.cz jsme zjistily, že z některých obcí se člověk ve všední den ráno do dané školy prostě nedostane. => nezbytnost auta v malých obcích.
++ Další pokus byl s Routing API Playground (https://apidocs.geoapify.com/playground/routing/), ale nevyšel už pokus mezi zkušebními lokalitami, API zřejmě nepracuje s vlaky?
++ Tohle je určitě možný směr jak pokračovat - najít api na měření dojezdu veřejnou dopravou, protože u teenagerů je pravděpodobně primární volbou.


#### Proč tyto velikosti obcí??
+ vynechaly jsme největší obce (kvůli způsobu zpracování - redukujeme obec na jeden bod)
+ rozhodly jsme se pro porovnání nejmenších vesnic, větších vesnic a menších měst


#### Mezery
+ nevěnujeme se Praze - měřit dojezd autem nemá už vůbec smysl
