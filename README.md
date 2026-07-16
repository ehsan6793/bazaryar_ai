<div dir="rtl">

# 🤖 بازاریار (BazarYar)

### دستیار فروش هوشمند تلگرامی — ساخته‌شده با LangGraph

</div>

> An AI-powered Telegram sales assistant built with **LangGraph**, combining **RAG (ChromaDB)**
> and **structured Pandas filtering** over a 500-product catalog — with persistent memory,
> image/link understanding, and scheduled follow-ups.

<div dir="rtl">

بازاریار یک دستیار فروش هوشمند برای فروشگاه‌های اینترنتی است: نیاز واقعی مشتری را با پرسیدن
سوال‌های هوشمند می‌فهمد، با ترکیب جستجوی معنایی (RAG) و فیلتر دقیق (Pandas) بهترین محصولات را
پیشنهاد می‌دهد، مکالمه را به‌خاطر می‌سپارد، و در صورت نیاز مشتری را با پیام‌های پیگیری به سمت خرید
هدایت می‌کند.

</div>

---

## فهرست

- [ویژگی‌ها](#-ویژگی‌ها)
- [نمودار گراف](#-نمودار-گراف)
- [تک‌استک](#-تک‌استک)
- [ساختار پروژه](#-ساختار-پروژه)
- [راه‌اندازی](#-راه‌اندازی)
- [تست](#-تست-بدون-api-key)
- [نحوه کارکرد](#-نحوه-کارکرد)
- [نگاشت به معیارهای ارزیابی](#-نگاشت-به-معیارهای-ارزیابی)
- [مراحل باقی‌مانده تحویل](#-مراحل-باقی‌مانده-تحویل)
- [لایسنس](#-لایسنس)

---

## ✨ ویژگی‌ها

| | ویژگی | توضیح |
|---|---|---|
| 🧠 | تشخیص نیت مکالمه‌محور | با LLM؛ اگر اطلاعات ناقص باشد، حداکثر ۲ سوال کوتاه می‌پرسد |
| 🔍 | جستجوی ترکیبی RAG + Pandas | فیلتر دقیق (قیمت/برند/دسته) + شباهت معنایی روی توضیحات محصول |
| 🖼️ | تحلیل تصویر محصول | عکس ارسالی کاربر با مدل ویژن Claude توصیف و به query تبدیل می‌شود |
| 🔗 | استخراج از لینک خارجی | لینک محصول از سایت دیگر → پیدا کردن مشابه در کاتالوگ داخلی |
| 💬 | حافظه دو‌لایه | کوتاه‌مدت (بین پیام‌های یک چت) + بلندمدت (برای زمان‌بند پیگیری) — بدون جستجوی تکراری |
| ⏰ | پیگیری خودکار | پیام بعد از ۱ ساعت بی‌پاسخی، و کد تخفیف بعد از ۴۸ ساعت بدون خرید |
| 🔁 | Retry + Error Handling | تمام Tool Callها با retry خودکار (حداکثر ۳ تلاش) پوشش داده شده‌اند |
| 🎭 | پرسونای ثابت | لحن و شخصیت یکسان («بازاریار») در طول کل مکالمه |

---

## 🗺️ نمودار گراف

<table>
<tr><td align="center"><b>English</b></td></tr>
<tr><td><img src="docs/graph_en.png" alt="LangGraph flow - English"></td></tr>
<tr><td align="center"><b>فارسی</b></td></tr>
<tr><td><img src="docs/graph_fa.png" alt="نمودار گراف - فارسی"></td></tr>
</table>

---

## 🛠️ تک‌استک

| لایه | ابزار |
|---|---|
| Agent orchestration | LangGraph, LangChain |
| LLM | Claude (Anthropic) — متن و ویژن |
| Vector search / RAG | ChromaDB + Sentence-Transformers (چندزبانه) |
| Structured filtering | Pandas |
| Bot interface | python-telegram-bot |
| Scheduling | APScheduler |
| Memory | SqliteSaver (کوتاه‌مدت) + SQLite (بلندمدت) |

---

## 📁 ساختار پروژه

```text
bazaryar_ai/
├── data/
│   └── products_500.csv        # کاتالوگ محصولات
├── docs/
│   ├── graph_en.png             # نمودار گراف (انگلیسی)
│   └── graph_fa.png             # نمودار گراف (فارسی)
├── src/
│   ├── state.py                 # AgentState — State Management
│   ├── config.py                # تنظیمات از .env
│   ├── persona.py               # پرسونای ثابت + پرامپت‌های سیستمی
│   ├── graph.py                 # اسمبل نهایی گراف LangGraph
│   ├── nodes/                   # نودهای گراف
│   │   ├── router.py                  # classify_intent + route_after_intent
│   │   ├── slot_filling.py            # extract_slots + ask_for_missing_slots
│   │   ├── hybrid_search.py           # ترکیب RAG + Pandas
│   │   ├── memory_answer.py           # پاسخ بدون جستجوی جدید
│   │   ├── compare_and_preference.py  # مقایسه و تغییر ترجیحات
│   │   ├── media_input.py             # image_analysis + link_extraction
│   │   ├── response_generation.py     # پاسخ نهایی با پرسونا
│   │   └── error_handler.py           # log_error
│   ├── tools/
│   │   ├── pandas_tool.py             # فیلتر دقیق روی CSV
│   │   ├── vector_tool.py             # RAG با ChromaDB
│   │   ├── image_tool.py              # ویژن Claude برای تحلیل عکس
│   │   ├── link_tool.py               # استخراج از لینک خارجی
│   │   ├── discount_tool.py           # کد تخفیف
│   │   └── retry.py                   # with_retry — Retry + Error Handling
│   └── memory/
│       ├── checkpointer.py            # حافظه کوتاه‌مدت (SqliteSaver)
│       └── long_term.py               # حافظه بلندمدت برای زمان‌بند پیگیری
├── bot/
│   ├── telegram_bot.py          # هندلرهای تلگرام + APScheduler
│   └── main.py                  # نقطه ورود اجرا
├── scripts/
│   └── build_vector_index.py    # ساخت یک‌باره ایندکس Chroma
├── tests/
│   └── test_graph_smoke.py      # تست بدون نیاز به API Key
├── requirements.txt
└── .env.example
```

---

## 🚀 راه‌اندازی

### پیش‌نیازها
- Python 3.11+
- کلید API از [Anthropic](https://console.anthropic.com) (برای LLM و ویژن)
- توکن ربات تلگرام از [@BotFather](https://t.me/BotFather)

### نصب

```bash
git clone https://github.com/USERNAME/bazaryar-ai.git
cd bazaryar-ai

python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

### تنظیمات

```bash
cp .env.example .env
```

مقادیر زیر را در `.env` پر کن:

```env
ANTHROPIC_API_KEY=your_key_here
TELEGRAM_BOT_TOKEN=your_token_here
```

### ساخت ایندکس RAG (یک‌بار)

```bash
python scripts/build_vector_index.py
```

### اجرای ربات

```bash
python bot/main.py
```

---

## ✅ تست بدون API Key

```bash
python tests/test_graph_smoke.py
```

این تست، بدون فراخوانی هیچ مدلی، موارد زیر را چک می‌کند:
- ساختار درست فایل CSV محصولات
- عملکرد فیلتر Pandas
- کامپایل‌شدن کامل گراف LangGraph (همه نودها و یال‌های شرطی)

---

## ⚙️ نحوه کارکرد

<details>
<summary><b>جستجوی ترکیبی RAG + Pandas</b></summary>
<br>

`hybrid_search_node` اول با Pandas فیلتر دقیق (بودجه/برند/دسته) می‌زند، سپس با جستجوی برداری
(Chroma) نتایج معنایی را اضافه می‌کند، ترکیب و رتبه‌بندی می‌کند، و در نهایت حداقل ۳ محصول به همراه
دلیل انتخاب هرکدام برمی‌گرداند. اگر نتایج کافی نبود، نزدیک‌ترین جایگزین‌های همان دسته پیشنهاد
می‌شوند.
</details>

<details>
<summary><b>حافظه بدون جستجوی تکراری</b></summary>
<br>

اگر intent برابر `followup_question` باشد (یعنی کاربر درباره محصولی که قبلاً معرفی شده سوال
می‌پرسد)، گراف مستقیم به `memory_answer_node` می‌رود و هیچ Tool Callای اجرا نمی‌شود — پاسخ فقط از
`last_products` در state ساخته می‌شود.
</details>

<details>
<summary><b>مدیریت خطا و Retry</b></summary>
<br>

هر Tool Call (`pandas_tool`, `vector_tool`, `image_tool`, `link_tool`) با دکوراتور `with_retry()`
پوشش داده شده (پیش‌فرض ۳ تلاش با backoff نمایی). اگر باز هم شکست بخورد، نود `log_error` خطا را ثبت
می‌کند و `response_generation_node` یک پیام fallback مؤدبانه به کاربر می‌دهد.
</details>

<details>
<summary><b>پیگیری زمان‌بندی‌شده</b></summary>
<br>

`bot/telegram_bot.py` با APScheduler هر ۱۰ دقیقه چک می‌کند:
- چه کاربرانی بیش از **۱ ساعت** پاسخ نداده‌اند → پیام پیگیری ساده
- چه کاربرانی بیش از **۴۸ ساعت** خرید نکرده‌اند → یادآوری محصول + کد تخفیف
</details>

---

## 📊 نگاشت به معیارهای ارزیابی

| بخش رابریک | پیاده‌سازی |
|---|---|
| طراحی Workflow | `docs/graph_en.png`, `docs/graph_fa.png`, `src/graph.py` |
| LangGraph (State / Routing / Tools / Memory / Error / Retry) | `src/state.py`, `src/nodes/router.py`, `src/tools/*`, `src/memory/*`, `src/tools/retry.py` |
| ترکیب RAG + Pandas | `src/nodes/hybrid_search.py` |
| State، Memory و شخصیت بات | `src/memory/checkpointer.py`, `src/persona.py` |
| قابلیت‌های ربات تلگرام (عکس/لینک/پیگیری) | `bot/telegram_bot.py` |
| کیفیت کد و مستندسازی | ساختار ماژولار + docstring دوزبانه در همه فایل‌ها |

---

MIT — استفاده و تغییر آزاد، مخصوصاً برای اهداف آموزشی.
