# JTSpots-release — instruktioner för Claude

Ärver globala regler från `C:\claude\CLAUDE.md`. Denna fil innehåller endast JTSpots-release-specifikt.

**Detta repo är den PUBLIKA distributionsytan för JTSpots** — bara den färdigbyggda .exe-filen (som en GitHub Release-asset) + `README.md`/`MANUAL.md`. Ingen källkod ligger här och ingen ska läggas här.

## Förhållande till källkodsrepot

Källkoden ligger i `C:\claude\git\jtspots\` (repo `sm5tog/jtspots`), som är **privat sedan 2026-07-15** — vi marknadsför/länkar INTE till Python-varianten eller källkodsrepot härifrån. Detta repo (`sm5tog/jtspots-release`) är den enda platsen som ska delas ut publikt.

## Uppdateringsflöde (görs BARA på uttrycklig begäran)

Bygg om och publicera en ny release **bara när användaren uttryckligen säger att det är dags** — inte automatiskt vid varje ny version i källkodsrepot. Detta är ett medvetet val (bekräftat 2026-07-15): "vi ska jobba som vanligt med python. exe-filsuppdatering sker bara när jag säger att det är dags."

När uppmaningen kommer:
1. `python -m py_compile jtspots.py` i källkodsrepot (redan gjort som en del av vanligt jtspots-arbete, men dubbelkolla).
2. Bygg med PyInstaller, `--onedir --windowed --noconfirm --clean`, `--hidden-import pymysql --hidden-import websocket --hidden-import win32com.client` (dessa är lazy imports i källkoden — PyInstaller hittar dem annars inte via statisk analys, se jtspots/DOCUMENTATION.md).
3. Kopiera `jtspots_bandplan.json` + `lang/`-mappen in i dist-mappen bredvid .exe-filen (INTE `--add-data`, enklare att bara kopiera efteråt). Kopiera ÄVEN `default_settings.json` (i detta repo) → `jtspots_settings.json` i dist-mappen (tillagd v1.17.78) — en ny installation ska ha användarens riktiga regler + en förprogrammerad SK4BW-klusterpost (callsign/lösenord tömda) som förval, inte en helt tom regellista. Hålls uppdaterad manuellt (kopiera från `C:\claude\aktiv\jtspots\jtspots_settings.json`s `rules`-nyckel vid behov, men BARA `rules`/`clusters` — aldrig `callsign`/`qth_grid`/`was_pass`/andra personliga fält, se global regel 21).
4. **Testa att .exe:n faktiskt startar** (kör den, vänta några sekunder, kontrollera att den fortfarande kör och att den skapar sina egna settings/cache-filer i SIN EGEN mapp — bekräftar att `APP_DIR` löser rätt, se punkt nedan). Städa bort eventuella cache-/loggfiler .exe:n skapat under testet innan zip:ning — releasen ska levereras i "första körning"-skick.
5. Zip:a hela mappen, `gh release create vX.Y.Z <zip> --title "JTSpots vX.Y.Z" --notes "..."` i det här repot.
6. Uppdatera `MANUAL.md` om UI/funktioner ändrats sedan förra releasen (`Version covered:`-raden överst + relevanta avsnitt).
7. Uppdatera `README.md` bara om något strukturellt ändrats (releaselänken pekar redan mot `../../releases/latest`, kräver ingen uppdatering per version).

## Viktig teknisk detalj: `APP_DIR`

`jtspots.py` använder `APP_DIR = Path(sys.executable).parent if getattr(sys, 'frozen', False) else Path(__file__).parent` (tillagd v1.17.77) — alla settings/cache/logg-filer samt `jtspots_bandplan.json`/`lang/`-uppslag går via denna konstant. Utan den hade en PyInstaller-frozen .exe läst/skrivit fel plats (bundlens interna extraktionsmapp istället för .exe:ns egen mapp). Rör INTE denna logik utan att förstå varför den finns — se `jtspots/DOCUMENTATION.md`.

## MANUAL.md

Skriven för slutanvändaren (inte utvecklare) — engelska, beskriver installation/Config/regelmotor/DXpedition-spårning/Statistik-flik/felsökning. Håll den i synk med faktiska UI-ändringar i källkodsrepot, särskilt Statistik-fliken (§8) som ändras ofta. `jtspots/DOCUMENTATION.md` är utvecklarreferensen — duplicera inte det innehållet här, MANUAL.md ska förbli användarvänd.
