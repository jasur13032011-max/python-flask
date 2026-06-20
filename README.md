# python-flask
Mana Flask va Flask-SQLAlchemy yordamida to'liq CRUD (Yaratish, O'qish, Yangilash, O'chirish) amallarini va PRG (Post/Redirect/Get) patternini o'z ichiga olgan mukammal namuna.

Siz so'ragan barcha shartlar (forma ko'rsatish, postni qabul qilish, flash xabarlari, redirect va 404 xatoligi) ushbu kodda jamlangan.

1. Flask Ilovasi Kodi (app.py)
Python
from flask import Flask, render_template, request, redirect, url_for, flash, abort
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.exc import IntegrityError

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///notes.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'super-secret-key-123'  # flash() xabarlari uchun shart

db = SQLAlchemy(app)

# --- Model ---
class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)

    def __repr__(self):
        return f'<Note {self.title}>'

# Bazani yaratish (Ilova ishga tushganda avtomatik yaratiladi)
with app.app_context():
    db.create_all()


# --- Routes ---

# 1. READ: Bosh sahifada barcha qaydlarni ko'rsatish
@app.route('/')
def index():
    notes = Note.query.all()
    return render_template('index.html', notes=notes)


# 2. CREATE: Yangi qayd yaratish (GET forma + POST saqlash)
@app.route('/notes/new', methods=['GET', 'POST'])
def create_note():
    if request.method == 'POST':
        title = request.form.get('title')
        content = request.form.get('content')
        
        if not title or not content:
            flash("Sarlavha va kontent bo'sh bo'lishi mumkin emas!", "error")
            return redirect(url_for('create_note'))
            
        new_note = Note(title=title, content=content)
        db.session.add(new_note)
        
        try:
            db.session.commit()
            flash("Qayd muvaffaqiyatli yaratildi!", "success")
            return redirect(url_for('index'))  # PRG Pattern: redirect qilinadi
        except IntegrityError:
            db.session.rollback()
            flash("Tizimda xatolik yuz berdi.", "error")
            return redirect(url_for('create_note'))
            
    return render_template('create.html')


# 3. READ DETALI: Bitta qayd tafsiloti (Mavjud bo'lmasa 404)
@app.route('/notes/<int:id>')
def note_detail(id):
    # .get_or_404() avtomatik ravishda obyekt topilmasa 404 xatolik qaytaradi
    note = Note.query.get_or_404(id)
    return render_template('detail.html', note=note)


# 4. UPDATE: Qaydni tahrirlash (GET forma + POST yangilash)
@app.route('/notes/<int:id>/edit', methods=['GET', 'POST'])
def edit_note(id):
    note = Note.query.get_or_404(id)
    
    if request.method == 'POST':
        note.title = request.form.get('title')
        note.content = request.form.get('content')
        
        try:
            db.session.commit()
            flash("Qayd muvaffaqiyatli yangilandi!", "success")
            return redirect(url_for('note_detail', id=note.id))  # PRG Pattern
        except IntegrityError:
            db.session.rollback()
            flash("Yangilashda xatolik yuz berdi.", "error")
            return redirect(url_for('edit_note', id=note.id))
            
    return render_template('edit.html', note=note)


# 5. DELETE: Qaydni o'chirish (Faqat POST xavfsizligi uchun)
@app.route('/notes/<int:id>/delete', methods=['POST'])
def delete_note(id):
    note = Note.query.get_or_404(id)
    db.session.delete(note)
    
    try:
        db.session.commit()
        flash("Qayd muvaffaqiyatli o'chirildi!", "success")
    except IntegrityError:
        db.session.rollback()
        flash("O'chirishda xatolik yuz berdi.", "error")
        
    return redirect(url_for('index'))  # PRG Pattern


if __name__ == '__main__':
    app.run(debug=True)
2. HTML Shablonlar (templates/)
Ilova chiroyli ishlashi va flash() xabarlarini ko'rsatishi uchun templates nomli papka ochib, ichiga quyidagi fayllarni joylashtiring.

templates/base.html (Asosiy qolip)
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <title>Qaydlar Ilovasi</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background: #f4f4f4; }
        .container { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        .flash { padding: 10px; margin-bottom: 15px; border-radius: 4px; }
        .success { background-color: #d4edda; color: #155724; }
        .error { background-color: #f8d7da; color: #721c24; }
        .note-item { border-bottom: 1px solid #ddd; padding: 10px 0; }
        .btn { display: inline-block; padding: 8px 12px; background: #007bff; color: white; text-decoration: none; border-radius: 4px; border: none; cursor: pointer; }
        .btn-danger { background: #dc3545; }
        .btn-secondary { background: #6c757d; }
    </style>
</head>
<body>
    <div class="container">
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="flash {{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}

        {% block content %} {% endblock %}
    </div>
</body>
</html>
templates/index.html (Bosh sahifa)
HTML
{% extends 'base.html' %}
{% block content %}
    <h1>Mening Qaydlarim</h1>
    <a href="{{ url_for('create_note') }}" class="btn">+ Yangi Qayd Qo'shish</a>
    <hr>
    {% if notes %}
        {% for note in notes %}
            <div class="note-item">
                <h3><a href="{{ url_for('note_detail', id=note.id) }}">{{ note.title }}</a></h3>
                <p>{{ note.content[:100] }}...</p>
            </div>
        {% endfor %}
    {% else %}
        <p>Hozircha hech qanday qayd yo'q.</p>
    {% endif %}
{% endblock %}
templates/create.html (Yaratish formasi)
HTML
{% extends 'base.html' %}
{% block content %}
    <h1>Yangi Qayd Yaratish</h1>
    <form method="POST">
        <label>Sarlavha:</label><br>
        <input type="text" name="title" style="width: 100%; padding: 8px;" required><br><br>
        
        <label>Kontent:</label><br>
        <textarea name="content" rows="5" style="width: 100%; padding: 8px;" required></textarea><br><br>
        
        <button type="submit" class="btn">Saqlash</button>
        <a href="{{ url_for('index') }}" class="btn btn-secondary">Orqaga</a>
    </form>
{% endblock %}
templates/detail.html (Bitta qayd detali)
HTML
{% extends 'base.html' %}
{% block content %}
    <h1>{{ note.title }}</h1>
    <p style="white-space: pre-wrap;">{{ note.content }}</p>
    <hr>
    <a href="{{ url_for('index') }}" class="btn btn-secondary">Bosh sahifa</a>
    <a href="{{ url_for('edit_note', id=note.id) }}" class="btn">Tahrirlash</a>
    
    <form action="{{ url_for('delete_note', id=note.id) }}" method="POST" style="display: inline;" onsubmit="return confirm('Rostdan ham o\'chirmoqchimisiz?');">
        <button type="submit" class="btn btn-danger">O'chirish</button>
    </form>
{% endblock %}
templates/edit.html (Tahrirlash formasi)
HTML
{% extends 'base.html' %}
{% block content %}
    <h1>Qaydni Tahrirlash</h1>
    <form method="POST">
        <label>Sarlavha:</label><br>
        <input type="text" name="title" value="{{ note.title }}" style="width: 100%; padding: 8px;" required><br><br>
        
        <label>Kontent:</label><br>
        <textarea name="content" rows="5" style="width: 100%; padding: 8px;" required>{{ note.content }}</textarea><br><br>
        
        <button type="submit" class="btn">Yangilash</button>
        <a href="{{ url_for('note_detail', id=note.id) }}" class="btn btn-secondary">Bekor qilish</a>
    </form>
{% endblock %}
Nima uchun ushbu tuzilma ideal?
PRG Pattern (Post/Redirect/Get) to'liq ta'minlangan: Formadan ma'lumot POST bo'lib kelgandan keyin sahifa qayta render qilinmaydi, balki redirect() yordamida boshqa sahifaga yo'naltiriladi. Bu foydalanuvchi sahifani yangilaganida (F5 bosganda) ma'lumotlar bazaga ikki marta yozilib qolishini oldini oladi.

get_or_404(id) ishlatilgan: Agar foydalanuvchi brauzerda /notes/999 deb mavjud bo'lmagan ID kiritib kirsa, Flask avtomatik ravishda chiroyli 404 Not Found xatoligini qaytaradi.

Xavfsiz O'chirish (Delete): O'chirish amali oddiy <a> havola (GET) orqali emas, balki POST so'rovi yuboradigan forma orqali bajarilgan. Bu tasodifiy yoki qidiruv botlari havola orqali o'tib ma'lumotlarni o'chirib yuborishini oldini oladi.
