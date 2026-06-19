# python-flask
Mana barcha talablarga javob beradigan va tushunarli qilib yozilgan Flask loyiha strukturasi va kodi. Bu oddiy eslatmalar (Notes) ilovasi bo'lib, uning yordamida loyihani tezda ishga tushirishingiz mumkin.

1. Loyiha strukturasi
Loyiha papkasida quyidagi fayllarni yaratib oling:

Plaintext
note_app/
│
├── app.py
├── templates/
│   └── index.html
└── README.md
2. Kod qismi
app.py fayli
Bu yerda Flask ilovasi, SQLite bazasi, Note modeli va bosh sahifa logikasi joylashgan. Shuningdek, bazada ma'lumot bo'lmasa, avtomatik 3 ta seed notlarni qo'shadigan funksiya yozilgan.

Python
from datetime import datetime
from flask import Flask, render_template
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# SQLite ma'lumotlar bazasi sozlamalari
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

# 1. Note modeli
class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    body = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f'<Note {self.title}>'

# 2. Bosh sahifa (Notlar ro'yxati, yangilari yuqorida)
@app.route('/')
def index():
    # order_by(Note.created_at.desc()) yangi notlarni yuqoriga chiqaradi
    notes = Note.query.order_by(Note.created_at.desc()).all()
    return render_template('index.html', notes=notes)

# 3. Bazani yaratish va Seed ma'lumotlarni qo'shish funksiyasi
def seed_data():
    with app.app_context():
        db.create_all()  # Bazani va jadvallarni yaratadi
        
        # Agar baza bo'sh bo'lsa, seed ma'lumotlarni qo'shadi
        if Note.query.count() == 0:
            seed_notes = [
                Note(title="Birinchi eslatma", body="Bu loyihadagi eng birinchi seed eslatma hisoblanadi."),
                Note(title="Xaridlar ro'yxati", body="Sut, non, tuxum va mevalar sotib olish kerak."),
                Note(title="Flask haqida", body="Flask juda yengil va moslashuvchan mikro freymvorkdir.")
            ]
            db.session.bulk_save_objects(seed_notes)
            db.session.commit()
            print("Seed ma'lumotlar muvaffaqiyatli qo'shildi!")

if __name__ == '__main__':
    seed_data()  # Dastur ishga tushishidan oldin bazani tekshiradi
    app.run(debug=True)
templates/index.html fayli
Notlarni chiroyli ko'rinishda chiqarish uchun oddiy HTML shablon.

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mening Eslatmalarim</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f4f9; }
        .container { max-width: 600px; margin: auto; }
        h1 { color: #333; text-align: center; }
        .note-card { background: white; padding: 20px; margin-bottom: 15px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .note-title { margin-0: 0; color: #007BFF; }
        .note-date { font-size: 0.8rem; color: #777; margin-bottom: 10px; }
        .note-body { color: #555; line-height: 1.5; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Eslatmalar Ro'yxati</h1>
        <hr>
        {% if notes %}
            {% for note in notes %}
                <div class="note-card">
                    <h2 class="note-title">{{ note.title }}</h2>
                    <div class="note-date">Yaratilgan vaqti: {{ note.created_at.strftime('%Y-%m-%d %H:%M:%S') }}</div>
                    <p class="note-body">{{ note.body }}</p>
                </div>
            {% endfor %}
        {% else %}
            <p style="text-align:center;">Hozircha hech qanday eslatma yo'q.</p>
        {% endif %}
    </div>
</body>
</html>
3. README.md fayli (Ishga tushirish yo'riqnomasi)
Siz loyiha ildiz papkasiga joylashtirishingiz kerak bo'lgan tayyor markdown matni:

Markdown
# Note Application

Bu Flask va Flask-SQLAlchemy yordamida yaratilgan oddiy eslatmalar ilovasi. Loyihada SQLite ma'lumotlar bazasidan foydalanilgan.

## Loyihani ishga tushirish qadamlari

Loyihani kompyuteringizda ishga tushirish uchun quyidagi buyruqlarni ketma-ket bajaring:

### 1. Virtual muhitni yaratish va faollashtirish
Terminalda loyiha papkasiga kiring va quyidagilarni yozing:

**Windows uchun:**
```bash
python -m venv venv
venv\Scripts\activate
Mac/Linux uchun:

Bash
python3 -m venv venv
source venv/bin/activate
2. Zaruriy kutubxonalarni o'rnatish
Ilova ishlashi uchun Flask va Flask-SQLAlchemy kutubxonalarini o'rnatamiz:

Bash
pip install Flask Flask-SQLAlchemy
3. Loyihani ishga tushirish
Dasturni ishga tushirish uchun quyidagi buyruqni bering:

Bash
python app.py
Ilova ishga tushganda avtomatik ravishda instance/app.db fayli (ma'lumotlar bazasi) yaratiladi va unga kamida 3 ta seed notlar qo'shiladi.

4. Brauzerda tekshirish
Brauzeringizni oching va quyidagi manzilga kiring:
http://127.0.0.1:5000/

Bosh sahifada eng oxirgi qo'shilgan eslatmalar eng yuqorida joylashgan tartibda ko'rinadi.
