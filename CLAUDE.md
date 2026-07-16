# JTSpots-release — instruktioner för Claude

Ärver globala regler från `C:\claude\CLAUDE.md`. Denna fil innehåller endast JTSpots-release-specifikt.

**Detta repo är den PUBLIKA distributionsytan för JTSpots** — bara den färdigbyggda .exe-filen (som en GitHub Release-asset) + `README.md`/`MANUAL.md`. Ingen källkod ligger här och ingen ska läggas här.

## Förhållande till källkodsrepot

Källkoden ligger i `C:\claude\git\jtspots\` (repo `sm5tog/jtspots`), som är **privat sedan 2026-07-15** — vi marknadsför/länkar INTE till Python-varianten eller källkodsrepot härifrån. Detta repo (`sm5tog/jtspots-release`) är den enda platsen som ska delas ut publikt.

## Uppdateringsflöde (görs BARA på uttrycklig begäran)

Bygg om och publicera en ny release **bara när användaren uttryckligen säger att det är dags** — inte automatiskt vid varje ny version i källkodsrepot. Detta är ett medvetet val (bekräftat 2026-07-15): "vi ska jobba som vanligt med python. exe-filsuppdatering sker bara när jag säger att det är dags."

**KRITISKT (skärpt 2026-07-15 efter en överträdelse samma dag):** ett brett "kör allt"/"kör"-godkännande för kod-/config-ändringar i källkodsrepot ger INTE klartecken att också bygga om och releasa exe:n — det är ETT SEPARAT steg som kräver sin EGEN uttryckliga begäran ("bygg exe", "dags att uppdatera exe:n" e.d.), oavsett hur brett det tidigare godkännandet var formulerat. Anledningen är inte bara "invänta godkännande"-principen i sig, utan en konkret kostnad: att bygga+testa+paketera+releasa en exe innan de underliggande funktionerna ens hunnit provköras lokalt (i vanlig `python jtspots.py`-körning) slösar bort ett helt PyInstaller-byggpass + test + release i onödan om något behöver ändras efter den lokala provkörningen. Vänta alltså tills användaren SJÄLV har kört/testat ändringarna lokalt OCH uttryckligen bett om en ny exe-release, innan `pyinstaller`/`gh release create` körs igen.

När uppmaningen kommer:
1. `python -m py_compile jtspots.py jtspots_updater.py` i källkodsrepot (redan gjort som en del av vanligt jtspots-arbete, men dubbelkolla).
2. Bygg BÅDA med PyInstaller:
   - `jtspots.py`: `--onedir --windowed --noconfirm --clean --name jtspots`, `--hidden-import pymysql --hidden-import websocket --hidden-import win32com.client` (dessa är lazy imports i källkoden — PyInstaller hittar dem annars inte via statisk analys, se jtspots/DOCUMENTATION.md).
   - `jtspots_updater.py` (v1.17.86, se jtspots/DOCUMENTATION.md §2.9i): `--onefile --noconfirm --clean --name JTSpotsUpdater` — **INTE** `--windowed`, den ska visa en vanlig konsol (progress/prompten "Uppdatera/skriv över allt"). Kopiera den färdiga `JTSpotsUpdater.exe` in i `jtspots.exe`s dist-mapp, bredvid `jtspots.exe` — de ska ligga i SAMMA mapp/zip, aldrig som en separat nedladdning (det är det som gör att updatern kan uppdatera sig själv, se §2.9i).
3. Kopiera `jtspots_bandplan.json` + `lang/`-mappen in i dist-mappen bredvid .exe-filerna (INTE `--add-data`, enklare att bara kopiera efteråt). Kopiera ÄVEN `default_settings.json` (i detta repo) → `jtspots_settings.default.json` (OBS: `.default.json`, INTE `jtspots_settings.json` — bytt namn i v1.17.85 efter en bekräftad bugg där den gamla namngivningen skrev över en befintlig användares riktiga inställningar vid uppdatering, se jtspots/DOCUMENTATION.md §2.9i/versionshistorik) i dist-mappen. Hålls uppdaterad manuellt (kopiera från `C:\claude\aktiv\jtspots\jtspots_settings.json`s `rules`-nyckel vid behov, men BARA `rules`/`clusters` — aldrig `callsign`/`qth_grid`/`was_pass`/andra personliga fält, se global regel 21).
4. **Testa att båda .exe-filerna faktiskt fungerar:**
   - `jtspots.exe`: kör den, vänta några sekunder, kontrollera att den fortfarande kör och att den skapar sina egna settings/cache-filer i SIN EGEN mapp (bekräftar att `APP_DIR` löser rätt, se punkt nedan). Städa bort eventuella cache-/loggfiler den skapat under testet innan zip:ning.
   - `JTSpotsUpdater.exe`: helst ett fristående test i en TOM mapp med bara `default_settings.json` kopierad in som `jtspots_settings.default.json` — kör updatern (installationsläge, ingen `jtspots.exe` finns än), bekräfta att den hämtar+packar upp senaste release och startar jtspots.exe. Ett uppdateringstest (befintlig installation, `--update`) är svårare att göra fullständigt innan releasen faktiskt finns på GitHub — verifiera då åtminstone att extraheringen INTE rör en riktig `jtspots_settings.json` och att self-update-swapen (`_finish_self_update`) fungerar mot den egna gamla exe-filen.
   - Releasen ska levereras i "första körning"-skick — inga kvarlämnade cache-/loggfiler i dist-mappen.
5. Zip:a hela mappen (jtspots.exe + JTSpotsUpdater.exe + `_internal`/bandplan/lang/settings.default), `gh release create vX.Y.Z <zip> --title "JTSpots vX.Y.Z" --notes "..."` i det här repot.
6. Uppdatera `MANUAL.md` om UI/funktioner ändrats sedan förra releasen (`Version covered:`-raden överst + relevanta avsnitt).
7. Uppdatera `README.md` bara om något strukturellt ändrats (releaselänken pekar redan mot `../../releases/latest`, kräver ingen uppdatering per version).

## Viktig teknisk detalj: `APP_DIR`

`jtspots.py` använder `APP_DIR = Path(sys.executable).parent if getattr(sys, 'frozen', False) else Path(__file__).parent` (tillagd v1.17.77) — alla settings/cache/logg-filer samt `jtspots_bandplan.json`/`lang/`-uppslag går via denna konstant. Utan den hade en PyInstaller-frozen .exe läst/skrivit fel plats (bundlens interna extraktionsmapp istället för .exe:ns egen mapp). Rör INTE denna logik utan att förstå varför den finns — se `jtspots/DOCUMENTATION.md`.

## MANUAL.md

Skriven för slutanvändaren (inte utvecklare) — engelska, beskriver installation/Config/regelmotor/DXpedition-spårning/Statistik-flik/felsökning. Håll den i synk med faktiska UI-ändringar i källkodsrepot, särskilt Statistik-fliken (§8) som ändras ofta. `jtspots/DOCUMENTATION.md` är utvecklarreferensen — duplicera inte det innehållet här, MANUAL.md ska förbli användarvänd.
