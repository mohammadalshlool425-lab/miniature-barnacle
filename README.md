# شمعة - منصة البيع والشراء

## التشغيل

شغّل PowerShell داخل مجلد المشروع ثم نفّذ:

```powershell
.\start-local.ps1
```

ثم افتح:

```text
http://127.0.0.1:3000
```

عند النشر على Render أو Railway يضبط الخادم تلقائيًا المنفذ من المتغير
`PORT` ويستمع على جميع الواجهات الشبكية. استخدم أمر التشغيل `npm start`،
وأضف `DATABASE_URL` كمتغير سري للاتصال بقاعدة Neon.

عند ضبط `DATABASE_URL` يستخدم الخادم PostgreSQL عبر حزمة `pg` (يمكن تشغيل
`postgres-schema.sql` أولًا، مع إنشاء الجداول تلقائيًا عند بدء الخادم). إذا لم
يكن المتغير موجودًا، يستمر التشغيل المحلي باستخدام ملف `souqna.sqlite`.

لترحيل البيانات من SQLite، صدّر الجداول المطلوبة إلى CSV ثم حمّلها إلى
PostgreSQL بعد تشغيل المخطط، مع الحفاظ على قيم `id` و`created_at`. مثال:

```powershell
$env:DATABASE_URL = "postgres://user:password@host:5432/souqna"
psql $env:DATABASE_URL -f postgres-schema.sql
npm start
```

البيانات المحلية محفوظة في ملف `souqna.sqlite`. حاليًا تم تجهيز API للإعلانات:

- `GET /api/health`
- `GET /api/listings`
- `POST /api/listings`
- `GET /api/my-listings`
- `PATCH /api/listings/:id`
- `DELETE /api/listings/:id`
- `GET /api/messages?listingId=:id`
- `POST /api/messages`
- `POST /api/reports`
- `GET /api/reviews?listingId=:id`
- `POST /api/reviews`
- `POST /api/orders`
- `GET /api/orders` (للمستخدم المسجل)
- `GET /api/admin/orders` و`PATCH /api/admin/orders` (للمدير)

تُحفظ الطلبات في جدولي `orders` و`order_items`. يمكن للزائر إتمام الطلب،
بينما تظهر الطلبات المرتبطة بالحساب داخل نافذة "حسابي". حالة الطلب الحالية
هي `pending` وتظهر للمستخدم كمرحلة "تم الطلب" إلى أن تُضاف إدارة الشحن لاحقًا.
يمكن للمدير تحديث الحالة إلى `processing` أو `shipped` أو `delivered` أو
`cancelled` من لوحة الإدارة.

يمكن للإعلان الآن الاحتفاظ بما يصل إلى 5 صور مضغوطة تلقائيًا من المتصفح عبر
الحقل `images`. تُحفظ الصور داخل PostgreSQL بصيغة JSON Data URL، ويُرفض الطلب
إذا تجاوز الحجم الإجمالي 8 ميجابايت. يضيف الخادم عمود `images` تلقائيًا عند
بدء التشغيل، لذلك لا يلزم ترحيل يدوي لقاعدة البيانات الحالية.

وحسابات المستخدمين والجلسات:

- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/auth/me`
- `POST /api/auth/logout`

كلمات المرور تحفظ كقيم مشتقة باستخدام `scrypt`، والجلسة تحفظ في Cookie محمية من JavaScript. لا تتغير مسارات API أو استجاباتها عند استخدام PostgreSQL.

المحادثات المحلية مرتبطة بالإعلان والمستخدم، ولا يستطيع المستخدم قراءة محادثة إعلان لا يملكه أو لم يرسل لها رسالة.

المرحلة الثانية أضافت البلاغات والتقييمات مع منع التقييم الذاتي وتكرار التقييم والبلاغ. شارة التوثيق الحالية واجهة أولية، وتحتاج لاحقًا إلى إجراء تحقق إداري فعلي.

للوحة الإدارة المحلية استخدم البريد `admin@souqna.local` عند إنشاء الحساب، ثم افتح "حسابي". ويمكن تغيير البريد الإداري عبر متغير البيئة `ADMIN_EMAIL` قبل تشغيل الخادم. لوحة الإدارة تعرض البلاغات وتسمح بإبقاء الإعلان أو إخفائه.
