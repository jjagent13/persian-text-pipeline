# Persian Text Pipeline 🇮🇷

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-EE4C2C?logo=pytorch&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?logo=fastapi&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Proof--of--Concept-yellow)

[🇬🇧 English](README.en.md) | 🇮🇷 فارسی

پایپ‌لاین پردازش متن فارسی: **Sanitizer** + **Transformer Spell Corrector** + **REST API**

---

## 🎯 مشکل

متن فارسی خام از منابع مختلف (وب، OCR، ASR، چت) پر از نویز است:
- کاراکترهای عربی (ك، ي، ة) به‌جای فارسی (ک، ی، ه)
- اعراب اضافی، نیم‌فاصله اشتباه
- ارقام عربی/فارسی به‌جای انگلیسی
- غلط املایی، تایپو، حذف/جابجایی حروف

## ✅ راه‌حل

| لایه | رویکرد | خروجی |
|------|---------|-------|
| ۱. Sanitizer | قاعده‌محور (regex + نگاشت نویسه) | متن نرمال‌شده |
| ۲. Spell Corrector | Transformer از صفر (۴M پارامتر) | متن تصحیح‌شده |
| ۳. Confidence Guard | ۳ گارد (char_sim، word_overlap، length_ratio) | رد خروجی بی‌کیفیت |
| ۴. REST API | FastAPI + cloudflared tunnel | endpoint عمومی |

**فلسفه طراحی:** هرگز ورودی کاربر را خراب نکن. اگر مدل مطمئن نیست، متن اصلی را برگردان + دلیل.

---

## 📂 ساختار پروژه

| Notebook | توضیح |
|----------|--------|
| `notebooks/pars_corrector.ipynb` | تلاش اول با mT5-small (خطای NaN loss) — درس‌آموخته |
| `notebooks/Persian_Spell_Corrector.ipynb` | Transformer از صفر — کشف باگ shared weights |
| `notebooks/persian_text_pipeline.ipynb` | نسخه نهایی fix‌شده + sanitizer + API |
| `notebooks/Text_Sanitizer_+_API.ipynb` | اتصال checkpoint + Confidence Guard |

---

## 🏗️ معماری

Input Text
↓
[Sanitizer] NFKC → strip controls → ar2fa → digits → zwnj → whitespace
↓
[Tokenizer] Char-level vocab (V=140)
↓
[Encoder] 3 layers × (MHA + FFN) | d_model=256, heads=8, d_ff=512
↓
[Decoder] 3 layers × (Self + Cross + FFN)
↓
[Generator] Linear → softmax → greedy decode
↓
[Guard] char_sim ≥ 0.90 AND word_overlap ≥ 0.70 AND 0.75 ≤ len_ratio ≤ 1.33
↓
Output (یا متن اصلی اگر گارد رد کند)


---

## 📊 نتایج

| متریک | مقدار |
|-------|-------|
| Best val loss | 1.73 |
| پارامترهای مدل | 4,061,579 |
| دقت گارد (روی OOD) | ۱۰۰٪ رد شد (fail-safe) |

> **صادقانه:** مدل روی ۱۲۰ جمله seed آموزش داده شده — proof-of-concept است، نه SOTA. اما معماری و pipeline آماده production هستند.

---

## 🚀 اجرا

```bash
pip install -r requirements.txt
# سپس notebook 03 را در Colab (GPU T4) اجرا کن

🗺️ Roadmap
نگاه کن به docs/ROADMAP.md.

📄 مجوز
MIT — استفاده آزاد.



