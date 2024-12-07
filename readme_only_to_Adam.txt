Részletes leírás a kód működéséről és jellemzőiről
1. Feladat és cél
A kód célja Sentinel-2 műholdfelvételek osztályozása két kategóriába:

CLEAR: Árvízmentes terület.
FLOODED: Árvízzel érintett terület.
A modell tanítása figyelembe veszi az osztályok közötti aránytalanságot, valamint támogatja az egyedi képek és teljes adathalmaz osztályozását. Az eredmények CSV formátumban menthetők, illetve egy Gradio interfész segítségével webes osztályozás is lehetséges.

2. Adatok előkészítése
Adatok forrása:

A képek a SEN12FLOOD mappából kerülnek betöltésre. Minden mappa egy képpárt tartalmaz:
Eredeti képek: stack.tif.
Augmentált képek: astack.tif.
A címkék a flooding.txt fájlokból származnak:
"True": FLOODED (osztály: 1).
"False": CLEAR (osztály: 0).
Adatok partíciózása:

Az adatok 70%-a a tanítókészletbe, 30%-a a tesztkészletbe kerül.
A mappák sorrendjét véletlenszerűen határozza meg.
Kép előfeldolgozás:

Az egycsatornás (grayscale) képek 4 csatornássá alakulnak, hogy egységesek legyenek.
Az összes képet normalizálja, a 16 bites értékeket 
0
−
1
0−1 tartományba skálázva.
Augmentáció:

Minden képpárhoz tartozik egy augmentált változat. Ez a feldolgozás független a címkétől, az augmentáció minden képre vonatkozik.
Az augmentált képek ugyanazt a címkét kapják, mint az eredetiek.
Osztályarányok:

CLEAR osztály: 2186 kép (76,6%).
FLOODED osztály: 668 kép (23,4%).
3. Adatok kiegyensúlyozása
Osztályarányok kezelése:

A tanítókészlet kiegyensúlyozására súlyozott mintavételt használ.
Az egyes osztályok súlya fordított arányban áll a mintaszámukkal, így a kisebb osztály (FLOODED) nagyobb súlyt kap.
WeightedRandomSampler:

A tanító adathalmazból véletlenszerűen választ mintákat, az osztályarányok súlyozása alapján.
Ez biztosítja, hogy a minibatch-ek egyensúlyban legyenek az osztályok között, még akkor is, ha az alap adathalmaz aránytalan.
4. Modell architektúra
Konvolúciós rétegek:

Az első réteg 4 csatornás bemenetet fogad, és 16 csatornás kimenetet hoz létre.
A második réteg 16 csatornás bemenetből 32 csatornás kimenetet generál.
Normalizáció és aktiváció:

Minden konvolúciós réteg után Batch Normalization javítja a tanulás stabilitását.
Aktivációs függvényként ReLU-t használ.
Lehúzás (MaxPool):

A rétegek közötti lehúzás 
2
×
2
2×2-es méretű MaxPool rétegekkel történik.
Teljesen összekötött rétegek:

Az FC réteg bemeneti dimenziója dinamikusan számított, így a modell különböző méretű képekkel is kompatibilis.
A bemeneti dimenzió után 64 neuront tartalmazó réteg következik, amelyet Dropout réteg egészít ki, csökkentve a túltanulás kockázatát.
Kimeneti réteg:

A kimeneti réteg 2 neuronból áll, amelyek a CLEAR és FLOODED osztályokhoz tartoznak.
Súlyinicializáció:

Minden konvolúciós és teljesen összekötött réteg súlyait Xavier-uniform inicializációval állítja be, amely stabilabb tanulást eredményez.
5. Tanítás és validáció
Tanítási folyamat:

A tanítás során keresztentrópia veszteségfüggvényt használ, amely a különböző osztályok predikciós pontosságát méri.
A pontosság (accuracy) mérésére a torchmetrics.Accuracy osztályt használja.
Validáció:

A validáció során ugyanazt a veszteségfüggvényt és pontosságmérőt használja, mint a tanítás során.
A validációs eredmények folyamatosan logolásra kerülnek.
Optimalizálás:

Az Adam optimalizálót használja, amely dinamikusan állítja a tanulási rátát.
A tanulási ráta csökkentésére ReduceLROnPlateau ütemezőt alkalmaz, amely a validációs veszteség stagnálása esetén lép működésbe.
Korai megállás:

Ha a validációs veszteség 3 egymást követő epoch során nem javul, az edzési folyamat automatikusan leáll.
6. Kimenet és eredmények mentése
Modell mentése:

A betanított modell súlyai elmentésre kerülnek, amely később betölthető osztályozási feladatokhoz.
Validációs eredmények:

A modell folyamatosan naplózza a validációs veszteséget és pontosságot, amely a legjobb modell kiválasztását segíti.
CSV export:

A tesztkészlet kiértékelése során a modell osztályozási eredményeit (predikció és valós címke) egy CSV fájlba menti.
7. Osztályozás és kiértékelés
Gradio interfész:

Egyéni képek feltöltésére és osztályozására alkalmas webes felületet biztosít.
A TIFF képeket előfeldolgozza (csatornák és normalizáció), majd a modell osztályozza őket.
Az osztályozási eredményként a CLEAR vagy FLOODED kategória jelenik meg.
Teszthalmaz kiértékelése:

Az összes tesztkép osztályozásra kerül, az eredmények tartalmazzák:
A mappa nevét.
Az előrejelzett osztályt.
A valós osztályt.
Támogatott képformátum:

Csak a TIFF képek támogatottak, és minden kép 4 csatornával kerül feldolgozásra.
Trükkök és jellemzők
Osztály kiegyensúlyozás:

Súlyozott mintavétel (WeightedRandomSampler) biztosítja az osztályok egyenletes eloszlását.
Az augmentált képek növelik az adatok változatosságát.
Dinamikus architektúra:

A teljesen összekötött réteg automatikusan igazodik a bemeneti dimenzióhoz, így a modell rugalmasan kezel különböző méretű képeket.
Xavier inicializáció:

Stabilabb és gyorsabb tanulást tesz lehetővé az egyenletes súlyeloszlás révén.
Gradio interfész:

Könnyen használható webes platformot biztosít egyéni képek osztályozására.
CSV export:

Automatikusan generált osztályozási eredmények, amelyek a tesztkészlet kiértékeléséhez ideálisak.