# python-flask
Siz aytgan barcha shartlarni, xususan, ketma-ket bir xil xabar yuborishni bloklash (anti-spam) va oxirgi 20 ta xabarni teskari tartibda ko'rsatish mantiqlarini qamrab olgan to'liq Flask ilovasi kodi:

1. app.py — Server kodi
Python
import os
from datetime import datetime
from flask import Flask, render_template, request, redirect, url_for, session, flash

app = Flask(__name__)
# GitHub'ga chiqib ketmasligi uchun maxfiy kalitni xavfsiz muhitdan olamiz
app.secret_key = os.environ.get('FLASK_SECRET_KEY', 'default_vaqtinchalik_kalit_12345')

# Ma'lumotlar bazasi o'rniga vaqtinchalik ro'yxat (In-Memory)
# Har bir element: {"author": ..., "text": ..., "time": ...}
MESSAGES = []

# Test uchun foydalanuvchilar (username: password)
USERS = {
    "ali": "123",
    "valisiz_kod_yoq": "parol123"
}

# 1. Bosh sahifa (Xabarlar ro'yxati va forma)
@app.route('/')
def index():
    # Faqat oxirgi 20 ta xabarni olish (ro'yxat kesmasi yordamida)
    recent_messages = MESSAGES[:20]
    return render_template('index.html', messages=recent_messages)

# 2. Login sahifasi (GET + POST)
@app.route('/login', methods=['GET', 'POST'])
def login():
    if 'username' in session:
        return redirect(url_for('index'))
        
    if request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        
        if username in USERS and USERS[username] == password:
            session['username'] = username
            flash("Tizimga muvaffaqiyatli kirdingiz!", "success")
            return redirect(url_for('index'))
        else:
            flash("Username yoki parol xato!", "danger")
            
    return render_template('login.html')

# 3. Yangi xabar qo'shish (Faqat POST)
@app.route('/post', methods=['POST'])
def post_message():
    # Login bo'lmagan foydalanuvchini tekshirish
    if 'username' not in session:
        flash("Xabar yozish uchun avval tizimga kiring!", "danger")
        return redirect(url_for('login'))
        
    message_text = request.form.get('text', '').strip()
    author = session['username']
    
    if not message_text:
        flash("Xabar matni bo'sh bo'lishi mumkin emas!", "warning")
        return redirect(url_for('index'))
        
    # BONUS: Bir xil matnli xabarni ketma-ket ikki marta yuborishni bloklash
    if MESSAGES:
        last_message = MESSAGES[0] # Eng oxirgi qo'shilgan xabar har doim 0-indeksda turadi
        if last_message['author'] == author and last_message['text'] == message_text:
            flash("Bir xil xabarni ketma-ket yubora olmaysiz!", "danger")
            return redirect(url_for('index'))
            
    # Yangi xabarni yaratish va ro'yxat BOSHIGA qo'shish (insert(0, ...))
    new_entry = {
        "author": author,
        "text": message_text,
        "time": datetime.now().strftime("%H:%M:%S | %d.%m.%Y")
    }
    MESSAGES.insert(0, new_entry)
    
    flash("Xabar muvaffaqiyatli qo'shildi!", "success")
    return redirect(url_for('index'))

# 4. Logout (Sessiyani tozalash)
@app.route('/logout')
def logout():
    session.clear() # Sessiya butunlay tozalanadi
    flash("Siz tizimdan chiqdingiz.", "info")
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
2. Jinja2 Shablonlari (Templates)
Loyiha jildida templates nomli papka oching va ichiga quyidagi ikkita HTML faylni joylashtiring:

templates/index.html (Bosh sahifa)
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <title>Bosh sahifa - Mehmonlar kitobi</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 30px; background-color: #f4f4f4; }
        .container { max-width: 600px; background: white; padding: 20px; border-radius: 8px; }
        .message-box { border-bottom: 1px solid #eee; padding: 10px 0; }
        .meta { font-size: 0.85em; color: #666; }
        .flash { padding: 10px; margin-bottom: 15px; border-radius: 4px; }
        .success { background-color: #d4edda; color: #155724; }
        .danger { background-color: #f8d7da; color: #721c24; }
        .info { background-color: #d1ecf1; color: #0c5460; }
    </style>
</head>
<body>
<div class="container">
    <h2>Mehmonlar kitobi</h2>

    {% with messages = get_flashed_messages(with_categories=true) %}
        {% if messages %}
            {% for category, message in messages %}
                <div class="flash {{ category }}">{{ message }}</div>
            {% endfor %}
        {% endif %}
    {% endwith %}

    {% if session.get('username') %}
        <p>Siz tizimdasiz: <strong>{{ session['username'] }}</strong> | <a href="{{ url_for('logout') }}">Chiqish</a></p>
        
        <form action="{{ url_for('post_message') }}" method="POST">
            <textarea name="text" rows="3" style="width: 100%;" placeholder="Xabaringizni yozing..." required></textarea><br><br>
            <button type="submit">Yuborish</button>
        </form>
    {% else %}
        <p><a href="{{ url_for('login') }}">Tizimga kirish</a> orqali xabar qoldirishingiz mumkin.</p>
    {% endif %}

    <hr>
    <h3>Oxirgi xabarlar (Maksimal 20 ta)</h3>
    
    {% if not messages %}
        <p>Hozircha xabarlar yo'q.</p>
    {% endif %}

    {% for msg in messages %}
        <div class="message-box">
            <strong>{{ msg.author }}</strong>: {{ msg.text }}
            <div class="meta">Vaqt: {{ msg.time }}</div>
        </div>
    {% endfor %}
</div>
</body>
</html>
templates/login.html (Login sahifasi)
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <title>Tizimga kirish</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 50px; background-color: #f4f4f4; }
        .login-card { max-width: 300px; background: white; padding: 25px; border-radius: 8px; margin: 0 auto; }
        .flash { padding: 8px; color: red; margin-bottom: 10px; font-size: 0.9em; }
    </style>
</head>
<body>
<div class="login-card">
    <h3>Tizimga kirish</h3>
    
    {% with messages = get_flashed_messages(with_categories=true) %}
        {% if messages %}
            {% for category, message in messages %}
                <div class="flash">{{ message }}</div>
            {% endfor %}
        {% endif %}
    {% endwith %}

    <form method="POST">
        <p><input type="text" name="username" placeholder="Username (masalan: ali)" required style="width: 90%; padding: 5px;"></p>
        <p><input type="password" name="password" placeholder="Parol (masalan: 123)" required style="width: 90%; padding: 5px;"></p>
        <button type="submit" style="width: 96%; padding: 7px;">Kirish</button>
    </form>
    <br>
    <a href="{{ url_for('index') }}">← Orqaga qaytish</a>
</div>
</body>
</html>
Kod mantiqining muhim jihatlari:
Teskari Xronologiya: MESSAGES.insert(0, new_entry) yangi xabarni doim birinchi o'ringa qo'shadi. recent_messages = MESSAGES[:20] qismi esa ro'yxatdan faqat eng birinchi turgan (eng yangi) 20 ta elementni kesib oladi.

Anti-Spam Blokirovka: /post yo'nalishida MESSAGES[0] (eng oxirgi qo'shilgan xabar) tekshiriladi. Agar uning muallifi va matni hozir yuborilayotgan xabar bilan bir xil bo'lsa, flash xabari qaytarilib, jarayon to'xtatiladi.

Sessiya Tozaligi va Xavfsizlik: session.clear() qilinganda foydalanuvchining identifikatori butunlay o'chadi. Agar u logoutdan keyin sahifani yangilab, eski formani qayta yuborishga (Re-submit) urinsa ham, server /post ichidagi if 'username' not in session: to'sig'idan uni o'tkazmaydi.****
