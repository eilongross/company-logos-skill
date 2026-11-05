---
name: "company-logos"
description: "Fetch official company/brand logos as SVG: the company's own website first, then a 1,432-brand SVG library, local-language and English Wikipedia, Wikidata, Iconify. Strong on local and regional companies (banks, utilities, retailers) that global logo libraries miss. Use whenever a deck, doc, page or design needs a real company logo, in any market or language."
---

# Company logos

Use this whenever the user asks for a company's logo, or a deliverable (deck, doc, one-pager, dashboard, design) needs real client/partner/competitor logos ("get me Siemens' logo", "put the local banks' logos on the slide", "logo wall"), in whatever language the user writes. Pull the real file. Never redraw, trace or approximate a real logo in SVG/CSS; if no official file is found, say so and leave a labeled placeholder.

## Sources (in order, first hit wins)

1. **The company's own website** (most authoritative and current). Domain comes from `"Name=domain"` on the command line, or from Wikidata P856 (official website) via the Wikipedia article. The script reads the homepage and picks the header logo (`<img>` or inline `<svg>` whose own tag or parent says "logo"; icon-sized SVGs, partner/client/award/social/app-store images are excluded). It downloads the top candidates, keeps the dark/colour version as the main file and an official white (reversed) version as `-white` for dark slides.
2. **Library**: `kaankiziltug/logo-design-skill` reference library. 1,432 curated SVGs, ~1,200 brands (tech/SaaS heavy: Google, Microsoft, monday, Wix...). Pinned to commit `1ec463a`; files fetched one by one from raw.githubusercontent.com (no 24 MB clone). Many brands have a `main` lockup and an `-icon` symbol.
3. **Wikipedia infobox image** via the REST API, the company's **local-language edition first**, then English. Best for local and non-tech organisations (banks, utilities, retailers, consultancies) that global libraries miss. `--lang` sets the editions explicitly; otherwise the local edition comes from the name's script (Hebrew, Arabic, Cyrillic, Greek, Japanese, Korean, Chinese, Thai, Devanagari), else from the pinned domain's country TLD (`.de`, `.fr`, `.com.br`, `.co.jp`...); with neither, English only.
4. **Wikidata P154** ("logo image") resolved to Wikimedia Commons.
5. **Iconify** `logos` (colour) and `simple-icons` (mono) sets.

If the site gives only a raster (PNG), the script also fetches the best SVG from the next sources as `alt-...svg`. Use the SVG only if it visually matches the site's current logo; otherwise use the site's PNG.

## Website rules (legal and polite)

- Using a company's logo to refer to that company (client deck, market map, competitor comparison) is normally fine; implying a partnership or endorsement that doesn't exist, or using it as the user's own branding, is not. For public marketing material (e.g. an "our clients" wall) check consent and the user's organisation's own brand rules, which override this skill.
- Honest User-Agent, robots.txt respected. If a site blocks (403, bot check, JS-only page, robots disallow), the script reports it and moves on. **Never bypass bot protection**: no fake browser headers, no scraping tricks. Banks, utilities and some consultancies often block automated requests; for those, Wikipedia/Wikidata usually have the current logo.
- If the site has a brand/press/media-kit page, its files beat the homepage: WebSearch `<company> brand guidelines logo download` or `<company> media kit`, download from that page, and follow its usage rules.
- Opening a blocked site in the browser as a normal visit is acceptable only when the user asks for it; don't automate around the block.

## Wikipedia rules (adapted from cyanheads/wikipedia-mcp-server and its wikipedia-guide)

- Coverage differs by edition. For a local company the local-language edition is often more current than English, which can keep a logo from before a rebrand for years. When currency matters, fetch both (e.g. `--lang de` and `--lang en`) and compare.
- The infobox image is sometimes a photo (HQ building, CEO). The script only accepts files that are SVG or named "logo" (in the common local spellings); anything else is a last resort flagged `VERIFY`.
- Wikimedia throttles shared cloud IPs hard (waits of 30-70 s are normal): batch all brands into one run instead of many runs.

## Steps

1. Write the script below to `fetch_logo.py` in the working directory (once per session) and run it with all brands at once. **For clients, always pin the domain** (exact, faster, avoids namesakes and the Wikipedia lookup):
   `python3 -I fetch_logo.py "Siemens=siemens.de" "Itaú Unibanco=itau.com.br" "KPMG=kpmg.com" "トヨタ自動車=toyota.jp" --out logos`
   Without a domain the script resolves it from Wikidata (needs a correct Wikipedia match; generic names like "monday" may hit the wrong article, so pin them).
   Options: `--variant icon|main|all` (library symbol vs lockup), `--source website|library|wikipedia|wikidata|iconify`, `--lang xx[,yy]` (Wikipedia editions in order, e.g. `--lang pt,en`; applies to the whole run, so for a Latin-script local company without a pinned domain run it separately with its language).
   Output lines: `OK` (file, source, format, matched entity, `site:` the business's official website), `FAIL` (download error), `MISS`, plus `!` notes on blocked sites. A `manifest.json` in the out folder records the source URL of every file.
2. **Look at every logo before using it.** Render a contact sheet and view it: `rsvg-convert -h 120 X.svg -o X.png` for each file, tile with PIL, put `-white` files on a dark tile, and open the PNG. Check: right company (compare with `matched`; not a product, subsidiary or namesake), it is the logo (not a photo or icon), current version, and not a white version shown on white.
3. On MISS / FAIL / outdated logo: try the brand/press page (above), the other Wikipedia edition (`--lang`), the local-language or English legal name (e.g. "Deutsche Telekom AG" vs "Telekom", or the name in its own script), or `--variant all`. Then WebSearch `<brand> logo svg site:commons.wikimedia.org`; download via `https://commons.wikimedia.org/wiki/Special:FilePath/<File_name.svg>` and verify the same way.
4. Using the files: keep SVG for web pages, artifacts and Slides/Docs types. For .pptx/.docx, convert to high-res PNG (`rsvg-convert -h 600`) unless the target supports SVG. Keep aspect ratio; never recolour, stretch or crop an official logo unless the user asks. On dark backgrounds use the official `-white` file if one was found.
5. Report briefly: which logos came from where (site / library / Wikipedia...), which sites blocked, anything flagged, anything not found.
6. **Give a website link for every brand that is not a global household name**, so the user can confirm it is the right business. Global household names (Facebook, Google, Apple, Microsoft, Amazon, Coca-Cola, Nike, McDonald's and the like, recognised on sight worldwide) need no link. Everything else does: local and regional companies, small businesses, anything with a common or generic name, anything with a namesake, and anything you are not sure about. When in doubt, give the link.
   - The link is the `site:` field of the output, put next to the logo (e.g. `Itaú Unibanco: itau.com.br (logo from the website)`); if it says `not found`, say so. Never guess a URL.
   - If the domain does not obviously belong to the name asked for (a different brand, a parent group, a reseller), flag it.

## Limits

- Logos are trademarks of their owners; the source of the file doesn't change that. Don't present them as the user's own marks.
- Homepage detection is heuristic: the visual check in step 2 is mandatory.
- Wikipedia/Commons may lag a recent rebrand; library brands are a snapshot of the pinned commit. The company's own site or brand page wins when they disagree.
- The white/dark and aspect checks need `rsvg-convert` and PIL; without them every website file is kept as main and must be checked by eye.

## Script: fetch_logo.py

```python
#!/usr/bin/env python3
"""Fetch official brand logos (SVG first) by brand name. Standard library only (+ rsvg-convert/PIL for tone checks). See SKILL.md."""
import argparse, json, os, re, sys, time, unicodedata, urllib.error, urllib.parse, urllib.request, urllib.robotparser

COMMIT = "1ec463a3362461378afabf6bc36abe34e7c01e40"
RAW = f"https://raw.githubusercontent.com/kaankiziltug/logo-design-skill/{COMMIT}/skills/logo-design/assets/library"
CACHE = os.path.expanduser("~/.cache/logo-library")
UA = "LogoFetcherSkill/1.0 (+https://github.com/eilongross/company-logos-skill)"  # honest, descriptive User-Agent (Wikimedia policy too)
# script of the name -> local Wikipedia edition (kana before Han, so Japanese is not read as Chinese)
SCRIPTS = [(r"[\u0590-\u05FF]", "he"), (r"[\u0600-\u06FF]", "ar"), (r"[\u0400-\u04FF]", "ru"), (r"[\u0370-\u03FF]", "el"),
           (r"[\u3040-\u30FF]", "ja"), (r"[\uAC00-\uD7AF]", "ko"), (r"[\u4E00-\u9FFF]", "zh"), (r"[\u0E00-\u0E7F]", "th"),
           (r"[\u0900-\u097F]", "hi")]
# country TLD of a pinned domain -> local Wikipedia edition (only where one language clearly dominates)
TLD_LANG = dict(il="he", de="de", at="de", fr="fr", es="es", mx="es", ar="es", cl="es", pe="es", it="it",
                br="pt", pt="pt", nl="nl", pl="pl", cz="cs", sk="sk", hu="hu", ro="ro", bg="bg", gr="el", tr="tr",
                se="sv", no="no", dk="da", fi="fi", ru="ru", ua="uk", jp="ja", kr="ko", cn="zh", tw="zh", th="th",
                vn="vi", id="id", sa="ar", ae="ar", eg="ar", ma="fr", ir="fa")
BAD = re.compile(r"partner|client|customer|award|badge|certif|app-?store|google-?play|facebook|twitter|linkedin|instagram"
                 r"|youtube|whatsapp|tiktok|payment|visa|mastercard|sponsor|press-logo|as-seen|trustpilot|g2|capterra")


def get(url, binary=False, headers=None, tries=3):
    """GET with retry/backoff on 429/503, honouring Retry-After (capped)."""
    for i in range(tries):
        req = urllib.request.Request(url, headers={"User-Agent": UA, **(headers or {})})
        try:
            with urllib.request.urlopen(req, timeout=30) as r:
                data = r.read()
                return data if binary else data.decode("utf-8", "replace")
        except urllib.error.HTTPError as e:
            if e.code not in (429, 503) or i == tries - 1:
                raise
            wait = min(int(e.headers.get("Retry-After") or 0) or (5 * 3 ** i), 70)
            print(f"  .. throttled ({e.code}), waiting {wait}s", file=sys.stderr)
            time.sleep(wait)


def norm(s):
    """Lower-case letters and digits of any script, accents dropped (Itaú == itau)."""
    return "".join(ch for ch in unicodedata.normalize("NFKD", s.lower()) if ch.isalnum())


def wiki_summary(name, lang, a):
    """First non-disambiguation article for name in a Wikipedia edition (cached)."""
    key = (name, lang)
    if key not in a.summaries:
        a.summaries[key] = None
        base = f"https://{lang}.wikipedia.org"
        res = json.loads(get(f"{base}/w/rest.php/v1/search/title?q={urllib.parse.quote(name)}&limit=5"))
        for page in res.get("pages", []):
            s = json.loads(get(f"{base}/api/rest_v1/page/summary/{urllib.parse.quote(page['key'])}"))
            if s.get("type") != "disambiguation":  # wikipedia-guide gate: a disambiguation page is never the entity
                a.summaries[key] = s
                break
    return a.summaries[key]


def langs_for(name, a):
    """Wikipedia editions to try: --lang, else the local edition (script of the name, else pinned domain's TLD) then en."""
    if a.lang:
        return [l.strip() for l in a.lang.split(",") if l.strip()]
    loc, site = next((lang for pat, lang in SCRIPTS if re.search(pat, name)), None), a.sites.get(name)
    if not loc and site:
        loc = TLD_LANG.get((urllib.parse.urlparse(site if "://" in site else "https://" + site).hostname or "").rsplit(".", 1)[-1])
    return [loc, "en"] if loc and loc != "en" else ["en"]


# ---------- 1. official website ----------
def resolve_site(name, a):
    """Official website as a full URL: the pinned domain, else Wikidata P856. Cached in a.sites."""
    if name not in a.sites:
        a.sites[name] = None
        for lang in langs_for(name, a):
            s = wiki_summary(name, lang, a)
            if s and s.get("wikibase_item"):
                ent = json.loads(get(f"https://www.wikidata.org/wiki/Special:EntityData/{s['wikibase_item']}.json"))
                claims = sorted(list(ent["entities"].values())[0].get("claims", {}).get("P856", []),
                                key=lambda c: c.get("rank") != "preferred")
                a.sites[name] = next((v for c in claims if (v := c["mainsnak"].get("datavalue", {}).get("value"))), None)
                if a.sites[name]:
                    break
    site = a.sites[name]
    return site if not site or site.startswith("http") else "https://" + site


def from_website(name, a):
    home = resolve_site(name, a)
    if not home:
        return []
    rp = urllib.robotparser.RobotFileParser(urllib.parse.urljoin(home, "/robots.txt"))
    try:
        rp.read()
        if not rp.can_fetch(UA, home):
            print(f"  ! website: robots.txt disallows {home}, skipping", file=sys.stderr)
            return []
    except Exception:
        pass
    try:
        with urllib.request.urlopen(urllib.request.Request(home, headers={"User-Agent": UA}), timeout=30) as r:
            html, home = r.read().decode("utf-8", "replace"), r.geturl()  # resolve relative URLs against the final page
    except urllib.error.HTTPError as e:
        print(f"  ! website: {home} answered {e.code} (blocked or missing), not bypassing", file=sys.stderr)
        return []
    if len(html) < 3000 or not re.search(r"<(img|svg)\b", html, re.I):
        print(f"  ! website: {home} returned a bot check / JS-only page, not bypassing", file=sys.stderr)
        return []
    brand = norm(urllib.parse.urlparse(home).hostname.removeprefix("www.").split(".")[0])
    cands, L = [], len(html)
    for m in re.finditer(r"<img\b[^>]*>", html, re.I):
        tag = m.group(0)
        src = re.search(r'\s(?:src|data-src)=["\']([^"\']+)', tag) or re.search(r'srcset=["\']([^"\'\s,]+)', tag)
        if not src or src.group(1).startswith("data:"):
            continue
        ctx = (html[max(0, m.start() - 150):m.start()] + tag).lower()  # own tag + direct parent only
        cands.append(dict(kind="img", url=urllib.parse.urljoin(home, src.group(1)), ctx=ctx, pos=m.start() / L))
    for m in re.finditer(r"<svg\b.*?</svg>", html, re.I | re.S):
        body = m.group(0)
        vb = re.search(r'viewBox=["\'][\d.\-]+[ ,]+[\d.\-]+[ ,]+([\d.]+)[ ,]+([\d.]+)', body[:500])
        if vb and (float(vb[1]) < 40 or float(vb[2]) < 8):  # icon-sized (arrows, chevrons): never a logo
            continue
        if len(body) > 300 and re.search(r"<(path|polygon|rect|circle)\b", body, re.I):
            ctx = (html[max(0, m.start() - 150):m.start()] + body[:body.find(">") + 1]).lower()
            cands.append(dict(kind="inline", data=body, ctx=ctx, pos=m.start() / L))
    scored = []
    for c in cands:
        t = c["ctx"] + c.get("url", "").lower()
        score = 6 * ("logo" in t) + 3 * (brand in norm(t)) + 2 * (c["pos"] < 0.2)
        score += 3 if (c["kind"] == "inline" or ".svg" in c.get("url", "").lower()) else 1
        score -= 8 * bool(BAD.search(t))
        if score >= 9:  # needs "logo" plus brand/svg/header signals
            scored.append((score, -c["pos"], c))
    scored.sort(key=lambda x: x[:2], reverse=True)
    out, seen = [], set()
    for _, _, c in scored[:5]:
        key = c.get("url") or c["data"][:200]
        if key in seen:
            continue
        seen.add(key)
        try:
            data = c["data"].encode() if c["kind"] == "inline" else get(c["url"], binary=True)
        except Exception:
            continue
        if c["kind"] == "inline" and b"xmlns=" not in data[:400]:
            data = data.replace(b"<svg", b'<svg xmlns="http://www.w3.org/2000/svg"', 1)
        is_svg = b"<svg" in data[:2000].lower()
        tone, aspect = ink_tone(data, is_svg)
        named = brand in norm(c["ctx"] + c.get("url", ""))
        ext = ".svg" if is_svg else (os.path.splitext(urllib.parse.urlparse(c.get("url", "")).path)[1] or ".png")
        out.append(dict(source="website", site=home, url=c.get("url", home), data=data, tone=tone, aspect=aspect, named=named,
                        matched=home + (" (inline svg)" if c["kind"] == "inline" else ""), ext=ext))
    primary = next((o for o in out if o["tone"] != "white"), None)
    # reversed extra must be the same brand and the same shape as the primary (not a product logo or app icon)
    rev = next((o for o in out if o["tone"] == "white" and o["named"] and (not primary or not primary["aspect"] or not o["aspect"]
                or 0.65 < o["aspect"] / primary["aspect"] < 1.5)), None)
    keep, res = ("source", "site", "url", "data", "matched"), []
    if primary:
        res.append(dict({k: primary[k] for k in keep}, file=f"{brand}-website{primary['ext']}"))
    if rev:  # official reversed (white) version: keep it for dark slides, never as the main logo
        res.append(dict({k: rev[k] for k in keep}, file=f"{brand}-website-white{rev['ext']}",
                        verify=None if primary else "only a white (reversed) version found on the site; for dark backgrounds"))
    return res


def ink_tone(data, is_svg):
    """(tone, aspect): tone 'white' if most ink is near-white (reversed logo) else 'dark'. Needs rsvg-convert + PIL."""
    try:
        import io, subprocess
        from PIL import Image
        png = subprocess.run(["rsvg-convert", "-h", "200"], input=data, capture_output=True, check=True).stdout if is_svg else data
        im = Image.open(io.BytesIO(png)).convert("RGBA")
        bg = im.getpixel((0, 0))  # opaque baked-in background (e.g. PNG on white) is not ink
        same = (lambda p: bg[3] > 128 and all(abs(p[i] - bg[i]) < 12 for i in range(3)))
        flat = im.get_flattened_data() if hasattr(im, "get_flattened_data") else im.getdata()
        px = [p for p in flat if p[3] > 128 and not same(p)]
        aspect = im.width / im.height
        if not px:
            return "unknown", aspect
        white = sum(1 for r, g, b, _ in px if r > 235 and g > 235 and b > 235)
        return ("white" if white / len(px) > 0.6 else "dark"), aspect
    except Exception:
        return "unknown", None


# ---------- 2. curated library ----------
def catalog():
    os.makedirs(CACHE, exist_ok=True)
    p = os.path.join(CACHE, f"catalog-{COMMIT[:7]}.json")
    if not os.path.exists(p):
        open(p, "w", encoding="utf-8").write(get(f"{RAW}/catalog.json"))
    return json.load(open(p, encoding="utf-8"))


def from_library(name, a):
    cat, n = catalog(), norm(name)
    if not n:
        return []
    exact = [r for r in cat if norm(r["brand"]) == n]
    hits = exact or [r for r in cat if norm(r["brand"]).startswith(n)]
    if not hits:
        return []
    if not exact:  # prefix match: keep only the closest brand
        best = min({r["brand"] for r in hits}, key=len)
        hits = [r for r in hits if r["brand"] == best]
    if a.variant != "all":
        hits = [r for r in hits if r.get("variant") == a.variant] or hits
    return [dict(source="library", url=f"{RAW}/svg/{r['file']}", file=r["file"], matched=r["brand"]) for r in hits]


# ---------- 3. Wikipedia infobox image ----------
def original_from_thumb(url):
    url = "https:" + url if url.startswith("//") else url
    m = re.search(r"//(?:thumb|upload)\.wikimedia\.org/wikipedia/([\w-]+)/thumb/(\w)/(\w\w)/([^/?]+)", url)
    return f"https://upload.wikimedia.org/wikipedia/{m[1]}/{m[2]}/{m[3]}/{m[4]}" if m else url.split("?")[0]


def from_wikipedia(name, a):
    for lang in langs_for(name, a):
        s = wiki_summary(name, lang, a)
        img = s and (s.get("originalimage") or s.get("thumbnail") or {}).get("source")
        if not img:
            continue
        src = original_from_thumb(img)
        fname = urllib.parse.unquote(src.rsplit("/", 1)[1])
        hit = dict(source=f"wikipedia:{lang}", url=src, file=fname,
                   matched=f"{s.get('title')} ({s.get('wikibase_item')}) - {s.get('description', '')}")
        if re.search(r"logo|לוגו|شعار|логотип|λογότυπο|ロゴ|로고|标志|標誌|\.svg$", fname, re.I):
            return [hit]
        # infobox shows a photo (building, person...): last resort only
        a.fallback.setdefault(name, dict(hit, verify="infobox image is probably not a logo"))
    return []


# ---------- 4. Wikidata P154 ----------
def from_wikidata(name, a):
    lang = langs_for(name, a)[0]
    q = ('SELECT ?item ?itemLabel ?logo WHERE { SERVICE wikibase:mwapi { bd:serviceParam wikibase:api "EntitySearch"; '
         'wikibase:endpoint "www.wikidata.org"; mwapi:search "%s"; mwapi:language "%s". '
         '?item wikibase:apiOutputItem mwapi:item. } ?item wdt:P154 ?logo. '
         'SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } } LIMIT 1') % (name.replace('"', ""), lang)
    d = json.loads(get("https://query.wikidata.org/sparql?query=" + urllib.parse.quote(q),
                       headers={"Accept": "application/sparql-results+json"}))
    out = []
    for b in d["results"]["bindings"]:
        fname = urllib.parse.unquote(b["logo"]["value"].rsplit("/", 1)[1]).replace(" ", "_")
        out.append(dict(source="wikidata", url="https://commons.wikimedia.org/wiki/Special:FilePath/" + urllib.parse.quote(fname),
                        file=fname, matched=f'{b["itemLabel"]["value"]} ({b["item"]["value"].rsplit("/", 1)[1]})'))
    return out


# ---------- 5. Iconify ----------
def from_iconify(name, a):
    d = json.loads(get(f"https://api.iconify.design/search?query={urllib.parse.quote(name)}&prefixes=logos,simple-icons&limit=20"))
    n, out = norm(name), []
    for icon in d.get("icons", []):
        prefix, slug = icon.split(":", 1)
        if norm(re.sub(r"-icon$", "", slug)) == n:
            out.append(dict(source=f"iconify:{prefix}", url=f"https://api.iconify.design/{prefix}/{slug}.svg",
                            file=f"{slug}-{prefix}.svg", matched=slug))
    return out


def official_site(name, a):
    """The business's official website as a full URL, for the user to verify the logo against; None if unknown."""
    try:
        return resolve_site(name, a)
    except Exception:
        return None


SOURCES = {"website": from_website, "library": from_library, "wikipedia": from_wikipedia,
           "wikidata": from_wikidata, "iconify": from_iconify}


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("names", nargs="+", help='brand names; "Name=domain" pins the official site')
    ap.add_argument("--out", default="logos")
    ap.add_argument("--variant", default="main", choices=["main", "icon", "all"], help="library: lockup, symbol or both")
    ap.add_argument("--source", default="auto", choices=["auto", *SOURCES])
    ap.add_argument("--lang", help="Wikipedia editions in order, e.g. de,en; default: local edition (from script or domain TLD) then en")
    a = ap.parse_args()
    a.fallback, a.summaries, a.sites = {}, {}, {}
    names = []
    for raw in a.names:
        name, _, site = raw.partition("=")
        names.append(name.strip())
        if site:
            a.sites[name.strip()] = site.strip()
    os.makedirs(a.out, exist_ok=True)
    mpath = os.path.join(a.out, "manifest.json")
    manifest = json.load(open(mpath, encoding="utf-8")) if os.path.exists(mpath) else []
    for name in names:
        found = []
        order = list(SOURCES) if a.source == "auto" else [a.source]
        for i, s in enumerate(order):
            try:
                found = SOURCES[s](name, a)
            except Exception as e:
                print(f"  ! {s} failed for {name}: {e}", file=sys.stderr)
            if found:
                break
        # site gave only a raster: also fetch the best SVG from the next sources as an alternative to compare
        if found and found[0]["source"] == "website" and not found[0]["file"].endswith(".svg") and a.source == "auto":
            for s in order[i + 1:]:
                try:
                    alt = [f for f in SOURCES[s](name, a) if f["file"].lower().endswith(".svg")][:1]
                except Exception:
                    alt = []
                if alt:
                    found += [dict(alt[0], file="alt-" + alt[0]["file"],
                                   verify="SVG alternative: use it only if it matches the site's current logo")]
                    break
        if not found and name in a.fallback:
            found = [a.fallback[name]]
        if not found:
            print(f"MISS  {name}")
            manifest.append({"query": name, "found": False})
            continue
        for f in found:
            path = os.path.join(a.out, re.sub(r'[\\/:*?"<>|]', "_", f["file"]))
            try:
                data = f.pop("data") if "data" in f else get(f["url"], binary=True)
            except Exception as e:
                print(f"FAIL  {name}: {f['url']} ({e})")
                continue
            open(path, "wb").write(data)
            f["format"] = "svg" if b"<svg" in data[:2000].lower() else path.rsplit(".", 1)[-1]
            f.update(query=name, found=True, path=path, site=f.get("site") or official_site(name, a))
            flag = f"  VERIFY: {f['verify']}" if f.get("verify") else ""
            print(f"OK    {name} -> {path}  [{f['source']}, {f['format']}] matched: {f['matched']}"
                  f"  site: {f['site'] or 'not found'}{flag}")
            manifest.append(f)
    json.dump(manifest, open(mpath, "w", encoding="utf-8"), ensure_ascii=False, indent=1)


if __name__ == "__main__":
    main()
```