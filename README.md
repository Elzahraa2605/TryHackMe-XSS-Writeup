<div dir="rtl" style="text-align: right; width: 100%; word-wrap: break-word; overflow-wrap: break-word; line-height: 2; letter-spacing: 0.2px;">

<p align="center">
  <img src="XSS-screens/step1.png" width="500">
</p>

# Room Information

- **Room Name**: XSS
- **Platform**: TryHackMe
- **Topics Covered**: XSS Root Causes, Reflected XSS, Stored XSS, DOM-based XSS, Context-Aware Payload Crafting, Filter Evasion, Real-World CVE Exploitation (CVE-2023-38501, CVE-2021-38757).

---

# Room Overview

الغرفة دي مش بتركز بس على استغلال XSS، دي بتشرح **الأسباب الجذرية** وراء الثغرة نفسها: إزاي بتحصل جوه الكود بلغات مختلفة (PHP, Node.js, Python/Flask, C#/ASP.NET)، وإزاي بتتصلح، وبعدين بتطبق النظرية دي على تطبيقين حقيقيين فيهم ثغرات موثقة رسمياً (CVE).

الغرفة مقسمة لـ 3 أنواع أساسية: Reflected XSS، Stored XSS، وDOM-based XSS، بالإضافة لجزء عن الـ Context اللي الـ payload بيتنفذ فيه وطرق الـ Evasion.

---

# Task 1-2: Introduction & JavaScript Basics

<p align="center">
  <img src="XSS-screens/step2.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step3.png" width="600">
</p>

* **Prerequisites**: معرفة بسيطة بـ HTTP وإزاي المواقع بتشتغل، وغرفة `Intro to Cross-site Scripting` كأساس.
* **Core Concept**: XSS بتستغل ثقة المستخدم في موقع موثوق عشان تنفذ script خبيث جوه الـ browser بتاعه، وبالتالي بتتخطى الـ Same-Origin Policy (SOP).

<p align="center">
  <img src="XSS-screens/step4.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step5.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step6.png" width="600">
</p>

* **JavaScript Console**: تم استخدام الـ Console (Ctrl+Shift+K على Firefox / Ctrl+Shift+J على Chrome) لتجربة الدوال الأساسية:
  - `alert()` لعرض popup.
  - `console.log()` لطباعة قيمة في الـ console.
  - `btoa()` / `atob()` للتحويل من وإلى base64.

<p align="center">
  <img src="XSS-screens/step7.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step8.png" width="600">
</p>

* **XSS Types Recap**:
  | Type | Description |
  | --- | --- |
  | Stored XSS | الـ payload بيتخزن في الداتابيز ويتعرض لاحقاً |
  | Reflected XSS | الـ payload بينفذ فوراً من غير ما يتخزن |
  | DOM-based XSS | بتحصل بالكامل client-side بدون رجوع للسيرفر |

<p align="center">
  <img src="XSS-screens/step9.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step10.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step11.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step12.png" width="600">
</p>

* **Practical Testing**: تجربة `alert(1)`, `alert("XSS")`, `console.log()`, و`btoa()/atob()` مباشرة على bWAPP لتأكيد سلوك الـ Console.

---

# Task 3: Causes and Implications

<p align="center">
  <img src="XSS-screens/step13.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step14.png" width="600">
</p>

## Root Causes

* **Insufficient input validation and sanitization**: الـ input بيتاخد من المستخدم ويتحط في الـ HTML مباشرة من غير تحقق.
* **Lack of output encoding**: عدم عمل encode للحروف `< > " ' &` بيسمح بحقن script.
* **Improper Security Headers**: misconfiguration في CSP (استخدام `unsafe-inline` / `unsafe-eval`) بيسهّل الاستغلال.
* **Framework/Language Vulnerabilities**: frameworks قديمة من غير حماية مدمجة.
* **Third-party Libraries**: مكتبة خارجية واحدة معرضة ممكن تعرّض التطبيق كله.

<p align="center">
  <img src="XSS-screens/step15.png" width="600">
</p>

## Implications

| Impact | Description |
| --- | --- |
| Session Hijacking | سرقة الكوكيز والاستيلاء على جلسة المستخدم |
| Phishing / Credential Theft | نوافذ تسجيل دخول وهمية داخل الموقع الموثوق |
| Social Engineering | popups شرعية الشكل لخداع المستخدم |
| Content Manipulation / Defacement | تغيير محتوى الموقع |
| Data Exfiltration | تسريب بيانات حساسة من متصفح الضحية |
| Malware Installation | drive-by download attacks |

* **Mitigation Summary**: `validation and sanitization` على الـ input، و`encoding` على الـ output.

---

# Task 4: Reflected XSS — Code Analysis (Multi-language)

<p align="center">
  <img src="XSS-screens/step16.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step17.png" width="600">
</p>

* **Vulnerability Pattern**: قيمة query string بتترجع (reflected) في الصفحة زي ما هي من غير sanitization، زي `<script>alert(document.cookie)</script>` اللي المفروض يتحول لـ `&lt;script&gt;...`.

## PHP

<p align="center">
  <img src="XSS-screens/step17.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step18.png" width="600">
</p>

* **Root Cause**: `$_GET['q']` بتتعرض مباشرة في الـ response.
* **Fix**: استخدام `htmlspecialchars()` لتحويل `< > & " '` لـ HTML entities.

## JavaScript (Node.js / Express)

<p align="center">
  <img src="XSS-screens/step18.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step19.png" width="600">
</p>

* **Root Cause**: `req.query.q` بتضاف للـ response من غير escaping.
* **Fix**: مكتبة `sanitize-html` ودالة `sanitizeHtml()`، أو بديلاً `escapeHtml()`.

## Python (Flask)

<p align="center">
  <img src="XSS-screens/step20.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step21.png" width="600">
</p>

* **Root Cause**: `request.args.get("q")` بتتحط في f-string من غير حماية.
* **Fix**: دالة `escape()` (alias لـ `markupsafe.escape()`).

## C# (ASP.NET)

<p align="center">
  <img src="XSS-screens/step21.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step22.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step23.png" width="600">
</p>

* **Root Cause**: `Request.QueryString["q"]` بتتكتب في الـ `Response` مباشرة.
* **Fix**: دالة `HttpUtility.HtmlEncode()`.

---

# Phase: Practical Exploitation — copyparty (CVE-2023-38501)

<p align="center">
  <img src="XSS-screens/step24.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step25.png" width="600">
</p>

## Execution Parameters
```
Target: http://10.128.146.106:3923
Payload (raw): <img src=copyparty onerror=alert(1)>
Payload (URL-encoded): ?k304=y%0D%0A%0D%0A%3Cimg+src%3Dcopyparty+onerror%3Dalert(1)%3E
```

## Evidence & Outputs

<p align="center">
  <img src="XSS-screens/step26.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step27.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step28.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step29.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step30.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step31.png" width="600">
</p>

## Technical Analysis

* **Findings**: السيرفر شغال نسخة قديمة من `copyparty` على بورت 3923، وفيها ثغرة Reflected XSS مسجلة رسمياً (CVE-2023-38501). حقن الـ payload مباشرة في الرابط أدى لتنفيذ الـ JavaScript عن طريق event `onerror` بتاع الـ `<img>` tag اللي مصدره غير موجود.
* **Impact**: نجح تنفيذ `alert(1)` فعلياً، وظهرت بعد كده صفحة الـ admin panel الخاصة بـ copyparty (`/?h#cc`) وفيها خيارات إدارية (scanning, hash-q, tag-q, mtp-q, db-act, enable k304).
* **Assessment & Conclusions**: الثغرة دي مثال واقعي على إن حتى الأدوات المستخدمة بشكل واسع ممكن يفوتها sanitization بسيط، وده أدى لثغرة موثقة بـ CVE رسمي.

## Next Logical Step

الانتقال لدراسة Stored XSS نظرياً، ثم تطبيقها على مشروع حقيقي تاني (Hospital Management System).

---

# Task 6: Stored XSS — Code Analysis (Multi-language)

<p align="center">
  <img src="XSS-screens/step32.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step33.png" width="600">
</p>

* **Vulnerability Pattern**: الـ input بيتخزن في الداتابيز (comments, reviews, forum posts) وبيتعرض لاحقاً لمستخدمين تانيين من غير sanitization.

## Mitigation Best Practices

* Validate and sanitize input (whitelist rules صارمة).
* Output escaping لكل الحروف الخاصة بـ HTML.
* Context-specific encoding (HTML / JavaScript / URL).
* Defence in depth — server-side validation مش client-side بس.

## PHP

<p align="center">
  <img src="XSS-screens/step33.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step34.png" width="600">
</p>

* **Root Cause**: الكومنت بيتخزن في جدول `comments` ويترجع للعرض من غير sanitization.
* **Fix**: `htmlspecialchars()` عند عرض كل comment.

## JavaScript (Node.js)

<p align="center">
  <img src="XSS-screens/step34.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step35.png" width="600">
</p>

* **Root Cause**: array الـ `comments` بيتحط جوه الـ HTML مباشرة.
* **Fix**: `sanitizeHtml()` — بتشيل عناصر زي `<script>` و`onload` مع السماح بـ tags آمنة زي `<b>` و`<i>`.

## Python (Flask + SQLAlchemy)

<p align="center">
  <img src="XSS-screens/step36.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step37.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step38.png" width="600">
</p>

* **Root Cause**: `request.form['comment']` بيتخزن ويتعرض من غير أي معالجة.
* **Fix**: `escape()` من `markupsafe` عند عرض كل comment (الـ storage نفسه بيفضل زي ما هو، العرض بس هو اللي بيتحمي).

## C# (ASP.NET)

<p align="center">
  <img src="XSS-screens/step38.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step39.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step40.png" width="600">
</p>

* **Root Cause**: `SaveComment` و`DisplayComments` بيستخدموا string concatenation مباشرة (Stored XSS + SQL Injection).
* **Fix**: `HttpUtility.HtmlEncode()` للعرض + `Parameters.AddWithValue()` للاستعلامات (Parametrized Queries).

<p align="center">
  <img src="XSS-screens/step41.png" width="600">
</p>

---

# Phase: Practical Exploitation — Hospital Management System (CVE-2021-38757)

<p align="center">
  <img src="XSS-screens/step42.png" width="600">
</p>

## Execution Parameters
```
Target: http://10.128.181.106
Vulnerable Endpoint: /contact.php (Contact form → txtMsg parameter)
Payload: <script>alert(document.cookie)</script>
Receptionist Credentials: admin / admin123
```

## Evidence & Outputs

<p align="center">
  <img src="XSS-screens/step51.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step52.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step53.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step54.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step55.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step43.png" width="600">
</p>

## Technical Analysis

* **Findings**: المشروع (Hospital Management System) مفتوح المصدر ومحدثش من سنين، وفيه ثغرة Stored XSS موثقة رسمياً (CVE-2021-38757) في صفحة Contact. الكود بتاع `contact.php` بياخد `$_POST['txtMsg']` ويخزنه في الداتابيز مباشرة من غير أي sanitization.
* **Impact**: بعد إرسال الـ payload من صفحة Contact، وتسجيل الدخول كـ Receptionist، الـ script اتنفذ فوراً في لوحة التحكم وظهر alert box فيه قيمة كوكي الجلسة (`PHPSESSID`).
* **Assessment & Conclusions**: الثغرة دي بتوضح إزاي أي حقل input عام (زي Contact form) ممكن يستخدم كنقطة دخول لسرقة جلسات المستخدمين المصرح لهم (Receptionist)، حتى لو المهاجم نفسه مش مسجل دخول.

## Next Logical Step

الانتقال لدراسة النوع الثالث (DOM-based XSS) اللي بيحصل بالكامل client-side بدون تفاعل مع الـ backend.

---

# Task 8: DOM-based XSS

<p align="center">
  <img src="XSS-screens/step44.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step45.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step46.png" width="600">
</p>

* **Core Concept**: الـ DOM (Document Object Model) هو representation شجري لمستند الـ HTML، وبيتلاعب بيه عن طريق JavaScript (`document.createElement()`, `element.append()`). DOM-based XSS بتحصل بالكامل client-side من غير أي رحلة للسيرفر، وده اللي بيخليها أقل انتشاراً حالياً مقارنة بـ Reflected وStored.

## Vulnerable "Static" Site

<p align="center">
  <img src="XSS-screens/step46.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step47.png" width="600">
</p>

* **Root Cause**: قيمة `?name=` من الرابط بتتعرض مباشرة عن طريق `document.write()` من غير أي حماية.
* **Exploitation**: إدخال `?name=Web Tester<script>alert("XSS")</script>` أدى لتنفيذ الـ script فعلياً، وظهور عنصر جديد جوه شجرة الـ DOM (الـ `<body>` بقى عنده 4 children بدل 3).

<p align="center">
  <img src="XSS-screens/step48.png" width="600">
</p>

## Fixed "Static" Site

<p align="center">
  <img src="XSS-screens/step48.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step49.png" width="600">
</p>

* **Fix**: استبدال `document.write()` بـ `encodeURIComponent()` مع `textContent`. النتيجة إن أي محاولة حقن بتتعرض كـ نص مشفر (encoded) مش كـ كود ينفذ فعلياً، وشجرة الـ DOM مبتتأثرش.

<p align="center">
  <img src="XSS-screens/step50.png" width="600">
</p>

* **Key Takeaways**:
  - DOM-based XSS مش بترجع من السيرفر (Nay).
  - بتحصل client-side بس (Yea).
  - الدالة المستخدمة للحماية: `encodeURIComponent()`.

---

# Task 9: Context and Evasion

<p align="center">
  <img src="XSS-screens/step56.png" width="600">
</p>
<p align="center">
  <img src="XSS-screens/step57.png" width="600">
</p>

## Context

الـ payload ممكن يتنفذ في واحدة من 3 حالات:
* **بين HTML tags**: `<script>alert(document.cookie)</script>` مباشرة.
* **جوه HTML tag**: لازم قفل الـ tag الأول، مثلاً `"><script>alert(document.cookie)</script>`.
* **جوه JavaScript موجودة**: إنهاء الـ script الحالي بـ `</script>`، أو قفل الـ string بـ `'` وتنفيذ أمر جديد وعمل comment على الباقي بـ `//`.

## Evasion Techniques

| Character | Hex Representation |
| --- | --- |
| Horizontal Tab (TAB) | `09` |
| New Line (LF) | `0A` |
| Carriage Return (CR) | `0D` |

* استخدام الحروف دي بيسمح بتكسير الـ payload والتحايل على blocklist filters، بناءً على **XSS Filter Evasion Cheat Sheet**. مصادر مفيدة لبناء payloads: **XSS Payload List** و**Tiny XSS Payloads** (لو فيه قيود على طول الـ payload).

---

# Task 10: Conclusion

<p align="center">
  <img src="XSS-screens/step58.png" width="600">
</p>

* الغرفة غطت الأسباب الجذرية لـ XSS، الحلول بأربع لغات برمجة مختلفة لكل نوع (Reflected / Stored)، شرح DOM-based XSS النظري والعملي، وتطبيقين عمليين على ثغرات موثقة رسمياً (CVE-2023-38501 في copyparty، وCVE-2021-38757 في Hospital Management System).

---

# Vulnerabilities Summary

| # | Vulnerability Type | Target | CVE / Reference | Result |
| --- | --- | --- | --- | --- |
| 1 | Reflected XSS | copyparty (port 3923) | CVE-2023-38501 | `alert(1)` executed via `onerror` |
| 2 | Stored XSS | Hospital Management System (`contact.php`) | CVE-2021-38757 | Session cookie (`PHPSESSID`) disclosed to Receptionist |

</div>
