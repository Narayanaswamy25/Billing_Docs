# Internationalization and Localization

| Topic | Rule |
| --- | --- |
| Launch languages | English, Hindi, Tamil |
| Screen vs document language | Separate settings: a bill can print in Tamil while the screen is English |
| Amount in words | English, Hindi or Tamil, following the **document** language |
| Strings | ARB files (Flutter, ICU messages), written short first; layouts allow longer text; switching language redraws at once |
| Fonts | Checked for line height and joined letters; names typed in one language show correctly on another screen |
| Numbers | Indian grouping by default (12,34,567.50) with a company switch to international grouping |
| Dates | Day first (dd-mm-yyyy) for India; stored UTC with a business date in the company time zone |
| Currency | INR only in release 1 (multi-currency is out of scope) |
| Plain vs technical words | See [08-ui-ux-design-system.md](08-ui-ux-design-system.md); the Accountant keeps the technical names |
| Directionality | Use directional padding APIs so right-to-left can be added later |

Translation file layout (proposed): `l10n/app_en.arb`, `app_hi.arb`, `app_ta.arb` per client package; no hardcoded user-facing strings. Test amount-in-words and fonts in each language before launch.
