# 🏡 Náš domov – společný nákupní plán

Tahle verze původní lokální aplikace používá **GitHub Pages + Supabase**. Dva schválení uživatelé sdílí stejný seznam, obrázky jsou v soukromém úložišti, obsah je chráněný přihlášením e-mailem. Aplikace má katalog podle místností, nákupní seznam, odškrtávání, PDF tisk a JSON zálohy.

## 1. Vytvořte repozitář

1. Na https://github.com/new vytvořte repozitář **nas-domov**, vlastník **subrtovajohanka** a nastavte **Public**.
2. Nahrajte z tohoto ZIPu soubory `index.html`, `config.js`, `supabase.sql`, `README.md` do kořene repozitáře.
3. Zatím neaktivujte aplikaci, nejprve založte databázi.

> GitHub Pages veřejně servíruje zdrojový kód, **ne seznam nákupů**. Žádné nákupy, soukromé fotografie nebo autentizační hesla se do repozitáře neukládají.

## 2. Vytvořte Supabase projekt

1. Otevřete https://supabase.com/dashboard → **New project** (Free plán stačí pro malé použití, s omezeními bezplatné varianty).
2. V **SQL Editoru** otevřete obsah `supabase.sql`, **nahraďte na konci** `TVUJ_EMAIL@example.cz` a `VASKUV_EMAIL@example.cz` skutečnými e-maily a spusťte SQL. Upravujte jen řádky s e-maily.
3. V Supabase **Project Settings / API Keys** (nebo **Connect**) najděte **Project URL** a klíč **publishable** (`sb_publishable_...`) nebo starší **anon**. **Nikdy nepoužívejte `service_role` ani secret key.**
4. Otevřete soubor `config.js` a doplňte obě hodnoty. Změnu uložte na GitHub.

`config.js` je veřejný: publishable/anon klíč je k tomu určený. Všechny datové operace chrání Row Level Security (RLS) a privátní Storage. Jen dva e-maily z SQL seznamu se ke společným datům dostanou.

## 3. Přihlášení a URL

1. V Supabase → **Authentication → URL Configuration** nastavte **Site URL** na `https://subrtovajohanka.github.io/nas-domov/`.
2. Stejnou URL vložte do **Redirect URLs** (přesně s koncovým lomítkem).
3. V **Authentication → Providers → Email** ponechte zapnuté email OTP / email přihlášení. Přihlašovat se můžete kódem z e-mailu nebo kliknutím na odkaz. Pokud vám přichází jen odkaz bez kódu, lze jej použít rovnou; případně upravte šablonu e-mailu a vložte proměnnou `{{ .Token }}` pro OTP kód.
4. Pokud e-mail nepřichází, ověřte doručování, spam a případné limity výchozího SMTP v Supabase.

## 4. Zapněte GitHub Pages

V repozitáři `nas-domov` otevřete **Settings → Pages → Build and deployment → Deploy from a branch**, zvolte **main / (root)**, uložte. Adresa stránky bude:

**https://subrtovajohanka.github.io/nas-domov/**

Na prvním zařízení se přihlaste svým e-mailem, potom Vašek svým. Oba musejí být uvedeni v `private.allowed_emails` v SQL. Přidání e-mailu do této tabulky se neprovádí ve veřejném repozitáři.

## 5. Přenesení položek ze staré aplikace

1. Otevřete starý lokální HTML soubor, klikněte **Záloha JSON** a uložte soubor.
2. Otevřete novou online aplikaci, přihlaste se a klikněte **Načíst zálohu** → vyberte JSON.
3. Import **nahradí celý online seznam** (i pro druhého uživatele). Udělejte ho proto jen jednou na začátku. Nahrané obrázky se překopírují do soukromého úložiště.

## Co umí synchronizace

- Změny se ukládají online a přes Supabase Realtime aktualizují i další připojené zařízení.
- Při návratu do otevřené karty se stav znovu načte.
- Pokud dva lidé provedou úpravy doslova současně, aplikace **raději odmítne konfliktní uložení** a znovu načte aktuální stav, místo aby někomu tiše přepsala data.
- Nutné je internetové připojení; aktuální verze nepodporuje offline zápisy.
- PDF export používá systémový tisk: **Uložit jako PDF**; velké místnosti mohou mít více stran A4.
- JSON export obsahuje fotografie, tedy úplnou lokální zálohu.

## Bezpečnost a údržba

- Nezveřejňujte e-maily ani přihlašovací tokeny na GitHubu. V repozitáři má být pouze veřejný Supabase publishable/anon klíč.
- Fotografie jsou ve **Storage bucketu `domov-images` s nastavením private**, aplikace pro zobrazení používá časově omezené podepsané URL.
- SQL RLS pravidla omezuje zobrazení i změny dat na e-maily z povoleného seznamu.
- Supabase Free může po neaktivitě projekt pozastavit; příležitostně se přihlaste a důležité věci zálohujte JSON.
- Při smazání produktu zatím může zůstat jeho starý obrázek v Supabase Storage (ne veřejně přístupný), lze jej časem ručně smazat v dashboardu.
- Zrušit přístup někomu lze smazáním e-mailu z `private.allowed_emails`; existující přihlášení tím ztratí přístup k datům podle RLS.

## Řešení problémů

- **Píše „Nejdřív propojit databázi“:** nebyly vyplněné hodnoty v `config.js`.
- **Přihlášení funguje, ale seznam není vidět:** ověřte e-mail v `private.allowed_emails`, dokončení SQL a že jde o stejný Supabase projekt.
- **Fotky se nenačtou:** ověřte existenci bucketu `domov-images` a storage RLS pravidla.
- **Druhému uživateli se změna neobjeví:** ověřte `supabase_realtime` publikaci v SQL, případně obnovte kartu.
