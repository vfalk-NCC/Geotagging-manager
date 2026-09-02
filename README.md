# Site Photos – Trimble Connect-tillägg

Ladda upp geotaggade platsfoton, transformera GPS-koordinater (WGS84) till
projektets SWEREF-system med explicit EPSG-kod, och se dem som klickbara
markörer i 3D-modellen.

## Så här används det

1. Klicka **"🌐 Koordinatsystem"** och välj rätt SWEREF99-zon för projektet
   (eller ange en egen EPSG-kod + proj4-definition). Detta styr hur
   GPS-koordinater räknas om till projektets koordinater — **kontrollera
   mot en känd punkt i modellen om du är osäker på vilken zon som gäller.**
2. Klicka **"+ Ladda upp foton"** och välj en eller flera bilder.
3. För varje foto:
   - Om bilden har GPS-data i EXIF läses den in automatiskt och visas med
     en grön "GPS hittad"-badge.
   - Saknas GPS visas "Ingen GPS" — klicka **"📍 Placera manuellt i
     3D-vyn"** och klicka sedan på rätt plats i modellen.
   - Fyll i beskrivning och taggar om du vill.
4. Klicka **"Spara alla"**. Varje foto placeras som en markör i 3D-vyn
   (se avsnittet om `addIcon` nedan för hur detta fungerar tekniskt).
5. **Klicka på en foto-markör i 3D-vyn** (om `viewer.onIconPicked` fungerar
   i din version) eller på **"🔍 Visa"** i listan för att öppna en
   förstorad vy med all metadata, redigera beskrivning/taggar, zooma till
   fotot, eller ta bort det.

## Vad som sparas per foto

Photo ID, filnamn (file reference), miniatyrbild, tidsstämpel,
originalkoordinater (WGS84 lat/lon + höjd om tillgänglig), transformerade
projektkoordinater, vilken EPSG-kod som användes, kamerariktning (om
tillgänglig i EXIF), beskrivning och taggar.

## Varningar om osäkerhet

Verktyget flaggar automatiskt när:
- Fotot saknar GPS och placerats manuellt.
- GPS-koordinaten ligger utanför ett rimligt intervall för Sverige (kan
  tyda på fel EXIF-data eller att bilden är tagen någon annanstans).
- GPS-höjd saknas (Z sätts då till 0 som platshållare) eller finns men är
  av naturliga skäl ofta osäker (±10–50 m är vanligt för telefonkameror).

Varningarna visas i den förstorade vyn för varje foto.

## Foto-markörer i 3D-vyn

Efter mycket testande visade sig `viewer.addIcon()` vara **opålitlig** —
samma anrop med identiska parametrar kunde både lyckas och misslyckas
(`InvalidStateError: source image could not be decoded`) mellan körningar,
och även när anropet "lyckades" syntes ingen markör. Istället används nu
samma beprövade tekniker som redan fungerar väl i de andra tilläggen:

- En liten **treaxlig kryssmarkering** (linjemarkeringar) på fotots position.
- En **textetikett** ("📷 filnamn") som flyter strax ovanför.
- Om kamerariktning finns i EXIF: en kort **pil** som visar riktningen
  fotot togs mot (0° = norr, standard SWEREF/UTM-konvention).

**Att klicka på markören i 3D-vyn öppnar inte fotot** — till skillnad från
den ursprungliga idén med `addIcon` + `onIconPicked` går det inte att
tillförlitligt särskilja klick på en linje-/textmarkering från andra klick
i vyn. Markören är alltså en **visuell platsangivelse**; öppna det
förstorade fotot via **"🔍 Visa"** i listan längst ner istället — det
fungerar garanterat.

## Installation

Samma mönster som övriga tillägg: ladda upp `index.html` + `manifest.json`
till en egen hosting/repo, uppdatera `url`/`icon` i manifestet, och lägg
till raw-länken till manifestet via **Project Settings → Apps &
Capabilities → Add Custom**.

## Tekniska val & kända begränsningar

- **Koordinattransformation (proj4 + explicita SWEREF-EPSG-koder) är den
  säkra, verifierade delen** av detta tillägg — SWEREF99-zonernas
  proj4-parametrar är stabila, publikt dokumenterade värden från
  Lantmäteriet, inte en gissning mot Trimbles API.
- **Foto-markörer i 3D-vyn ritas nu med beprövade linje-/textmarkeringar**
  (samma teknik som kommentar- och mätverktygen) istället för `addIcon`,
  efter att den senare visat sig opålitlig i praktiska tester. Se avsnittet
  "Foto-markörer i 3D-vyn" ovan för detaljer.
- **Ingen persistent lagring ännu.** Precis som de andra tilläggen är detta
  just nu session-only (foton och markörer försvinner vid omladdning).
  Eftersom foto-loggen rimligen ska bestå över tid är nästa naturliga steg
  att koppla på samma fil-uppladdningslager vi byggde för
  PDF-overlay-verktyget (ladda upp bild + en liten JSON-fil med metadata
  per foto till projektets filer, läs in dem igen vid uppstart) — men det
  bygger i sin tur på att den experimentella TC-filåtkomsten faktiskt
  fungerar i din miljö. Säg till när du testat grundfunktionerna här, så
  kopplar vi på det.
- Manuell placering återanvänder samma `viewer.onPicked`-mönster som
  övriga verktyg — kräver klick på synlig geometri/punktmoln.
