# python-flask
Siz aytgan barcha shartlar va xavfsizlik qoidalariga rioya qilgan holda, Python va Flask freymvorkida yozilgan to'liq kod namunasini taqdim etaman.

secret_key xavfsizligini ta'minlash uchun uni GitHub'ga yuklanmaydigan .env faylida saqlaymiz va kod ichiga os.environ orqali yuklab olamiz.

1. .env faylini yaratish
Loyiha jildida .env nomli fayl yarating va uning ichiga maxfiy kalitni yozing:

Code snippet
FLASK_SECRET_KEY=boshqalardan_sir_tutiladigan_juda_uzun_va_murakkab_kalit_12345
(Bu faylni .gitignore ichiga qo'shishni unutmang, shunda u GitHub'ga chiqib ketmaydi).

2. app.py — Asosiy dastur kodi
Ushbu kodda sessiya bilan ishlash, yo'naltirish (redirect) va login holatini tekshirish mantiqlari to'liq qamrab olingan. Ishga tushirishdan oldin pip install flask python-dotenv buyrug'i orqali kerakli kutubxonalarni o'rnatib oling.

Python
import os
from flask import Flask, render_template_string, request, redirect, url_for, session
from dotenv import load_dotenv

# .env faylidan o'zgaruvchilarni yuklash
load_dotenv()

app = Flask(__name__)

# Secret keyni muhit o'zgaruvchilaridan olish
app.secret_key = os.environ.get('FLASK_SECRET_KEY')

# Oddiy test uchun foydalanuvchilar bazasi (namuna sifatida)
USERS = {
    "admin": "password123",
    "user1": "mypassword"
}

# 1. Login sahifasi (GET va POST)
@app.route('/login', methods=['GET', 'POST'])
def login():
    # Agar foydalanuvchi allaqachon login bo'lgan bo'lsa, dashboardga o'tkazish
    if 'username' in session:
        return redirect(url_for('dashboard'))
        
    error = None
    if request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        
        # Logika va parolni tekshirish
        if username in USERS and USERS[username] == password:
            session['username'] = username  # Sessionda username saqlash
            return redirect(url_for('dashboard'))
        else:
            error = "Logindagi xatolik! Username yoki parol noto'g'ri."
            
    # Oddiy HTML interfeys (ko'rgazmali bo'lishi uchun render_template_string ishlatildi)
    return render_template_string('''
        <h2>Tizimga kirish</h2>
        {% if error %}<p style="color: red;">{{ error }}</p>{% endif %}
        <form method="post">
            <p><input type="text" name="username" placeholder="Username" required></p>
            <p><input type="password" name="password" placeholder="Parol" required></p>
            <p><button type="submit">Kirish</button></p>
        </form>
    ''', error=error)

# 2. Dashboard sahifasi (Faqat login bo'lganlar uchun)
@app.route('/dashboard')
def dashboard():
    # Login'siz kirmoqchi bo'lsa — /login ga redirect qilish
    if 'username' not in session:
        return redirect(url_for('login'))
        
    # Sessiondan usernameni olib dashboardda ko'rsatish
    current_user = session['username']
    return render_template_string('''
        <h2>Dashboard (Shaxsiy kabinet)</h2>
        <p>Xush kelibsiz, <strong>{{ username }}</strong>!</p>
        <p>Bu sahifani faqat tizimga kirganlargina ko'ra oladi.</p>
        <a href="{{ url_for('logout') }}">Chiqish (Logout)</a>
    ''', username=current_user)

# 3. Logout sahifasi
@app.route('/logout')
def logout():
    # Sessiondan usernameni o'chirish (tizimdan chiqarish)
    session.pop('username', None)
    # Logoutdan keyin dashboard yopiq qoladi va foydalanuvchi login sahifasiga qaytadi
    return redirect(url_for('login'))

if __name__ == '__main__':
    app.run(debug=True)
Kod qanday ishlaydi?
Xavfsizlik: os.environ.get('FLASK_SECRET_KEY') yordamida maxfiy kalit o'qiladi. Bu kalit brauzerdagi kukilarni (cookies) shifrlash uchun zarur.

Himoya (Redirect): /dashboard yo'nalishida if 'username' not in session: tekshiruvi bor. Agar foydalanuvchi login qilmasdan to'g'ridan-to'g'ri shu havolaga kirmoqchi bo'lsa, u darhol /login sahifasiga qaytarib yuboriladi.

Logout mantiqi: /logoutga bosilganda session.pop('username') orqali session ichidagi ma'lumot tozalab tashlanadi. Buning natijasida brauzerda qayta /dashboardga kirishga urinish muvaffaqiyatsiz tugaydi va yana login so'raladi.
