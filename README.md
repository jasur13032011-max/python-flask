# python-flaskMana so'ralgan barcha shartlarni (User va Note munosabati, One-to-Many bog'lanish, xavfsizlik tekshiruvlari (abort(403)), sessiyalar va PRG pattern) o'z ichiga olgan to'liq Flask ilovasi.

1. Flask Ilovasi Kodi (app.py)
Python
from datetime import datetime
from flask import Flask, render_template, request, redirect, url_for, flash, session, abort
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.exc import IntegrityError

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///notes_app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'dev-secret-key-98765'

db = SQLAlchemy(app)

# --- MODELLAR ---

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    
    # cascade="all, delete-orphan" -> User o'chganda unga tegishli barcha Note'lar ham o'chib ketadi
    notes = db.relationship('Note', backref='author', lazy=True, cascade="all, delete-orphan")

class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    body = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Tashqi kalit (ForeignKey)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)


# Ma'lumotlar bazasini yaratish
with app.app_context():
    db.create_all()


# --- DEKORATOR (Avtorizatsiyani tekshirish uchun) ---
def get_current_user_id():
    return session.get('user_id')


# --- ROUTES ---

# 1. BOSH SAHIFA / LOGIN
@app.route('/', methods=['GET', 'POST'])
def login():
    if get_current_user_id():
        return redirect(url_for('list_notes'))
        
    if request.method == 'POST':
        username = request.form.get('username', '').strip()
        
        if not username:
            flash("Username bo'sh bo'lishi mumkin emas!", "error")
            return redirect(url_for('login'))
            
        # Foydalanuvchini bazadan qidiramiz
        user = User.query.filter_by(username=username).first()
        
        # Agar mavjud bo'lmasa, yangi yaratamiz
        if not user:
            user = User(username=username)
            db.session.add(user)
            try:
                db.session.commit()
                flash(f"Yangi profil yaratildi: @{username}", "success")
            except IntegrityError:
                db.session.rollback()
                flash("Xatolik yuz berdi, qaytadan urunib ko'ring.", "error")
                return redirect(url_for('login'))
        else:
            flash(f"Xush kelibsiz, @{username}!", "success")
            
        # Sessiyaga ID ni yozib qo'yamiz
        session['user_id'] = user.id
        return redirect(url_for('list_notes')) # PRG
        
    return render_template('login.html')


# 2. LOGOUT
@app.route('/logout')
def logout():
    session.clear()
    flash("Tizimdan muvaffaqiyatli chiqdingiz.", "success")
    return redirect(url_for('login'))


# 3. READ: Faqat joriy foydalanuvchining qaydlari
@app.route('/notes')
def list_notes():
    user_id = get_current_user_id()
    if not user_id:
        flash("Avval tizimga kiring!", "error")
        return redirect(url_for('login'))
        
    user = User.query.get(user_id)
    # Faqat shu user_id ga tegishli qaydlarni filter qilamiz
    user_notes = Note.query.filter_by(user_id=user_id).order_by(Note.created_at.desc()).all()
    
    return render_template('notes.html', user=user, notes=user_notes)


# 4. CREATE: Yangi qayd qo'shish
@app.route('/notes/new', methods=['GET', 'POST'])
def create_note():
    user_id = get_current_user_id()
    if not user_id:
        abort(401) # Unauthorized
        
    if request.method == 'POST':
        title = request.form.get('title')
        body = request.form.get('body')
        
        if not title or not body:
            flash("Sarlavha va matn to'ldirilishi shart!", "error")
            return redirect(url_for('create_note'))
            
        new_note = Note(title=title, body=body, user_id=user_id)
        db.session.add(new_note)
        db.session.commit()
        
        flash("Qayd muvaffaqiyatli saqlandi!", "success")
        return redirect(url_for('list_notes')) # PRG
        
    return render_template('create_note.html')


# 5. UPDATE: Qaydni tahrirlash (Xavfsizlik tekshiruvi bilan)
@app.route('/notes/<int:id>/edit', methods=['GET', 'POST'])
def edit_note(id):
    user_id = get_current_user_id()
    if not user_id:
        abort(401)
        
    note = Note.query.get_or_404(id)
    
    # MUHIM XAVFSIZLIK TEKSHIRUVI: Qayd qonuniy egasiga tegishlimi?
    if note.user_id != user_id:
        abort(403) # Forbidden (Taqiqlangan)
        
    if request.method == 'POST':
        note.title = request.form.get('title')
        note.body = request.form.get('body')
        
        db.session.commit()
        flash("Qayd muvaffaqiyatli yangilandi!", "success")
        return redirect(url_for('list_notes')) # PRG
        
    return render_template('edit_note.html', note=note)


# 6. DELETE: Qaydni o'chirish (Xavfsizlik tekshiruvi bilan)
@app.route('/notes/<int:id>/delete', methods=['POST'])
def delete_note(id):
    user_id = get_current_user_id()
    if not user_id:
        abort(401)
        
    note = Note.query.get_or_404(id)
    
    # MUHIM XAVFSIZLIK TEKSHIRUVI: Qayd qonuniy egasiga tegishlimi?
    if note.user_id != user_id:
        abort(403)
        
    db.session.delete(note)
    db.session.commit()
    
    flash("Qayd o'chirib tashlandi!", "success")
    return redirect(url_for('list_notes')) # PRG


if __name__ == '__main__':
    app.run(debug=True)
2. HTML Shablonlar (templates/)
templates/base.html
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <title>Sessiyalar va Qaydlar</title>
    <style>
        body { font-family: sans-serif; margin: 30px; background: #eee; }
        .box { background: white; padding: 20px; border-radius: 6px; max-width: 600px; margin: auto; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .flash { padding: 10px; margin-bottom: 10px; border-radius: 4px; }
        .success { background: #d4edda; color: #155724; }
        .error { background: #f8d7da; color: #721c24; }
        .note { border: 1px solid #ddd; padding: 10px; margin: 10px 0; border-radius: 4px; }
        .btn { padding: 6px 12px; text-decoration: none; border-radius: 4px; display: inline-block; cursor: pointer; border: none;}
        .btn-primary { background: #007bff; color: white; }
        .btn-danger { background: #dc3545; color: white; }
        .btn-link { background: none; color: #007bff; padding: 0; }
    </style>
</head>
<body>
    <div class="box">
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, msg in messages %}
                    <div class="flash {{ category }}">{{ msg }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}

        {% block content %} {% endblock %}
    </div>
</body>
</html>
templates/login.html
HTML
{% extends 'base.html' %}
{% block content %}
    <h2>Tizimga kirish / Ro'yxatdan o'tish</h2>
    <form method="POST">
        <label>Username kiriting:</label><br><br>
        <input type="text" name="username" placeholder="Masalan: alisher" style="width: 95%; padding: 8px;" required><br><br>
        <button type="submit" class="btn btn-primary">Kirish</button>
    </form>
{% endblock %}
templates/notes.html
HTML
{% extends 'base.html' %}
{% block content %}
    <div style="display: flex; justify-content: space-between; align-items: center;">
        <h3>@{{ user.username }} sahifasi</h3>
        <a href="{{ url_for('logout') }}" style="color: red;">Chiqish</a>
    </div>
    <hr>
    <a href="{{ url_for('create_note') }}" class="btn btn-primary">+ Yangi qayd yozish</a>
    
    <h4>Sizning qaydlaringiz:</h4>
    {% if notes %}
        {% for note in notes %}
            <div class="note">
                <h5>{{ note.title }} <small style="color: gray;">({{ note.created_at.strftime('%Y-%m-%d %H:%M') }})</small></h5>
                <p>{{ note.body }}</p>
                <a href="{{ url_for('edit_note', id=note.id) }}" class="btn-link">Tahrirlash</a> | 
                
                <form action="{{ url_for('delete_note', id=note.id) }}" method="POST" style="display:inline;" onsubmit="return confirm('O\'chirilsinmi?');">
                    <button type="submit" class="btn-link" style="color: red;">O'chirish</button>
                </form>
            </div>
        {% endfor %}
    {% else %}
        <p>Hozircha qaydlar yo'q.</p>
    {% endif %}
{% endblock %}
templates/create_note.html
HTML
{% extends 'base.html' %}
{% block content %}
    <h3>Yangi qayd qo'shish</h3>
    <form method="POST">
        <input type="text" name="title" placeholder="Sarlavha" style="width: 95%; padding: 8px;" required><br><br>
        <textarea name="body" placeholder="Matn..." rows="5" style="width: 95%; padding: 8px;" required></textarea><br><br>
        <button type="submit" class="btn btn-primary">Saqlash</button>
        <a href="{{ url_for('list_notes') }}">Orqaga</a>
    </form>
{% endblock %}
templates/edit_note.html
HTML
{% extends 'base.html' %}
{% block content %}
    <h3>Qaydni tahrirlash</h3>
    <form method="POST">
        <input type="text" name="title" value="{{ note.title }}" style="width: 95%; padding: 8px;" required><br><br>
        <textarea name="body" rows="5" style="width: 95%; padding: 8px;" required>{{ note.body }}</textarea><br><br>
        <button type="submit" class="btn btn-primary">Yangilash</button>
        <a href="{{ url_for('list_notes') }}">Bekor qilish</a>
    </form>
{% endblock %}
Diqqat qilinishi kerak bo'lgan asosiy nuqtalar:
cascade="all, delete-orphan": Bu munosabat (relationship) o'rnatilganda juda muhim. Agar tizimdan biror User o'chirib tashlansa, unga tegishli bo'lgan barcha Notelar bazada yetim (orphan) bo'lib qolmasdan, avtomatik ravishda tozalab tashlanadi.

Xavfsizlik Devori (abort(403)): Agar biron bir ayyor foydalanuvchi tizimga kirib, brauzer manzillar panelida qo'lda boshqa birovning qayd ID sini tahrirlashga urinsa (masalan: /notes/5/edit), kod darhol note.user_id != session['user_id'] shartini tekshiradi va unga 403 Forbidden xatoligini qaytarib, o'zgartirishga yo'l qo'ymaydi.
