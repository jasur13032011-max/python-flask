# python-flask
Mana yuqorida ko'rsatilgan barcha talablarga (Factory Pattern, Blueprints, Session-based Auth, Notes CRUD, JSON xatoliklar va curl misollari) to'liq javob beradigan toza va professional Flask loyihasi strukturasi hamda kodi.

Loyiha Strukturasi (Project Structure)
Plaintext
note_api_project/
│
├── app/
│   ├── __init__.py          # Flask app factory (create_app)
│   ├── auth.py              # Login, logout va me endpointlari (auth_bp)
│   └── notes.py             # Notes CRUD endpointlari (notes_bp)
│
├── README.md                # Loyihani ishga tushirish va curl misollari
└── run.py                   # Loyihani ishga tushiruvchi asosiy fayl
Kod realizatsiyasi
1. app/__init__.py (Application Factory)
Ushbu faylda create_app() funksiyasi yaratiladi, xatoliklar JSON formatiga keltiriladi va Blueprint'lar ro'yxatdan o'tkaziladi.

Python
from flask import Flask, jsonify

def create_app():
    app = Flask(__name__)
    
    # Session ishlashi uchun secret key (haqiqiy loyihada .env dan olinadi)
    app.config['SECRET_KEY'] = 'super-secret-key-12345'
    
    # In-memory "ma'lumotlar bazasi" simulyatsiyasi
    app.users = {
        1: {"id": 1, "username": "ali"},
        2: {"id": 2, "username": "vali"}
    }
    app.notes = {
        1: {"id": 1, "title": "Bozorlik", "content": "Sut va non olish kerak", "user_id": 1},
        2: {"id": 2, "title": "Darslar", "content": "Flask o'rganish", "user_id": 1},
        3: {"id": 3, "title": "Kino", "content": "Yangi kinoni ko'rish", "user_id": 2}
    }
    app.note_id_counter = 4

    # Global xatoliklarni JSON formatiga o'tkazish
    @app.errorhandler(400)
    def bad_request(e):
        return jsonify({'error': 'Bad Request'}), 400

    @app.errorhandler(401)
    def unauthorized(e):
        return jsonify({'error': 'Unauthorized'}), 401

    @app.errorhandler(403)
    def forbidden(e):
        return jsonify({'error': 'Forbidden'}), 403

    @app.errorhandler(404)
    def not_found(e):
        return jsonify({'error': 'Not Found'}), 404

    @app.errorhandler(405)
    def method_not_allowed(e):
        return jsonify({'error': 'Method Not Allowed'}), 405

    # Blueprint'larni import qilish va ro'yxatdan o'tkazish
    from app.auth import auth_bp
    from app.notes import notes_bp

    app.register_blueprint(auth_bp, url_prefix='/api/auth')
    app.register_blueprint(notes_bp, url_prefix='/api/notes')

    return app
2. app/auth.py (Authentication Blueprint)
Python
from flask import Blueprint, request, jsonify, session, current_app, abort

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/login', methods=['POST'])
def login():
    data = request.get_json() or {}
    username = data.get('username')
    
    if not username:
        return jsonify({'error': 'Username is required'}), 400
        
    # Foydalanuvchini qidirish
    user = None
    for u_id, u_data in current_app.users.items():
        if u_data['username'] == username:
            user = u_data
            break
            
    if not user:
        return jsonify({'error': 'User not found'}), 401
        
    # Sessionga saqlash
    session['user_id'] = user['id']
    return jsonify(user), 200

@auth_bp.route('/logout', methods=['POST'])
def logout():
    session.pop('user_id', None)
    return '', 204

@auth_bp.route('/me', methods=['GET'])
def me():
    user_id = session.get('user_id')
    if not user_id:
        abort(401)
        
    user = current_app.users.get(user_id)
    return jsonify(user), 200
3. app/notes.py (Notes CRUD Blueprint)
Python
from flask import Blueprint, request, jsonify, session, current_app, abort

notes_bp = Blueprint('notes', __name__)

# Login bo'lganini tekshirish uchun yordamchi funksiya
def get_current_user_id():
    user_id = session.get('user_id')
    if not user_id:
        abort(401)
    return user_id

@notes_bp.route('', methods=['GET'])
def get_notes():
    user_id = get_current_user_id()
    
    # Faqat joriy foydalanuvchiga tegishli notalarni filterlash
    user_notes = [note for note in current_app.notes.values() if note['user_id'] == user_id]
    return jsonify(user_notes), 200

@notes_bp.route('', methods=['POST'])
def create_note():
    user_id = get_current_user_id()
    data = request.get_json() or {}
    
    title = data.get('title')
    content = data.get('content')
    
    if not title or not content:
        return jsonify({'error': 'Title and content are required'}), 400
        
    new_id = current_app.note_id_counter
    new_note = {
        "id": new_id,
        "title": title,
        "content": content,
        "user_id": user_id
    }
    
    current_app.notes[new_id] = new_note
    current_app.note_id_counter += 1
    
    return jsonify(new_note), 201

@notes_bp.route('/<int:note_id>', methods=['GET', 'PUT', 'DELETE'])
def note_detail(note_id):
    user_id = get_current_user_id()
    note = current_app.notes.get(note_id)
    
    if not note:
        abort(404)
        
    # Boshqa foydalanuvchining notasiga ruxsat bermaslik
    if note['user_id'] != user_id:
        abort(403)
        
    if request.method == 'GET':
        return jsonify(note), 200
        
    elif request.method == 'PUT':
        data = request.get_json() or {}
        title = data.get('title')
        content = data.get('content')
        
        if not title or not content:
            return jsonify({'error': 'Title and content are required'}), 400
            
        note['title'] = title
        note['content'] = content
        return jsonify(note), 200
        
    elif request.method == 'DELETE':
        del current_app.notes[note_id]
        return '', 204
4. run.py
Loyihani ishga tushirish uchun asosiy kirish nuqtasi.

Python
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True, port=5000)
5. README.md (Ishga tushirish va curl hujjatlari)
Ushbu fayl API bilan qanday ishlashni va har bir endpoint uchun curl buyruqlarini o'z ichiga oladi. Session cookie faylini saqlash va yuborish uchun curl dagi -c cookies.txt va -b cookies.txt flaglaridan foydalaniladi.

Markdown
# Note Sharing JSON API

Bu session-based autentifikatsiyaga ega bo'lgan eslatmalar (notes) boshqaruv API tizimi.

## Ishga tushirish

1. Kutubxonalarni o'rnating:
```bash
pip install flask
Serverni yoqing:

Bash
python run.py
API Endpoints va curl Misollari
1. Auth Blueprint (/api/auth/...)
🔐 Login (POST)
Foydalanuvchi sifatida tizimga kirish va cookielarni saqlash.

Bash
curl -X POST [http://127.0.0.1:5000/api/auth/login](http://127.0.0.1:5000/api/auth/login) \
     -H "Content-Type: application/json" \
     -d '{"username": "ali"}' \
     -c cookies.txt
👤 Joriy foydalanuvchi (GET)
Kirgan foydalanuvchi ma'lumotlarini olish (Cookie talab qilinadi).

Bash
curl -X GET [http://127.0.0.1:5000/api/auth/me](http://127.0.0.1:5000/api/auth/me) \
     -b cookies.txt
🚪 Logout (POST)
Tizimdan chiqish (Sessionni o'chirish).

Bash
curl -X POST [http://127.0.0.1:5000/api/auth/logout](http://127.0.0.1:5000/api/auth/logout) \
     -b cookies.txt
2. Notes Blueprint (/api/notes/...)
📝 Barcha eslatmalarni olish (GET)
Faqat tizimga kirgan foydalanuvchining eslatmalarini qaytaradi.

Bash
curl -X GET [http://127.0.0.1:5000/api/notes](http://127.0.0.1:5000/api/notes) \
     -b cookies.txt
➕ Yangi eslatma yaratish (POST)
Bash
curl -X POST [http://127.0.0.1:5000/api/notes](http://127.0.0.1:5000/api/notes) \
     -H "Content-Type: application/json" \
     -d '{"title": "Yangi Nota", "content": "Bu test eslatma mazmuni"}' \
     -b cookies.txt
📖 Aniq bir eslatmani ko'rish (GET)
Bash
curl -X GET [http://127.0.0.1:5000/api/notes/1](http://127.0.0.1:5000/api/notes/1) \
     -b cookies.txt
✏️ Eslatmani to'liq yangilash (PUT)
Bash
curl -X PUT [http://127.0.0.1:5000/api/notes/1](http://127.0.0.1:5000/api/notes/1) \
     -H "Content-Type: application/json" \
     -d '{"title": "Yangilangan Sarlavha", "content": "Yangilangan matn"}' \
     -b cookies.txt
❌ Eslatmani o'chirish (DELETE)
Bash
curl -X DELETE [http://127.0.0.1:5000/api/notes/1](http://127.0.0.1:5000/api/notes/1) \
     -b cookies.txt
Xatolik Holatlari (JSON Error Responses)
Agar noto'g'ri ID berilsa yoki login qilmasdan so'rov yuborilsa, tizim quyidagicha toza JSON javob qaytaradi:

401 Unauthorized: {"error": "Unauthorized"}

403 Forbidden: {"error": "Forbidden"} (Boshqa odamning notasini o'chirmoqchi yoki ko'rmoqchi bo'lganda)

404 Not Found: {"error": "Not Found"}
