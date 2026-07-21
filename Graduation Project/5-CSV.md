بص، أول حاجة أحب أقولها إن **الـ AI عمل شغل نضيف جدًا**، ومش مجرد "خلى الـ CSV يشتغل"، لأ هو حاول يعمله بطريقة Production Ready.

لكن فيه كام نقطة أنا هقولك عليها في الآخر كنت هغيرها أو أحسنها لو أنا بعمل Code Review في شركة.

هشرحلك كل حاجة من أول ما اليوزر يضغط Upload لحد ما البيانات تدخل الداتا بيز.

---

# High Level Flow

الـ Flow كله بقى كده

```text
User

    │

Choose CSV

    │

POST /sku/import

    │

Controller

    │

SkuService.importCsv()

    │

Parse CSV

    │

Map CSV Row

    │

CreateSkuDto

    │

Validation

    │

Duplicate Check (CSV)

    │

Duplicate Check (Database)

    │

Mapper

    │

Transaction

    │

Insert

    │

Response
```

وده فعلا الـ Flow الصح.

---

# الجزء الأول

## Controller

```ts
@Post('import')
```

هو عمل Endpoint جديد.

يعني اليوزر هيبعت

```
POST /sku/import
```

---

بعدها

```ts
@UseInterceptors(FileInterceptor('file'))
```

وده أهم جزء.

---

## FileInterceptor

وده خاص بـ Multer.

هو المسئول عن

```
multipart/form-data
```

يعني

بدل JSON

الـ Request هيبقى

```
file = sku.csv
```

---

لو معملش FileInterceptor

مش هيعرف يستقبل فايل أصلا.

---

بعدها

```ts
@UploadedFile()
```

وده بيقول لـ Nest

"هاتلي الفايل اللي اليوزر رفعه."

هيجيلك

```ts
Express.Multer.File
```

اللى جواه

```
originalname

buffer

mimetype

size
```

مثلا

```
{
 originalname:"products.csv",

 buffer:<Buffer...>,

 size:2400
}
```

---

## Validation

هو عمل

```ts
if(!file)
```

وده Use Case مهم.

لو اليوزر ضغط Import من غير فايل

يرجع

```
400

CSV file is required
```

---

بعدها

```ts
endsWith(".csv")
```

عشان يمنع

```
photo.png

resume.pdf
```

---

بعدها

```ts
this.skuService.importCsv(file.buffer)
```

لاحظ

هو مبعتش

```
Express.Multer.File
```

هو بعت

```
Buffer
```

وده أحسن.

ليه؟

عشان Service ميكونش عندها أى علاقة بـ HTTP.

الـ Service تستقبل Data فقط.

وده Clean Architecture.

---

# الجزء التانى

## importCsv()

ده قلب الفيتشر.

كل الشغل الحقيقي هنا.

---

أول حاجة

```ts
if(buffer.length==0)
```

Use Case

لو الملف

```
0 KB
```

يرجع

```
Uploaded CSV file is empty
```

---

# Parsing

بعدها

```ts
parse(buffer)
```

هو استخدم

```
csv-parse
```

وده من أحسن Libraries.

---

هى بتحول

```
sku,name,cost

A1,Laptop,100

A2,Mouse,20
```

إلى

```ts
[
{

sku:"A1",

name:"Laptop",

cost:"100"

},

{

sku:"A2",

name:"Mouse",

cost:"20"

}

]
```

لاحظ

كل حاجة String.

---

## options

### columns:true

دى معناها

```
أول Row

Headers
```

بدل

```
[
["sku","name"],

["A1","Laptop"]
]
```

تبقى

```
{

sku:"A1",

name:"Laptop"
}
```

---

### trim:true

يشيل

```
SKU001
```

من

```
      SKU001
```

---

### bom:true

دى جميلة جدا.

لو الـ Excel حفظ

```
UTF8 BOM
```

أول Header

ممكن يبقى

```
﻿skuCode
```

مش

```
skuCode
```

وده كان هيبوظ الدنيا.

هو شاله.

---

### skip_empty_lines

لو فيه

```
سطر فاضى
```

يتجاهله.

---

### relax_column_count

لو Row ناقص Column

ميكسرش Parsing كله.

يسيب Validation تمسكه.

دى فكرة حلوة.

---

# لو Parsing فشل

مثلا

```
"

"

"

```

CSV بايظ.

هيعمل

```
BadRequest
```

بدل

```
500
```

وده صح.

---

# Pass 1

دى أهم فكرة.

هو عمل

```
Pass 1
```

وبعدين

```
Pass 2
```

بدل ما يحفظ أثناء القراءة.

وده Design ممتاز.

---

# Pass 1

يمشى على كل Row

```
for(...)
```

---

أول حاجة

```
mapCsvRowToPlainDto()
```

---

# ليه؟

لأن الـ CSV

ممكن يبقى

```
costPrice
```

أو

```
cost
```

هو خلاهم

```
cost
```

---

مثلا

```
sellingPrice

price
```

كلهم

↓

```
price
```

وده اسمه

Header Normalization.

---

بعدها

```
plainToInstance()
```

دى من

```
class-transformer
```

---

ليه؟

عشان

CreateSkuDto

عنده

```
@Transform()

@IsString()

@Matches()

```

كل ده يشتغل.

---

بعدها

```
validate()
```

دى من

```
class-validator
```

وهنا حصل السحر.

هو معملش Validation جديد.

هو استخدم نفس Validation بتاعة

```
POST /sku
```

وده أفضل قرار فى الكود كله.

---

## لو فيه Error

هيجمع كل Errors

```
sku required

price invalid

cost invalid
```

ويبعتهم

```
;

```

فى Message واحدة.

---

# Duplicate داخل CSV

مثلا

```
SKU001

SKU001
```

هو استخدم

```
Set
```

وده أسرع من Array.

التعقيد O(1) تقريبًا لكل فحص.

هيكتشف إن التانى مكرر قبل حتى ما يكلم الداتابيز.

وده ممتاز.

---

# Pass 2

بعد ما خلص Validation

بدأ يكلم الداتابيز.

وده صح.

---

# findExistingSkuCodes()

بدل

```
findOne()

findOne()

findOne()

```

1000 مرة

هو عمل

```
IN(...)
```

وده أسرع بمراحل.

مثلا

```
SELECT sku_code

FROM sku

WHERE sku_code IN(...)
```

مرة واحدة.

وده بيمنع مشكلة **N+1 Queries**.

---

# ليه Chunk 1000؟

عشان لو عندك 500 ألف SKU، متبعتش Query ضخمة جدًا.

يقسمها:

```
1000

1000

1000
```

وده عملي.

---

# Mapper

بعد Validation

عمل

```
skuMapper.toEntity(dto)
```

وده ممتاز.

معملش

```
new Sku()
```

فى Service.

احترم Architecture.

---

# Transaction

بعدها

```
transaction()
```

وده مهم جدًا.

يعنى

لو حصل Error أثناء حفظ Batch

الداتابيز هتقدر ترجع الحالة السابقة (Rollback) حسب مكان الخطأ داخل الـ Transaction.

---

# Batch Insert

هو عمل

```
500

500

500
```

بدل

```
save()

save()

save()
```

وده أسرع بكتير.

---

# Response

فى الآخر رجع

```
{

totalRows,

successful,

failed,

errors

}
```

وده Response احترافى.

---

# أهم الـ Use Cases

## ✅ الحالة 1

CSV صحيح.

```
100 Rows
```

النتيجة

```
100 Success

0 Failed
```

---

## ✅ الحالة 2

ملف فاضى.

```
0 Bytes
```

↓

```
400
```

---

## ✅ الحالة 3

CSV بدون Data.

```
sku,name
```

↓

```
CSV contains no rows
```

---

## ✅ الحالة 4

Duplicate داخل CSV.

```
SKU1

SKU1
```

↓

```
Duplicate SKU
```

---

## ✅ الحالة 5

Duplicate فى DB.

```
SKU001
```

موجود بالفعل.

↓

```
SKU already exists
```

---

## ✅ الحالة 6

Price

```
abc
```

↓

Validation Error.

---

## ✅ الحالة 7

Negative Cost

```
-10
```

↓

Validation Error.

---

## ✅ الحالة 8

Malformed CSV.

↓

BadRequest.

---

# هل الكود Production Ready؟

أقول **حوالي 9/10**.

فيه شوية تحسينات كنت هقترحها في Code Review:

1. **إعادة استخدام Create SKU Service**
    
    أنت قلت في البرومبت: _"Reuse existing business logic"_، لكن الكود أعاد استخدام الـ DTO والـ Mapper فقط. لو `createSku()` عندك فيها منطق إضافي (Audit Log، Events، Cache، Domain Rules...) فهو لم يُعد استخدامه. الأفضل استخراج منطق إنشاء SKU في Method داخلية مشتركة بدل ما يبقى فيه مسارين.
    
2. **MIME Type Validation**
    
    هو بيتحقق من الامتداد `.csv` فقط. ده سهل يتزور. الأفضل التحقق أيضًا من `file.mimetype` مع الامتداد.
    
3. **Race Condition**
    
    بين `findExistingSkuCodes()` و `save()` ممكن Request تاني يدخل بنفس `skuCode`. الحل الحقيقي هو الاعتماد على **Unique Constraint** في قاعدة البيانات مع التقاط الخطأ وتحويله إلى رسالة مناسبة.
    
4. **Streaming للملفات الضخمة**
    
    هو بيقرأ الملف كله في الذاكرة (`buffer`). ده ممتاز لملفات صغيرة ومتوسطة. لكن لو حد رفع ملف 500MB، الأفضل استخدام **Streaming Parser** بدل تحميل الملف بالكامل.
    

---

## التقييم النهائي

- **Architecture:** ⭐⭐⭐⭐⭐ (10/10)
    
- **Validation Reuse:** ⭐⭐⭐⭐⭐ (10/10)
    
- **Performance:** ⭐⭐⭐⭐⭐ (9.5/10)
    
- **Error Handling:** ⭐⭐⭐⭐⭐ (10/10)
    
- **Production Readiness:** ⭐⭐⭐⭐⭐ (9/10 بسبب النقطتين الأخيرتين)
    

لو سألك Tech Lead في انترفيو: **"ازاي عملت CSV Import؟"**، فالـ Flow اللي شرحناه فوق هو بالضبط اللي تتكلم عنه، وهيبان إنك فاهم كل قرار اتاخد في التنفيذ، مش مجرد استخدمت مكتبة وخلاص.