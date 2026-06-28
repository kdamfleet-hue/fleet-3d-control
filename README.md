# 🚛 Fleet 3D Control — نموذج أولي حقيقي

واجهة ثلاثية الأبعاد بـ **Three.js** لعرض مركبات الأسطول لحظيًا فوق مستوى أرضي، مع كاميرا
وتحكم بالدوران/التقريب، وتلوين حسب الحالة/السرعة. تقرأ المواقع من `/api/gps` (GpsCockpit)
وتتراجع تلقائيًا إلى **بيانات تجريبية متحركة** إذا تعذّر المصدر — فتعمل فورًا بلا إعداد.

## التشغيل محليًا
بما أنها تستخدم وحدات ES Modules من CDN، شغّلها عبر خادم محلي بسيط (وليس فتح الملف مباشرة):

```bash
cd fleet-3d-control
python -m http.server 8080
# ثم افتح: http://localhost:8080
```

عند الفتح ستظهر الأرضية والشبكة و8 مركبات تتحرك (وضع تجريبي) مع لوحة معلومات أعلى اليمين.

## الربط بالبيانات الحيّة
المصدر هو endpoint موجود فعلًا في تطبيق Flask:

```
GET /api/gps   →  يمرّر إلى GpsCockpit (يتطلب GPS_PERMANENT_TOKEN في بيئة الخادم)
```

في [config.js](config.js) اضبط `apiUrl`:
- **نفس النطاق (موصى به):** انشر هذه الواجهة على نفس خادم Flask واجعل `apiUrl = "/api/gps"`.
- **نطاق مختلف:** أبقِها مطلقة، لكن ستحتاج تفعيل **CORS** على `/api/gps` (وربما تجاوز حماية تسجيل الدخول لهذا المسار) وإلا يبقى الوضع تجريبيًا.

### تطبيع البيانات
شكل GpsCockpit غير ثابت، لذا توجد طبقة `normalize()` تقبل:
- مصفوفة مباشرة، أو كائن فيه مصفوفة (`data`/`assets`/`items`…)، أو **GeoJSON FeatureCollection**.
- تلتقط الإحداثيات من مفاتيح شائعة: `lat/latitude/y` و`lng/lon/longitude/x`، والاسم من `name/plate/label`،
  والحالة من `status/alarm`، والسرعة من `speed`.

إن كان شكل بياناتك مختلفًا، أرسل لي عيّنة JSON من `/api/gps` وأضبط المُطبّع عليها بدقّة.

## النشر على GitHub Pages
المستودع `kdamfleet-hue/fleet-3d-control` فارغ حاليًا. لرفع هذه الواجهة:

```bash
cd fleet-3d-control
git init -b main
git add -A
git commit -m "Fleet 3D control prototype"
git remote add origin https://github.com/kdamfleet-hue/fleet-3d-control.git
git push -u origin main      # سيطلب اسم المستخدم + توكن
```
ثم من GitHub: **Settings → Pages → Branch = main / root**. ملاحظة: عبر GitHub Pages ستحتاج CORS للبيانات الحيّة.

## الإعدادات ([config.js](config.js))
| المفتاح | الوصف |
|---|---|
| `apiUrl` | مصدر المواقع (اجعلها `/api/gps` عند نفس النطاق) |
| `pollIntervalMs` | فترة التحديث (افتراضي 3000) |
| `center` | مركز العالم (الدمام) |
| `worldScale` | وحدات المشهد لكل متر |
| `speed.warn / speed.danger` | عتبات تلوين الخطر |

## الحالة
نموذج أولي **يعمل فعليًا** (أرضية + مركبات 3D متحركة + كاميرا + تطبيع بيانات + تراجع تجريبي).
الخطوات التالية الممكنة: مسارات 3D، مناطق حرجة 3D، مجسّمات GLB حقيقية، مبانٍ، خطوط أثر للحركة.
