# python-flask
Loyiha talablariga to'liq javob beradigan, production-ready (Render, Railway yoki PythonAnywhere uchun moslashtirilgan) Flask ilovasini yaratamiz.

Ushbu bosqichda loyihaga .env bilan ishlash, production sozlamalari (ProxyFix, Gunicorn, DEBUG=False), requirements.txt, Procfile va Web UI (HTML interfeys) qismlari qo'shiladi.

Loyiha Strukturasi (Yaxshilangan shakli)
Plaintext
note_api_project/
│
├── app/
│   ├── __init__.py          # App factory + ProxyFix + Config
│   ├── auth.py              # Auth Blueprint
│   ├── notes.py             # Notes Blueprint
│   ├── templates/           # Web UI uchun HTML fayllar
│   │   ├── base.html
│   │   ├── login.html
│   │   └── index.html
│   └── static/              # CSS/JS (ixtiyoriy)
│
├── .env                     # Mahalliy sozlamalar (Maxfiy)
├── .env.example             # .env namunasi (GitHub uchun)
├── .gitignore               # Git-ga qo'shilmaydigan fayllar
├── Procfile                 # Production server (Gunicorn) uchun buyruq
├── requirements.txt         # Kerakli kutubxonalar ro'yxati
├── runtime.txt              # Python versiyasi (PaaS uchun)
├── run.py                   # Mahalliy ishga tushirish fayli
└── README.md                # To'liq hujjatlashtirish
Kod realizatsiyasi
1. app/__init__.py
Production xavfsizligi uchun ProxyFix qo'shildi va sozlamalar os.environ orqali yuklanadi.

Python
import os
from flask import Flask, jsonify, render_template
from werkzeug.middleware.proxy_fix import ProxyFix
from dotenv import load_dotenv

# .env faylini yuklash
load_dotenv()

def create_app():
    app = Flask(__name__)
    
    # .env dan o'qish, agar topilmasa default qiymat (faqat local uchun)
    app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY', 'fallback-local-key-321')
    
    # Production uchun ProxyFix-ni yoqish
    app.wsgi_app = ProxyFix(app.wsgi_app, x_for=1, x_proto=1, x_host=1, x_port=1)
    
    # In-memory DB
    app.users = {1: {"id": 1, "username": "ali"}, 2: {"id": 2, "username": "vali"}}
    app.notes = {
        1: {"id": 1, "title": "Bozorlik", "content": "Sut va non", "user_id": 1},
        2: {"id": 2, "title": "Darslar", "content": "Flask o'rganish", "user_id": 1}
    }
    app.note_id_counter = 3

    # Global Xatoliklar
    @app.errorhandler(401)
    def unauthorized(e): return jsonify({'error': 'Unauthorized'}), 401
    @app.errorhandler(403)
    def forbidden(e): return jsonify({'error': 'Forbidden'}), 403
    @app.errorhandler(404)
    def not_found(e): return jsonify({'error': 'Not Found'}), 404

    # Blueprints
    from app.auth import auth_bp
    from app.notes import notes_bp
    app.register_blueprint(auth_bp, url_prefix='/api/auth')
    app.register_blueprint(notes_bp, url_prefix='/api/notes')

    # Web UI Bosh sahifa (REST API bilan parallel ishlaydi)
    @app.route('/')
    def home():
        return render_template('index.html')

    @app.route('/login')
    def login_page():
        return render_template('login.html')

    return app
2. HTML Shablonlar (Web UI qismi)
Foydalanuvchi brauzer orqali kirganda API bilan fetch orqali muloqot qiladigan sodda Web UI.

app/templates/base.html:

HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Note App</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/water.css@2/out/water.css">
</head>
<body>
    <header>
        <nav><a href="/">Home</a> | <a href="/login">Login</a></nav>
    </header>
    <main>{% block content %}{% endblock %}</main>
</body>
</html>
app/templates/login.html:

HTML
{% extends 'base.html' %}
{% block content %}
<h2>Tizimga kirish</h2>
<input type="text" id="username" placeholder="Username (masalan: ali)">
<button onclick="login()">Kirish</button>

<script>
async function login() {
    const username = document.getElementById('username').value;
    const res = await fetch('/api/auth/login', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({username})
    });
    if (res.ok) window.location.href = '/';
    else alert('Xatolik yuz berdi!');
}
</script>
{% endblock %}
app/templates/index.html:

HTML
{% extends 'base.html' %}
{% block content %}
<h2>Mening eslatmalarim</h2>
<div id="notes-list">Yuklanmoqda...</div>

<hr>
<h3>Yangi eslatma</h3>
<input type="text" id="title" placeholder="Sarlavha"><br>
<textarea id="content" placeholder="Matn"></textarea><br>
<button onclick="addNote()">Qo'shish</button>

<script>
async function loadNotes() {
    const res = await fetch('/api/notes');
    if (res.status === 401) {
        document.getElementById('notes-list').innerHTML = "<p>Iltimos, avval <a href='/login'>login</a> qiling.</p>";
        return;
    }
    const notes = await res.json();
    let html = '';
    notes.forEach(n => {
        html += `<div><h4>${n.title}</h4><p>${n.content}</p></div>`;
    });
    document.getElementById('notes-list').innerHTML = html || '<p>Eslatmalar yo'q</p>';
}

async function addNote() {
    const title = document.getElementById('title').value;
    const content = document.getElementById('content').value;
    await fetch('/api/notes', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({title, content})
    });
    loadNotes();
}
loadNotes();
</script>
{% endblock %}
Configuration & Deployment Fayllari
3. .gitignore
GitHub-ga maxfiy kalitlar va virtual muhit chiqib ketishining oldini oladi.

Plaintext
.env
__pycache__/
*.pyc
.venv/
env/
cookies.txt
4. .env (Lokal uchun)
Plaintext
SECRET_KEY=9f82b3c4d5e6f7a8b9c0e1f2a3b4c5d6
FLASK_DEBUG=True
5. .env.example (GitHub uchun namuna)
Plaintext
SECRET_KEY=your_production_secret_key_here
6. requirements.txt
Plaintext
Flask==3.0.2
gunicorn==21.2.0
python-dotenv==1.0.1
werkzeug==3.0.1
7. Procfile
Render yoki Railway kabi PaaS platformalariga loyihani qanday ishga tushirishni o'rgatadi.

Plaintext
web: gunicorn run:app
8. runtime.txt
Plaintext
python-3.11.5
9. run.py
Python
import os
from app import create_app

app = create_app()

if __name__ == '__main__':
    # Mahalliy muhitda .env dagi FLASK_DEBUG o'qiladi, topilmasa False (Production xavfsizligi)
    debug_mode = os.environ.get('FLASK_DEBUG', 'False').lower() in ['true', '1']
    app.run(debug=debug_mode, host='0.0.0.0', port=5000)
10. README.md (To'liq yo'riqnoma va Hujjat)
Markdown
# Note Sharing Web UI & REST API

Ushbu loyiha eslatmalarni boshqarish uchun xavfsiz REST API va uning Web UI interfeysini taqdim etadi.

**Demo URL:** [https://your-app-name.onrender.com](https://your-app-name.onrender.com) *(O'zingizning demo havolangizni qo'ying)*

---

## Lokal O'rnatish Qadamlari

1. **Repozitoriyani yuklab oling va ichiga kiring:**
   ```bash
   git clone <repo-url>
   cd note_api_project
Virtual muhit yarating va faollashtiring:

Bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows
Kutubxonalarni o'rnating:

Bash
pip install -r requirements.txt
Ekosistema sozlamalarini sozlang:
.env.example faylini .env deb nusxalang va ichidagi SECRET_KEYni o'zgartiring.

Bash
cp .env.example .env
Lokal serverni yoqing:

Bash
python run.py
Endi loyiha brauzerda http://127.0.0.1:5000 manzilida ochiladi.

Production (Render.com) ga Deploy qilish Qadamlari
Loyihani GitHub-ga yuklang (.env fayli yuklanmaganiga ishonch hosil qiling).

Render.com ga kiring va New -> Web Service tanlang.

GitHub repozitoriyangizni ulang.

Quyidagi sozlamalarni kiriting:

Runtime: Python 3

Build Command: pip install -r requirements.txt

Start Command: gunicorn run:app

Advanced -> Environment Variables bo'limiga o'ting va quyidagi o'zgaruvchilarni qo'shing:

SECRET_KEY = juda_uzun_va_murakkab_maxfiy_str

FLASK_DEBUG = False (Avtomatik ravishda production rejimga o'tadi)

Create Web Service tugmasini bosing va deploy tugashini kuting.

API cURL Misollari (Ishlashini tekshirish)
Auth API
Bash
# Login (Session yaratish)
curl -X POST [https://your-app-name.onrender.com/api/auth/login](https://your-app-name.onrender.com/api/auth/login) \
     -H "Content-Type: application/json" \
     -d '{"username": "ali"}' \
     -c cookies.txt

# Me (Joriy foydalanuvchi)
curl -X GET [https://your-app-name.onrender.com/api/auth/me](https://your-app-name.onrender.com/api/auth/me) -b cookies.txt
Notes CRUD API
Bash
# Barcha notalarni ko'rish
curl -X GET [https://your-app-name.onrender.com/api/notes](https://your-app-name.onrender.com/api/notes) -b cookies.txt

# Yangi nota yaratish
curl -X POST [https://your-app-name.onrender.com/api/notes](https://your-app-name.onrender.com/api/notes) \
     -H "Content-Type: application/json" \
     -d '{"title": "Prod test", "content": "Production matni"}' \
     -b cookies.txt
