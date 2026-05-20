# Digit2-Project
TinkerCad: https://www.tinkercad.com/things/lreR6KvCfE7-projekt/editel?returnTo=https%3A%2F%2Fwww.tinkercad.com%2Fdashboard&sharecode=cMU24VVHH2WTGlFJjVd5TvDfiT8bUhMbrMwmSxaqBqA
A kód elérhető kommentezve a tinkercad oldalán keresztül.

Klímaszabályozó és automata öntözőrendszer

Környezetszabályozó berendezés, képes egy zárt ökoszisztéma (pl.: üvegház vagy beltéri növénytermesztő egység) klimatikus tényezőinek önálló menedzselésére.
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
