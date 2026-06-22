# python-flask
Ajoyib, ushbu talablar asosida Flask loyihasining minimal strukturasi, kodlari va GitHub uchun tayyor README.md faylini taqdim etaman.

Loyiha quyidagi fayllardan iborat bo'ladi:

app.py — asosiy dastur kodi.

requirements.txt — kerakli kutubxonalar ro'yxati.

README.md — loyihani ishga tushirish bo'yicha qo'llanma.

1. Dastur kodi (app.py)
Ushbu faylda so'ralgan ikkita (/ va /time) route yozilgan va dastur debug rejimida ishga tushadi:

Python
from flask import Flask, jsonify
from datetime import datetime

app = Flask(__name__)

# 1-route: Asosiy sahifa
@app.route('/')
def home():
    return "Xush kelibsiz! Flask ilovasi muvaffaqiyatli ishlamoqda. Vaqtni ko'rish uchun /time sahifasiga o'ting."

# 2-route: Joriy vaqtni JSON formatida qaytarish
@app.route('/time')
def get_time():
    now = datetime.now()
    return jsonify({
        "current_time": now.strftime("%Y-%m-%d %H:%M:%S"),
        "timezone": "Tashkent"
    })

if __name__ == '__main__':
    # debug=True rejimida ishga tushirish
    app.run(debug=True)
2. Kutubxonalar ro'yxati (requirements.txt)
Loyiha ishlashi uchun faqatgina Flask kutubxonasi talab etiladi:

Plaintext
flask>=3.0.0
3. GitHub uchun qo'llanma (README.md)
GitHub repoga joylashtirish va loyihani qanday ishga tushirishni tushuntiruvchi tayyor README.md fayli:

Markdown
# Sodda Flask Vaqt API Loyihasi

Ushbu loyiha Flask freymvorkida yaratilgan minimal web-ilova bo'lib, ikkita route (`/` va `/time`) orqali ishlaydi.

## Loyiha tarkibi
* `app.py` - Asosiy dastur kodi (debug rejimida ishlaydi)
* `requirements.txt` - Kerakli Python kutubxonalari

## Qanday ishga tushiriladi?

Loyihani kompyuteringizda ishga tushirish uchun quyidagi qadamlarni bajaring:

### 1. Repozitoriyani yuklab olish (Clone)
Terminal yoki Git kross-platformasida quyidagi buyruqni bajaring:
```bash
git clone <github-repo-linki>
cd <repo-nomi>
2. Virtual muhit yaratish va faollashtirish (Tavsiya etiladi)
Bash
# Windows uchun:
python -m venv venv
venv\Scripts\activate

# macOS / Linux uchun:
python3 -m venv venv
source venv/bin/activate
3. Kerakli kutubxonalarni o'rnatish
requirements.txt faylidagi barcha bog'liqliklarni yuklab olish:

Bash
pip install -r requirements.txt
4. Loyihani ishga tushirish
Ilovani ishga tushirish uchun quyidagi buyruqni bering:

Bash
python app.py
Ilova mahalliy serverda http://127.0.0.1:5000 manzilida ishga tushadi.

Raxna (Routes) ro'yxati
http://127.0.0.1:5000/ — Bosh sahifa (Oddiy matn ko'rinishida javob).

http://127.0.0.1:5000/time — Joriy vaqt (JSON formatida javob).


---

### GitHub-ga yuklash bosqichlari (Eslatma)
Agar loyihani GitHub-ga birinchi marta yuklayotgan bo'lsangiz, terminalda ketma-ket quyidagi buyruqlarni yozasiz:

```bash
git init
git add .
git commit -m "Initial commit: Flask app va README qo'shildi"
git branch -M main
git remote add origin <Sizning_GitHub_Repo_Linkigiz>
git push -u origin main
