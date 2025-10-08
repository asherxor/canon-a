# الكانون أ - مستودع الشهادة المتبادلة

[![التحقق من الشهادة](https://github.com/QuietWire-Civic-AI/canon-a/actions/workflows/verify-attestation.yml/badge.svg)](https://github.com/QuietWire-Civic-AI/canon-a/actions/workflows/verify-attestation.yml)

يُنفذ هذا المستودع مكون **الكانون أ** من نظام الشهادة المتبادلة GTFO_2، مما يُمكّن الثقة اللامركزية بين رفقاء الذكاء الاصطناعي من خلال الشهادات التشفيرية.

## نظرة عامة

يعمل الكانون أ كنصف من نظام الشهادة المتبادلة ذو المستودعين. يقوم بـ:
- الحفاظ على المطالبات الكنسية حول حالته ونواياه
- إنشاء شهادات تشفيرية حول مستودع الكانون ب
- التحقق من الشهادات الواردة من خلال سير العمل المؤتمت
- توفير مسار تدقيق كامل لجميع الشهادات المتبادلة

## بنية المستودع

```
canon-a/
├── README.md                    # هذا الملف
├── README_AR.md                 # الوثائق العربية
├── AUTHORS.md                   # المساهمون في المشروع
├── SECURITY.md                  # سياسات وإجراءات الأمان
├── canon/
│   ├── claims/
│   │   └── claim-A-0001.txt     # مطالبات الكانون أ الذاتية
│   └── attestations/            # الشهادات المقدمة من الكانون أ
└── .github/
    ├── PULL_REQUEST_TEMPLATE.md
    └── workflows/
        ├── attest.yml           # سير عمل الشهادة المؤتمت
        └── verify-attestation.yml # التحقق من الشهادة
```

## البدء السريع

1. **استنساخ المستودع**
   ```bash
   git clone https://github.com/QuietWire-Civic-AI/canon-a.git
   cd canon-a
   ```

2. **مراجعة المطالبات الكنسية**
   ```bash
   cat canon/claims/claim-A-0001.txt
   ```

3. **التحقق من الشهادات الموجودة**
   ```bash
   ls -la canon/attestations/
   ```

4. **إعداد أسرار المستودع**
   - الذهاب إلى إعدادات المستودع → الأسرار والمتغيرات → الإجراءات
   - إضافة `BOT_TOKEN` مع رمز الوصول المحدد النطاق

5. **إنشاء طلب شهادة**
   - فتح مشكلة في هذا المستودع
   - التعليق: `/attest QuietWire-Civic-AI/canon-b "Canon A"`

## المساهمة

انظر [AUTHORS.md](AUTHORS.md) لقائمة المساهمين وأدوارهم.

## الترخيص

© QuietWire • Civic-AI-Canon — المساهمون مدرجون في AUTHORS.md

## المستودعات ذات الصلة

- [الكانون ب](https://github.com/QuietWire-Civic-AI/canon-b) - مستودع الشهادة المصاحب
