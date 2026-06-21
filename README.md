# python-flask
Ilovani Blueprint'lar va create_app() factory pattern asosida modulli strukturaga o'tkazish loyihani kengaytiriladigan va tartibli qiladi.

Quyida loyihaning to'liq strukturasi va barcha kerakli fayllar kodi keltirilgan.

📁 Loyiha strukturasi
Plaintext
loyiha/
│
├── app.py                     # Faqat factory funksiyani chaqiradi
├── myapp/                     # Asosiy paket (package)
│   ├── __init__.py            # create_app() shu yerda bo'ladi
│   │
│   ├── blueprints/
│   │   ├── main.py            # Bosh sahifa va About (main_bp)
│   │   └── notes.py           # CRUD operatsiyalari (notes_bp)
│   │
│   └── templates/             # HTML shablonlar
│       ├── base.html
│       ├── main/
│       │   ├── index.html
│       │   └── about.html
│       └── notes/
│           ├── list.html
│           ├── create.html
│           └── edit.html
💻 Kodlar to'plami
1. myapp/__init__.py (Factory Pattern)
Bu faylda ilovani yaratuvchi va blueprintlarni ro'yxatdan o'tkazuvchi create_app() funksiyasi joylashadi.

Python
from flask import Flask

def create_app():
    app = Flask(__name__)
    app.config['SECRET_KEY'] = 'maxfiy-kalit-shu yerda'

    # Blueprintlarni import qilish
    from myapp.blueprints.main import main_bp
    from myapp.blueprints.notes import notes_bp

    # Blueprintlarni ro'yxatdan o'tkazish
    app.register_blueprint(main_bp)  # Bosh sahifa uchun url_prefix shart emas
    app.register_blueprint(notes_bp, url_prefix='/notes') # CRUD uchun prefiks

    return app
2. myapp/blueprints/main.py (Main Blueprint)
Bosh sahifa va "Biz haqimizda" sahifasi uchun mas'ul kod.

Python
from flask import Blueprint, render_template

main_bp = Blueprint('main', __name__)

@main_bp.route('/')
def index():
    return render_template('main/index.html')

@main_bp.route('/about')
def about():
    return render_template('main/about.html')
3. myapp/blueprints/notes.py (Notes CRUD Blueprint)
Eslatmalar bilan ishlash (CRUD) qismi. Bu yerda url_for ishlatganda notes.index, notes.create kabi murojaat qilinadi.

Python
from flask import Blueprint, render_template, redirect, url_for, request, flash

notes_bp = Blueprint('notes', __name__)

# Namunaviy ma'lumotlar bazasi (vaqtincha ro'yxat)
notes_db = [
    {"id": 1, "title": "Birinchi eslatma", "content": "Flask o'rganish juda qiziq!"}
]

@notes_bp.route('/')
def index():
    return render_template('notes/list.html', notes=notes_db)

@notes_bp.route('/create', methods=['GET', 'POST'])
def create():
    if request.method == 'POST':
        title = request.form.get('title')
        content = request.form.get('content')
        new_id = max([n['id'] for n in notes_db], default=0) + 1
        
        notes_db.append({"id": new_id, "title": title, "content": content})
        flash("Eslatma muvaffaqiyatli yaratildi!", "success")
        return redirect(url_for('notes.index')) # E'tibor bering: notes.index
        
    return render_template('notes/create.html')

@notes_bp.route('/<int:note_id>/edit', methods=['GET', 'POST'])
def edit(note_id):
    note = next((n for n in notes_db if n['id'] == note_id), None)
    if not note:
        flash("Eslatma topilmadi!", "danger")
        return redirect(url_for('notes.index'))

    if request.method == 'POST':
        note['title'] = request.form.get('title')
        note['content'] = request.form.get('content')
        flash("Eslatma yangilandi!", "success")
        return redirect(url_for('notes.index'))

    return render_template('notes/edit.html', note=note)

@notes_bp.route('/<int:note_id>/delete', methods=['POST'])
def delete(note_id):
    global notes_db
    notes_db = [n for n in notes_db if n['id'] != note_id]
    flash("Eslatma o'chirildi!", "warning")
    return redirect(url_for('notes.index'))
4. app.py (Kirish nuqtasi)
Asosiy faylingiz maksimal darajada sodda va qisqa bo'ladi. U faqat factory funksiyani chaqiradi xolos.

Python
from myapp import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
🔗 Shablonlarda url_for ishlatish qoidasi
Endi HTML fayllar ichida havolalar berganda blueprint nomini qo'shib yozasiz:

Bosh sahifaga o'tish: url_for('main.index')

Eslatmalar ro'yxatiga o'tish: url_for('notes.index')

Yangi eslatma yaratish: url_for('notes.create')

Tahrirlash sahifasi (ID bilan): url_for('notes.edit', note_id=note.id)
