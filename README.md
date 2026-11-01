# Watch Face Uploader

A single-page web tool that uploads a watch face `.bin` file to a Bluetooth smartwatch straight from the browser, using Web Bluetooth. It is meant for iPhone (through the **Bluefy** browser) and for Android (Chrome), when you want to send a face without the vendor app.

> **Status: experimental, confirmed on one watch.** The upload protocol in this page was written from observed behavior, not from an official specification. It has been reported working on a **WearFit W03 Pro**. Other watches may behave differently, so test with a small file first and use it at your own risk. This project is not affiliated with any watch maker or app.

---

## What it does

- Lets you pick a watch face `.bin` file from your phone.
- Connects to the watch over Bluetooth Low Energy (Nordic UART service).
- Sends the file in 1 KB blocks, each with a CRC-16, waits for the watch to confirm every block, and retries when it does not answer.
- Shows progress and a short log (the first replies from the watch are printed, which helps when something fails).

## Requirements

| Device | Browser |
|---|---|
| iPhone / iPad | **Bluefy** (Safari and Chrome on iOS have no Web Bluetooth) |
| Android | Chrome |
| Computer | Chrome or Edge |

- The page must be served over **HTTPS** (GitHub Pages is fine). Web Bluetooth does not work from a local file.
- Open the page directly in the browser, not inside another app's built-in viewer or a frame.
- Disconnect the watch from its normal phone app (for example Chronos) first. The watch usually accepts only one connection.

## How to use

1. Open the page in Bluefy (iPhone) or Chrome (Android).
2. Check the first line of the log: `Bluetooth: available`.
3. Choose the `.bin` watch face file.
4. Pick **Bytes per frame**. Start with `120` or `168` on iPhone. If sending fails, lower it.
5. Press **Connect and upload** and choose your watch in the list.
6. Wait for the progress bar to reach 100 %. The watch should apply the face.

If the watch freezes, hold its side button for 10 to 15 seconds to force a restart.

## Troubleshooting

| Problem | What to try |
|---|---|
| `Bluetooth: NOT available` | You are not in Bluefy or Chrome, or the page is inside a frame. Open the link directly. |
| The watch is not in the list | Disconnect it from the phone app first, and keep it close and awake. |
| `No ack for block N` repeats, then an error | Lower **Bytes per frame** (try 120). Move closer to the watch. |
| `the watch keeps asking to resend block N` | Same as above. The file may also be damaged. |
| Uploads reach 100 % but nothing changes | The file may not be accepted by the watch. Try a different known-good `.bin`. |
| The watch froze after the upload | Hold the side button 10 to 15 seconds to restart. Do not upload that file again. |

## Protocol summary (as implemented in this page)

This is what the code sends. It is documented here for people who want to check or improve it, not as an official specification.

- **Service** `6e400001-b5a3-f393-e0a9-e50e24dcca9e`, **write** characteristic `6e400002-…`, **notify** characteristic `6e400003-…`.
- **Start** command `ab 00 04 ff 87 80 01` (sent twice), **stop** command `ab 00 04 ff 87 80 00`.
- The file is cut into blocks of 1024 bytes. Each block starts with a 12-byte header:
  `b0`, data length (2 bytes), block index (2), CRC-16 (2), last-block flag (1), `00`, total file length (3 bytes).
- The block data follows in frames: `af`, frame length (2 bytes, payload + 5), `00`, sequence number, payload (up to *Bytes per frame*).
- CRC-16: polynomial `0xA001` (reflected), initial value `0xFFFF`.
- The watch answers each block with a notification `b0 02 <next block (2)> <done flag>`. The page sends the next block when it receives it, and resends the current block when there is no answer within 5 seconds (up to 5 tries).

## Tested on

| Watch | Phone | Browser | Result |
|---|---|---|---|
| WearFit W03 Pro | iPhone | Bluefy | Upload worked (reported by the author) |

If you try another watch, please report the model, phone, iOS or Android version, browser and result (an issue or a pull request that adds a row is welcome).

---

# أداة رفع واجهات الساعة

صفحة ويب واحدة ترفع ملف واجهة الساعة `.bin` إلى ساعة ذكية عبر البلوتوث مباشرة من المتصفح (Web Bluetooth). مصممة للآيفون عبر متصفح **Bluefy**، وللأندرويد عبر Chrome، لمن يريد إرسال واجهة بدون تطبيق الشركة.

> **الحالة: تجريبية، ومؤكدة على ساعة وحدة.** بروتوكول الرفع في هذه الصفحة مبني على ملاحظات عملية، مو على مواصفات رسمية. اشتغلت على ساعة **WearFit W03 Pro** حسب صاحب المشروع. ساعات ثانية ممكن تتصرف بشكل مختلف، فجرّب بملف صغير أولاً، والاستخدام على مسؤوليتك. المشروع غير تابع لأي شركة ساعات أو تطبيق.

## شنو تسوي

- تختار ملف الواجهة `.bin` من الموبايل.
- تتصل بالساعة عبر البلوتوث منخفض الطاقة (خدمة Nordic UART).
- ترسل الملف كتل بحجم 1 كيلوبايت، لكل كتلة CRC-16، وتنتظر تأكيد الساعة لكل كتلة، وتعيد المحاولة إذا ما ردّت.
- تعرض شريط تقدّم وسجلاً قصيراً (تطبع أول ردود الساعة، وهذا يساعد عند الفشل).

## المتطلبات

- **آيفون:** متصفح **Bluefy** (Safari وChrome على آيفون ما يدعمان Web Bluetooth).
- **أندرويد:** Chrome.
- لازم الصفحة تكون على رابط **HTTPS** (GitHub Pages يكفي). ما تشتغل من ملف محلي.
- افتحها مباشرة بالمتصفح، مو داخل عارض تطبيق آخر أو إطار.
- افصل الساعة من تطبيقها الأساسي (مثل Chronos) قبل الرفع، لأن الساعة غالباً تقبل اتصالاً واحداً.

## طريقة الاستخدام

1. افتح الصفحة بـ Bluefy (آيفون) أو Chrome (أندرويد).
2. تأكد إن أول سطر بالسجل `Bluetooth: available`.
3. اختر ملف `.bin`.
4. اختر **Bytes per frame**: ابدأ بـ `120` أو `168` على الآيفون، وقلّلها إذا فشل الإرسال.
5. اضغط **Connect and upload** واختر ساعتك من القائمة.
6. انتظر لين الشريط يوصل 100%. المفروض الساعة تطبّق الواجهة.

إذا الساعة علّقت، اضغط الزر الجانبي 10 إلى 15 ثانية حتى تعيد التشغيل.
b.io/اسم-المستودع/`. افتح هذا الرابط بـ Bluefy.

## حل المشاكل

- **`Bluetooth: NOT available`:** مو بـ Bluefy أو Chrome، أو الصفحة داخل إطار. افتح الرابط مباشرة.
- **الساعة ما تظهر بالقائمة:** افصلها من التطبيق الأساسي، وقرّبها وخلّها مفتوحة.
- **`No ack for block` تتكرر ثم خطأ:** قلّل **Bytes per frame** (جرّب 120) وقرّب من الساعة.
- **توصل 100% وما يتغير شي:** ممكن الساعة ما قبلت الملف. جرّب ملف `.bin` معروف إنه شغّال.
- **الساعة علّقت بعد الرفع:** اضغط الزر الجانبي 10 إلى 15 ثانية، ولا ترفع نفس الملف مرة ثانية.

## جرّبتها على

- **WearFit W03 Pro** مع **آيفون** و**Bluefy**: الرفع اشتغل (حسب صاحب المشروع).

إذا جرّبت ساعة ثانية، اكتب الموديل والموبايل وإصدار النظام والمتصفح والنتيجة (مرحّب بإضافة صف عن طريق issue أو pull request).

## License

Source-available, **all rights reserved** (see [LICENSE](LICENSE)). In short: personal, non-commercial use only, and every change must stay inside this GitHub repository. Do not republish the code or modified copies anywhere else. Pull requests to this repository are welcome. Use at your own risk: a wrong file can freeze a watch.

## الرخصة

الكود متاح للقراءة، **وكل الحقوق محفوظة** (شوف ملف [LICENSE](LICENSE)). باختصار: الاستخدام شخصي وغير تجاري فقط، وكل التعديلات لازم تبقى داخل مستودع GitHub هذا، وممنوع نشر الكود أو أي نسخة معدّلة منه بمكان ثاني. الـ pull requests على هذا المستودع مرحّب بيها. الاستخدام على مسؤوليتك: ملف غلط ممكن يعلّق الساعة.
