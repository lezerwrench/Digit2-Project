# Digit2-Project
TinkerCad: https://www.tinkercad.com/things/1Mt057K60O9-digit2km-authlink/editel?returnTo=%2Fdashboard%2Fdesigns%2Fall&sharecode=CkfA4hAt3SVMpb4P6cHDYEZodrtOHUB5ArnRTl8_82o

Kép a projektről:
<img width="2560" height="1208" alt="Digit2_KM Auth__Link" src="https://github.com/user-attachments/assets/9b3a2c35-67ee-4d13-9bda-7cb8cf6ff939" />


A kód elérhető kommentekkel a tinkercad oldalán keresztül. A használati utasítás az utolsó bekezdésben található.

Klímaszabályozó és automata öntözőrendszer

Környezetszabályozó berendezés, képes egy zárt ökoszisztéma (pl.: üvegház vagy beltéri növénytermesztő egység klimatikus tényezőinek önálló menedzselésére.
A rendszer két fő, egymástól funkcionálisan független, de egyazon vezérlőmagon osztozó alrendszerre tagozódik: a dinamikus hőmérséklet-kezelésre és az intelligens vízellátásra.

A hardveres megvalósítás alapját egy Arduino Uno R3 mikrovezérlő képezi.
A perifériák optimális sávszélességű kezelése és a vezetékezés minimalizálása érdekében a kijelzőrendszer az I2C (InégyzetC) buszrendszert használja,
ahol a fizikai réteg megosztásával két független LCD modul jeleníti meg a strukturált adatokat.

Hőmérsékletszabályozó Alrendszer

A hőmérséklet monitorozását egy analóg TMP36-os lineáris feszültségkimenetű érzékelő végzi. A szenzorból
érkező feszültségjelet a mikrovezérlő analóg-digitális átalakítója (ADC) mintavételezi. Mivel az analóg vonalak ki vannak téve a magas frekvenciás elektromágneses zajoknak,
a nyers adatok egy diszkrét mozgóátlag szűrőn haladnak keresztül. A szűrt érték képezi az alapját a diszkrét PID (Proporcionális-Integráló-Deriváló) algoritmusnak,
amely a PWM (Impulzusszélesség-moduláció) jel kitöltési tényezőjét változtatva vezérli a hűtőventilátort imitáló DC motort egy N-csatornás MOSFET (NMOS) teljesítményfokozaton keresztül.

Öntöző és Vízszintvédelmi Alrendszer

A talajnedvesség szintjének mérését egy analóg talajnedvesség-szenzor látja el.
A rendszer biztonságos működését egy HCSR04 ultrahangos távolságmérő garantálja, amely a víztartály folyadékszintjét méri felülről lefelé történő méréssel.
Ha a talajnedvesség a kritikus küszöbérték alá esik, az Arduino csak abban az esetben hozza működésbe a vízpumpát szimuláló motort LU-5-R relén keresztül,
ha a tartályban elégséges vízmennyiség áll rendelkezésre. Vízhiány esetén a rendszer letiltja a pumpát, és egy Piezo buzzer segítségével vészjelzést ad le, megelőzve a szivattyú szárazon futásból eredő károsodását.

Hardveres lábkiosztás és interfészek

<img width="1464" height="796" alt="image" src="https://github.com/user-attachments/assets/011a83a2-a07a-47ba-beb8-834a854cd048" />

Diszkrét Mozgóátlag Szűrő (Moving Average Filter)
Az Arduino nem egyetlen mérésre hagyatkozik, hanem folyamatosan megjegyzi az utolsó 15 mérést. Mindig kiszámolja ezek átlagát, és ezt írja ki. Ha az egyik mérés hibásan kiugró lenne, az átlagolás miatt a kijelzőn a hőmérséklet szabályozottan fog változni.

A Diszkrét PID Szabályozási Algoritmus
P (Azonnali): Minél messzebb van a környezet a célhőmérséklettől, annál jobban pörgeti a motort.
I (Múlt): Ha a ventilátor már pörög, de a hőmérséklet még mindig nem esik, az eltelt idő függvényében egyre több erőforrást ad neki.
D (Jövő): Figyeli, milyen gyorsan hűl a levegő, és ha már közeledik a célhoz, picit lefékezi a ventilátort, hogy ne hűtse túl a rendszert.

Anti-Windup és Korlátozások
Ha a rendszer tartósan meleg, a PID kód I (integráló) tagja hajlamos lenne a végtelenségig növekedni a memóriában.
Az Anti-Windup maximalizálja ezt az értéket. E nélkül a ventilátor akkor is maximális sebeségen működne, amikor a környezet már rég lehűlt,
mert a szoftvernek idő kellene, kiüríteni a felgyülemlett számokat.

Feszültség- és Fordulatszám-illesztés (RPM Leképzés)
Az Arduino PWM perifériája 8 bites felbontású, így a kimeneti kitöltési tényezőt [0, 255] közötti uint8_t (integer) érték határozza meg.
Ez a hardverközeli érték a felhasználó számára értelmezhetetlen. A kódban egy konstans alsó szoftveres limittel (pwmValue < 30) meghatározott az alsó tartomány, ezzel védve a hardvert a túlmelegedéstől.
Lineáris interpoláció: A vezérlőjelet a beépített map() függvénnyel transzformáljuk a fizikai ventilátor névleges fordulatszám-tartományára.

Az LU-5-R Relé Bekötési Logikája
Az Arduino 5V-os áramköre nem bír el egy vízpumpát. Ezért a pumpa egy külön 9V-os elemre csatlakozik Amikor az Arduino áramot ad a relé belső mágnesének, a mágnes behúzza a fém kapcsolót, bezárja a 9V-os kört, és a pumpa elindul.

Hangerő- és Frekvenciacsillapítás a Piezo Ágban
Egy 11k Ohm-os ellenállás a hangszóró előtt található. Ez lefojtja az Arduinóból kiáramló elektronokat, így kevesebb feszültség jut a csipogóra, amitől drasztikusan halkabb, elviselhetőbb lesz.

A hőmérséklet manipulálása:
Kattintson a TMP36 hőmérséklet-szenzor ikonjára.
A megjelenő csúszkát húzza a kívánt értékre (pl. 35°C fölé).

Elvárt rendszerreakció: A felső kijelzőn a hőmérséklet emelkedni kezd, a PID algoritmus beavatkozik: a PWM érték és az RPM ugrásszerűen megnő, és a ventilátor motorja felpörög. Ha a csúszkát visszahúzza 35°C alá, a motor fokozatosan lelassul, majd teljesen megáll.

A talajnedvesség manipulálása:
Kattintson a Talajnedvesség-érzékelő ikonjára.
Húzza a vízszintes csúszkát balra, a száraz, repedezett föld jel irányába (hogy az érték 400 alá essen).

Elvárt rendszerreakció: Amennyiben a tartályban van víz, a relé azonnal kattan, megjelenik az alsó kijelzőn az [O] jelzés, és a vízpumpa motorja működésbe lép. Ha a csúszkát jobbra húzza (ezzel szimulálva a sikeres öntözést), a pumpa azonnal lekapcsol.

Az ultrahangos folyadékszint-szenzor kezelése:

Kattintson a HC-SR04 ultrahangos távolságmérő ikonjára.
A kijelző alatt megjelenik a szenzor zöld színű, tölcsér alakú látómezeje, benne egy kis zöld ponttal, amely a tartályban lévő vízfelszín távolságát jelképezi.

A tartály feltöltése: Fogja meg az egérrel a zöld pontot, és húzza közelebb a szenzorhoz (60 cm alá). Ezzel azt szimulálja, hogy a tartály tele van vízzel. Ekkor a kijelzőn a VIZ: Rendben státusz lesz látható.
A tartály kiürítése: Húzza a zöld pontot távolabb a szenzortól (60 cm fölé). Ezzel azt szimulálja, hogy a vízszint kritikusan lesüllyedt, és a tartály kiürült.
