# Company Logos: a Claude skill that fetches real, official company logos

A Claude skill (Claude Code, Claude.ai and other Agent Skills clients) that fetches
**real, official company and brand logos** as SVG, for decks, documents, web pages and
designs. It never redraws or approximates a logo: if no official file is found, it says so.

Other logo skills *design* a new logo. This one *finds the real one*.

**Built for local markets.** Global logo libraries cover Google and Microsoft well, but miss
the local bank, utility or retailer your client actually works with. This skill searches
the company's own website and its **local-language Wikipedia** first, so a German, Brazilian,
Japanese or Israeli company is found in its own market, not only in English.

## Where the logos come from (first hit wins)

1. **The company's own website**: the header logo, plus an official white version for dark backgrounds
2. **A curated library** of 1,432 SVGs (~1,200 brands) from
   [kaankiziltug/logo-design-skill](https://github.com/kaankiziltug/logo-design-skill), pinned to one commit
3. **Wikipedia** infobox image: the local-language edition first, then English. The language
   comes from the name's script (Hebrew, Arabic, Cyrillic, Greek, Japanese, Korean, Chinese,
   Thai, Devanagari) or from the domain's country (`siemens.de` → German, `itau.com.br` → Portuguese)
4. **Wikidata** "logo image" property (Wikimedia Commons)
5. **Iconify**: the `logos` and `simple-icons` sets

**Every logo comes with a way to check it.** Unless the brand is a global household name
(Facebook, Google, Coca-Cola...), Claude gives you a link to the business's official website
next to its logo, so you can confirm it picked the right company and not a namesake.

It respects `robots.txt`, sends an honest User-Agent, and never bypasses bot protection.

## Install

**Claude Code**

```
/plugin marketplace add eilongross/company-logos-skill
/plugin install company-logos@eilongross
```

**Claude.ai / Claude desktop**: download the `skills/company-logos` folder as a ZIP
and upload it under Settings → Capabilities → Skills.

## Use

Just ask, in any language: *"Put the logos of Siemens, Itaú and KPMG on the title slide"*,
*"Logos der drei größten deutschen Banken"*, or *"תמשוך את הלוגו של חברת החשמל"*.

The skill needs Python 3 and network access. `rsvg-convert` and Pillow are optional:
they let it tell a white logo from a dark one and render a contact sheet for checking.

## Legal note

Logos are trademarks of their owners. Using a logo to refer to a company (a client
deck, a market map) is normally fine; implying a partnership or endorsement is not.
This skill fetches files, it does not grant any right to use them.

---

## בעברית

סקיל ל-Claude שמביא **לוגואים רשמיים ואמיתיים** של חברות בפורמט SVG, למצגות, מסמכים,
דפי אינטרנט ועיצובים. הוא אף פעם לא מצייר לוגו בעצמו. אם לא נמצא קובץ רשמי, הוא אומר את זה.

מקורות לפי סדר: האתר של החברה, ספרייה של 1,432 לוגואים, ויקיפדיה בשפה המקומית ובאנגלית,
ויקינתונים ו-Iconify. הדגש הוא על שווקים מקומיים: הסקיל מחפש קודם בוויקיפדיה בשפת החברה
(לפי הכתב של השם או לפי סיומת הדומיין), ולכן הוא מוצא גם בנקים, חברות תשתית וקמעונאים
מקומיים שספריות לוגו גלובליות מפספסות.

**התקנה ב-Claude Code:** שתי הפקודות שלמעלה.
**ב-Claude.ai:** מורידים את התיקייה `skills/company-logos` כ-ZIP ומעלים בהגדרות, תחת Skills.

לכל מותג שאינו מותג-על עולמי (כמו פייסבוק או גוגל) הסקיל מצרף קישור לאתר הרשמי של העסק,
כדי שתוכלו לוודא שזה הלוגו של העסק הנכון ולא של עסק אחר עם שם דומה.

לוגואים הם סימני מסחר של בעליהם. הסקיל מוריד את הקבצים ולא מעניק זכות שימוש בהם.

## License

MIT
