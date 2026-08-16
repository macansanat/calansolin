# راهنمای پروژه — دستیار دیابت (calansolin)

> این فایل را در ابتدای هر سشن جدید به Claude بده و بگو: **«این را بخوان تا در جریان قرار بگیری»**.
> آخرین نسخه: **۱۰.۴** — مخزن: `macansanat/calansolin` — شاخه‌ی توسعه: `claude/project-guide-v82-7axz7x` — شاخه‌ی اصلی: `main`.

---

## ۱) این پروژه چیست
یک **اپ آموزشی دیابت، تک‌فایلی** (`index.html`) به زبان **فارسی و راست‌به‌چپ (RTL)**.
- کل برنامه در همین یک فایل است: HTML + CSS + JavaScript خالص (بدون فریم‌ورک).
- ذخیره‌سازی داده: `localStorage` (و لایه‌ی `storageGet/storageSet`).
- کتابخانه‌ی بیرونی: فقط **SheetJS/XLSX** از CDN (خط ۱۰) برای ورود و خروجی اکسل.
- صدا: **Web Audio API** (آفلاین، بدون فایل صوتی).
- تقویم: **شمسی (جلالی)**.

## ۲) خط قرمزهای ایمنی (خیلی مهم — هرگز نقض نشود)
- برنامه **فقط آموزشی** است؛ **هرگز** دوز قطعی یا زمان‌بندی تزریق شخصی برای اجرای مستقیم نده.
- فقط فرمول‌های عمومی و تحلیل مرور/آینه‌ای (retrospective) نشان بده و همیشه کاربر را به **پزشک** ارجاع بده.
- داده‌ی نقطه‌ای گلوکومتر را **هرگز** به‌شکل TIR٪ / GMI / AGP استاندارد نشان نده (گمراه‌کننده است)؛ این‌ها فقط با CGM پیوسته معتبرند.

## ۳) قواعد کار و ترجیحات کاربر (باید رعایت شوند)
- **همیشه در چت فارسی بنویس** (کاربر قبلاً از پاسخ‌های انگلیسی ناراضی بود).
- **پاسخ‌ها کوتاه** باشند.
- **هر نسخه‌ی کامل‌شده را به‌صورت پیش‌فرض روی `main` ادغام کن** (مگر خلافش گفته شود). کاربر از ادغام‌نکردن ناراضی می‌شود.
- این فایل راهنما را در ذهن به‌روز نگه دار و در شروع سشن جدید نسخه‌ی کامل به‌روزشده را ارائه بده.
- **پیام‌های کامیت فارسی**؛ پایانشان:
  ```
  Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01LcYDh1TzmV56RC8VHeWqLY
  ```
- **شناسه‌ی مدل** را در هیچ آرتیفکت مخزن (کامیت، PR، کامنت کد) نگذار — فقط در چت.
- جریان کار: توسعه روی شاخه‌ی feature ← تست headless ← کامیت/پوش ← ادغام در `main`.

## ۴) تست headless (Playwright)
مسیر اجرایی Chromium: `/opt/pw-browsers/chromium-1194/chrome-linux/chrome`
الگو: صفحه را با `file://` باز کن، به `pageerror`/`console.error` گوش بده، داده‌ی نمونه را با `localStorage` تزریق کن، `reload` کن، سپس تعامل و اسنپ‌شات بگیر. هدف: **صفر خطای صفحه**.
اسکریپت‌های نمونه در پوشه‌ی scratchpad سشن‌ها بودند (`test104.js`, `shot104.js`, `navtest.js`, ...).

## ۵) نقشه‌ی کد — توابع و ساختارهای کلیدی

### کمکی‌های عمومی
- ارقام: `faDigits` (EN→FA برای نمایش)، `faToEnDigits` (FA/AR→EN، اعشار را نگه می‌دارد — برای **مقادیر**)، `faToEn` (فقط تاریخ، اعشار را حذف می‌کند — برای مقادیر استفاده نکن).
- تقویم شمسی: `g2j`, `j2g`, `jDateStr(jy,jm,jd)`, `parseJDate(s)`, `jDateFieldHTML(inputId, initialStr)` (رشته‌ی HTML یک فیلد تاریخ می‌سازد؛ می‌شود داخل تمپلیت inline صدا زد)، `openJCal(inputId,onPick,dotFn)`، `dayKey(d)` → `yyyy-mm-dd` محلی. `J_DAYS` نام روزهای هفته.

### مدل داده (کلیدهای localStorage)
- `weekly_plan_v1` → `planData`: نگاشت `planData[key] = [meal,...]` که key همان `dayKey`. هر وعده: `{type,time,insType,insDose,carb,bgBefore,bgAfter,peak,trough,foods:[str],id,excludeCalib}`.
- انسولین پایه: `planData._basal[key] = {type, dose, time}`.
- قند دستی: `bg_manual_v1` (`BGKEY`) → `[{time, iso, glu, manual}]`. لودر: `loadBG()`.
- دارو: `med_log_v1` → `[{iso,time,name,dose}]`.
- خواب: `sleep_log_v1` → `[{iso,date,hours,quality}]`.
- حالت روحی: `mood_log_v1` → `[{iso,time,mood,note}]`.
- ورزش: `exercise_log_v1` (`EX_KEY`) → `{entries:[{ts,time,items:[{name}]}]}` لودر `loadExLog()`؛ و `exercise_v1` (`EXKEY`) → `[{iso,what,dur}]` لودر `loadExercise()`.
- آزمایش: `labs_data_v1`؛ CGM: `cgm_data_v1` / `window._cgmReadings`.
- منبع قند: `bg_source` (auto/cgm/meter)؛ حالت ناوبری: `nav_mode`.

### گزارش‌ها و ناوبری
- `renderDashboard()` (خانه)، `renderDailyReport()` (روزانه)، `renderOverview()` (کلی).
- روزانه: `_dailyDate`, `dailyNav(±1)`, `dailyToday()`, `openDailyCal()`, `daySparkSVG(pts)`.
- کلی: `_ovRange` (7/14/30/90/'all')، `setOvRange`, `ovRangeReadings`, `ovComputeStats`, `ovDailyAverages`, `computeAGP`/`agpSVG`, `ovMeterView` (نمای گلوکومتری)، `bgScatterSVG`, `bgStats`.
- منبع قند: `getBgSource`, `resolveBgSrc(hasCgm,hasMeter)` (تصمیم بر اساس داده‌ی همان زمینه)، `effectiveBgSource`, `bgSrcChip`, `setBgSource`.
- ناوبری: تب بالا `.tabs[data-tab]` + نوار پایین `.bottomnav` (`.tab.bnav` + `.bnav-fab`)، `switchTab(name,btn)` هر دو را همگام می‌کند. نوار پایین: خانه / روزانه / [+] / گزارش کلی / پروفایل.
- ثبت سریع: شبکه‌ی `#qaOverlay`، مسیردهی با `quickAdd(type)`.

### فرم‌های ثبت سریع (مودال‌ها با کلاس `.qbg-modal`/`.qbg-box`)
- قند: `openQuickBG`/`saveQuickBG`. انسولین: `openQuickInsulin`/`saveQuickInsulin` با لیست‌های `INS_RAPID` و `INS_BASAL`.
- دارو/خواب/حالت روحی: `openQEntry(...)` + `saveQuickMed/Sleep/Mood`، کمکی‌ها `qDateTimeISO`, `qLogPush`, `uiToast`, `closeQEntry`.

### ماشین‌حساب و ماکروها
- `DB` دیتابیس غذا؛ `SEARCH_INDEX` (`{cat,item,norm}`) با `buildSearchIndex()`؛ `normalizeFa`.
- `MACRO_BY_CAT[cat] = [پروتئین, چربی]` به ازای «واحد پایه»؛ `NUTR_BY_CAT`، `unitFactor(unit)`، `pf(cat,item,unit,amt)`، `calcCarbs()`، `updateMacros(carb,p,fat)` (kcal = carb×4 + p×4 + fat×9).
- **توجه:** وعده‌ها فقط `carb` و نام غذاها (رشته) را ذخیره می‌کنند، **نه** مقدار/واحد؛ پس پروتئین/چربی فقط **تخمینی** قابل بازسازی‌اند.

### ویرایش وعده در تب دفترچه
- `editMealForm(key,m)`، `saveEditMeal(key,id)`، `delMeal(key,id)`، `renderSavedMeals()` (روی `#savedMeals`، روزِ انتخابی = `curWeekStart + selectedDayIdx`).

## ۶) کارهای انجام‌شده در نسخه ۱۰.۴ (آخرین)
1. **ویرایش هر رویداد با کلیک** در گزارش روزانه و تاریخچه.
2. **تاریخچه‌ی کل (Logbook)**: همه‌ی رویدادهای همه‌ی روزها، گروه‌بندی بر اساس تاریخ.
3. **خروجی اکسل** از کل رویدادها.
4. **تفکیک انسولین** روز: کل + پایه (اسم+واحد) + سریع (اسم+واحد).
5. **کالری/پروتئین/چربی تقریبی** در خلاصه‌ی روز.

### توابع افزوده‌شده در ۱۰.۴ (همه در `index.html`)
- تخمین ماکرو: `foodCat(str)` (نام غذا → دسته)، `estMealMacros(m)`.
- کمکی: `jlogGet/jlogSet(k)`، `jFromKey(key)`، `evOnclick(e,from)`.
- ویرایشگر رویداد: `openEventEditor(kind,a,b,from)` / `closeEvEdit()` / `refreshAfterEdit()`.
  - فرم‌ها و ذخیره/حذف برای هر نوع: meal (`evMealForm/saveEvMeal/delEvMeal`)، basal، bg، med، mood، sleep، و ورزش (`evExForm` فقط نمایش+حذف؛ ویرایش کامل در تب ورزش).
- تاریخچه: `collectAllEvents()` (همه‌ی منابع → آرایه)، `renderLogbook()`, `openLogbook()`, `closeLogbook()`.
- خروجی اکسل: `exportEventsXLSX()` (با گارد `typeof XLSX==='undefined'`).
- مودال‌های HTML جدید: `#evEditModal` (بدنه `#evEditBody`، z-index بالاتر تا روی تاریخچه بیاید) و `#logbookModal` (`#logbookBody` + دکمه‌ی `.lb-export`).
- CSS جدید: `.dtl-item.clickable`, `.dtl-edit`, `.insb-*` (کارت تفکیک انسولین)، `.dl-more-btn`, `.ev-*` (فرم ویرایش)، `.lb-*` (تاریخچه).

## ۷) تاریخچه‌ی خلاصه‌ی نسخه‌ها (تا ۱۰.۴)
- بازطراحی تب آزمایش (کشویی، رنج‌های سفارشی، بسته‌های پیشنهادی، کادر خارج‌ازمحدوده).
- بازطراحی هدر و تب‌ها؛ نوار پایین + دکمه‌ی + و ثبت سریع.
- داشبورد خانه؛ قند دستی با UI درست و تاریخ شمسی.
- دارو/خواب/حالت روحی؛ انتقال «حساب دوز/دفترچه/شروع ورزش» به دسترسی سریع.
- گزارش روزانه و کلی با نمودارها (اسپارک‌لاین، روند ساعتی، AGP، نوار TIR، انتخاب بازه‌ی روز، تقویم شمسی با نقطه‌ی داده).
- کلید سراسری منبع قند (CGM/گلوکومتر) + نمای گلوکومتری + رفع باگ حالت خودکار (CGM قدیمی روی گلوکومتر تازه سایه می‌انداخت).
- کتابخانه‌ی آهنگ ورزش (۱۲ ترک)، بلندی صدا + resume بعد از قفل صفحه (Wake Lock + شنونده‌ها).
- ۱۰.۳: ثبت سریع انسولین (سریع/پایه) از دکمه‌ی +.
- **۱۰.۴: ویرایش رویدادها + تاریخچه‌ی کل + خروجی اکسل + تفکیک انسولین + ماکروها.**

## ۸) محدودیت‌های صادقانه (به کاربر گفته شده)
- پخش پیوسته‌ی صدا هنگام قفل کامل گوشی و تایمر روی صفحه‌ی قفل در وب ممکن نیست (به‌خصوص iOS)؛ Wake Lock + resume بهترین حالت ممکن وب است.
- خروجی اکسل به CDN وابسته است؛ آفلاین کار نمی‌کند و پیام مناسب می‌دهد.
- ماکروها (پروتئین/چربی/کالری) تخمینی‌اند (با علامت ~) چون مقدار/واحد هر غذا ذخیره نمی‌شود.
