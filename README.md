# UTAS Student Business Platform — منصة مشاريع الطلاب

واجهة (HTML/CSS/JavaScript) مربوطة مع باك إند PHP + MySQL (PDO). كل الصفحات تتواصل مع الـ API عبر `fetch`.

## 1) التشغيل على XAMPP
1. انسخ المجلد كامل إلى `C:\xampp\htdocs\` (أي اسم للمجلد يعمل، الروابط نسبية).
2. شغّل **Apache** و **MySQL** من XAMPP Control Panel.
3. في `http://localhost/phpmyadmin`: إذا كانت قاعدة `utas_business` موجودة من نسخة قديمة فاحذفها (Drop)، ثم Import ← `database/schema.sql`.
4. راجع `config/config.php` (بيانات القاعدة، `allowed_email_domain`).
5. أنشئ حساب الأدمن من سطر الأوامر داخل المجلد:
   ```
   C:\xampp\php\php.exe database\create_admin.php "Admin Name" admin@utas.edu.om "كلمة_سر_قوية"
   ```
6. افتح الموقع من المتصفح: `http://localhost/<اسم-المجلد>/` (لا تفتح ملفات HTML بالنقر المزدوج، لأن الجلسة والـ API يحتاجان سيرفر).
7. اختبار سريع للـ API: `http://localhost/<اسم-المجلد>/api/categories.php` يجب أن يطلع JSON بالتصنيفات.

> رفع الصور: ملف `.htaccess` في الجذر يرفع حدود PHP (`upload_max_filesize=6M`). إذا ما اشتغل عندك (حد 2MB الافتراضي)،
> عدّل `C:\xampp\php\php.ini`: `upload_max_filesize = 6M` و `post_max_size = 32M` ثم أعد تشغيل Apache.

## 2) هيكل المجلد
```
index.html          الرئيسية: بحث وفلترة وعرض المنتجات
login.html / register.html
project.html?id=    تفاصيل منتج (صور + بيانات التواصل)
dashboard.html      لوحة صاحب المشروع (مشاريعي + منتجاتي)
edit_business.html  إضافة / تعديل مشروع (?id= للتعديل)
add_project.html    إضافة / تعديل منتج (?id= للتعديل)
admin.html          لوحة الأدمن
assets/css/style.css
assets/js/          common.js (مشترك) + ملف JS لكل صفحة
api/  config/  includes/  database/  uploads/
```

## 3) الأدوار والتدفق
- **طالب (student):** يتصفح فقط، ويرى بيانات التواصل بعد تسجيل الدخول.
- **صاحب مشروع (owner):** ينشئ مشروعاً (`edit_business.html`) ثم يضيف منتجات بصور (`add_project.html`).
- **أدمن (admin):** يرى كل شيء، يخفي/يُظهر أو يحذف المنتجات والمشاريع، ويحذف المستخدمين.
- المحتوى الجديد يظهر مباشرة (لا يوجد نظام موافقة مسبقة). الأدمن يقدر **يخفي** أي عنصر (حالة `removed`) وإرجاعه لاحقاً.
- الصفحات المحمية تحوّل الزائر تلقائياً إلى `login.html`، وبعد الدخول يرجع للصفحة المطلوبة.

## 4) تغييرات الباك إند مقارنة بالنسخة السابقة
| التغيير | السبب |
|---|---|
| التصنيفات في القاعدة صارت عربية وبنفس تصنيفات الواجهة (6 تصنيفات) وتُجلب من `categories.php` | الواجهة كانت تستخدم قيماً نصية ثابتة |
| `contact_info` ← `contact_email` + `contact_phone` في جدول `businesses` | الواجهة فيها بريد وهاتف منفصلان |
| السعر يقبل رقماً أو نطاقاً (`3` أو `2 - 5`) ويُخزَّن في `price` + `price_max`، ويقبل أرقاماً عربية | الواجهة تعرض نطاقات أسعار |
| `listings/mine.php` (جديد) | لوحة المالك تحتاج كل منتجاته بما فيها المخفية |
| `listings/get.php` يرجع `owner_name` وحالة المشروع | صفحة التفاصيل |
| تعديل المنتج: الحقول غير المرسلة تبقى كما هي (قبل: كانت تُمسح) | إصلاح خطأ |
| روابط الصور نسبية (`base_url` فارغ) | تعمل مهما كان اسم المجلد |
| كل رسائل الأخطاء بالعربي، وخطأ واضح عند تجاوز حجم الرفع | الواجهة عربية |
| كوكي الجلسة `secure` تلقائياً على HTTPS | أمان |

## 5) قائمة الـ Endpoints
كل الاستجابات: `{ "ok": true, ... }` أو `{ "ok": false, "error": "...", "fields": { "email": "..." } }`.
الأكواد: 200/201 نجاح، 401 لازم تسجل دخول، 403 ممنوع، 404 غير موجود، 405 طريقة خاطئة، 409 مكرر، 413 ملف كبير، 422 بيانات غير صحيحة.

| الطريقة | المسار | الجسم / المعاملات | الصلاحية |
|---|---|---|---|
| POST | `api/auth/register.php` | `full_name, email, password, role` (`student`/`owner`) | الكل |
| POST | `api/auth/login.php` | `email, password` | الكل |
| POST | `api/auth/logout.php` | — | الكل |
| GET | `api/auth/me.php` | — | الكل |
| GET | `api/categories.php` | — | الكل |
| GET | `api/listings/search.php` | `q, category_id, business_id, min_price, max_price, sort, page, per_page` | الكل |
| GET | `api/listings/get.php` | `id` | الكل |
| GET | `api/businesses/list.php` | `q, page` | الكل |
| GET | `api/businesses/get.php` | `id` | الكل |
| GET | `api/businesses/mine.php` | — | owner |
| POST | `api/businesses/save.php` | `business_name, description, contact_email, contact_phone` (+`business_id` للتعديل) | owner |
| POST | `api/businesses/delete.php` | `business_id` | owner/admin |
| GET | `api/listings/mine.php` | — | owner |
| POST | `api/listings/save.php` | multipart: `business_id, category_id, title, description, price, images[]` (+`listing_id` للتعديل) | owner |
| POST | `api/listings/delete.php` | `listing_id` | owner/admin |
| POST | `api/listings/delete_image.php` | `image_id` | owner/admin |
| GET | `api/admin/stats.php` | — | admin |
| GET | `api/admin/users.php` | `q, role, page` | admin |
| POST | `api/admin/delete_user.php` | `user_id` | admin |
| GET | `api/admin/content.php` | `type` (`businesses`/`listings`), `status, q, page` | admin |
| POST | `api/admin/set_status.php` | `type` (`business`/`listing`), `id`, `status` (`active`/`removed`) | admin |

## 6) خطوات اختبار مقترحة
1. سجّل حساب **صاحب مشروع** ← ينقلك لإنشاء مشروع ← احفظه.
2. أضف منتجاً بصورتين (جرّب السعر `3` ثم `2 - 5` ثم فارغاً) وتأكد من ظهوره في الرئيسية.
3. افتح المنتج وتأكد من الصور والتواصل، ثم سجّل خروج وافتحه كزائر (التواصل مقفول).
4. سجّل حساب **طالب** وجرّب البحث والتصنيفات.
5. ادخل بحساب الأدمن: أخفِ المنتج وتأكد أنه اختفى من الرئيسية، ثم أظهره.

## 7) ملاحظات وحدود معروفة
- `allowed_email_domain` في `config.php` هو `utas.edu.om`؛ تأكد أنه نفس دومين إيميلات الطلاب عندكم.
- التسجيل يتحقق من **شكل** الدومين فقط (لا يوجد رمز تأكيد بالإيميل)، ولا يوجد حد لمحاولات الدخول الخاطئة.
- `debug` في `config.php` مفعّل للتطوير ويعرض رسالة الخطأ الحقيقية؛ **اجعله `false` قبل التسليم**.
- عند الرفع على HTTPS يُفعَّل `secure` للكوكي تلقائياً.
