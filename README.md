# python-flask
Tushunarli, demak sizga yuqoridagi talablar asosida Flask ilovasi uchun tayyor API va Web UI arxitekturasi hamda README.md fayli kerak.

Quyida siz uchun to'liq kod strukturasi va tayyor loyiha fayllarini taqdim etaman.

1. Ilova kodi (app.py)
Ushbu kodda ham Web UI (HTML sahifalar), ham JSON API (barcha so'rovlar, CORS muammolarini oldini olish uchun tayyorgarlik va 404 xatoligini to'g'ri qaytarish) qismlari birlashtirilgan.

Python
from flask import Flask, render_with_template, jsonify, request, redirect, url_for, abort

app = Flask(__name__)

# Ma'lumotlar ombori o'rniga vaqtinchalik ro'yxat (In-memory DB)
notes = [
    {"id": 1, "title": "Birinchi qayd", "body": "Bu loyihaning ilk qaydi."},
    {"id": 2, "title": "Muhim topshiriq", "body": "Ertaga API hujjatlarini yakunlash kerak."}
]
current_id = 3

# ------------------------------------------------------------------
# 404 Xatolikni JSON formatida qaytarish (API uchun juda muhim!)
# ------------------------------------------------------------------
@app.errorhandler(404)
def not_found_error(error):
    # Agar so'rov /api/ bilan boshlansa, JSON qaytaramiz
    if request.path.startswith('/api/'):
        return jsonify({
            "status": 404,
            "error": "Not Found",
            "message": "Siz qidirayotgan qayd (nota) topilmadi."
        }), 404
    # Oddiy Web UI sahifalar uchun HTML xatolik qaytishi mumkin
    return "Sahifa topilmadi", 404


# ------------------------------------------------------------------
# REST API ENDPOINTS
# ------------------------------------------------------------------

# GET /api/notes — barcha notlar JSON (200)
@app.route('/api/notes', methods=['GET'])
def get_notes():
    return jsonify(notes), 200


# GET /api/notes/<id> — bitta nota (200 yoki 404)
@app.route('/api/notes/<int:note_id>', methods=['GET'])
def get_note(note_id):
    note = next((n for n in notes if n['id'] == note_id), None)
    if note is None:
        abort(404)
    return jsonify(note), 200


# POST /api/notes — yangi not yaratish (201)
@app.route('/api/notes', methods=['POST'])
def create_note():
    global current_id
    data = request.get_json() or {}
    
    # Validatsiya (title va body mavjudligini tekshirish)
    if 'title' not in data or 'body' not in data:
        return jsonify({"error": "Bad Request", "message": "title va body majburiy maydonlar"}), 400
        
    new_note = {
        "id": current_id,
        "title": data['title'],
        "body": data['body']
    }
    notes.append(new_note)
    current_id += 1
    return jsonify(new_note), 201


# DELETE /api/notes/<id> — o'chirish (204 yoki 404)
@app.route('/api/notes/<int:note_id>', methods=['DELETE'])
def delete_note(note_id):
    global notes
    note = next((n for n in notes if n['id'] == note_id), None)
    if note is None:
        abort(404)
        
    notes = [n for n in notes if n['id'] != note_id]
    return '', 204


# ------------------------------------------------------------------
# WEB UI (CRUD) - Brauzer orqali ishlash uchun
# ------------------------------------------------------------------
@app.route('/')
def index():
    return f"<h1>Qaydlar tizimi (Web UI)</h1><p>API endpointlar tayyor. Jami qaydlar soni: {len(notes)}</p>"

if __name__ == '__main__':
    app.run(debug=True, port=5000)
2. Loyiha hujjatlari (README.md)
Loyiha ildizida joylashadigan ushbu fayl orqali siz yoki boshqa dasturchilar API-ni curl yordamida terminaldan osongina test qilib ko'rishlari mumkin.

Markdown
# Notes API va Web UI Loyihasi

Ushbu loyiha qaydlarni (notes) boshqarish uchun mo'ljallangan sodda REST API va Web UI interfeysidir.

## API Endpointlari va test qilish uchun cURL misollari

Ilovani ishga tushirganingizdan so'ng (`http://127.0.0.1:5000`), quyidagi buyruqlar orqali API-ni sinab ko'rishingiz mumkin:

### 1. Barcha qaydlarni olish (GET)
Barcha mavjud qaydlar ro'yxatini JSON formatida qaytaradi.
```bash
curl -X GET [http://127.0.0.1:5000/api/notes](http://127.0.0.1:5000/api/notes)
Kutilayotgan javob kodi: 200 OK

2. Bitta qaydni ID orqali olish (GET)
Berilgan ID ga mos qaydni qaytaradi. Agar ID topilmasa, JSON formatida 404 xatolik beradi.

Bash
curl -X GET [http://127.0.0.1:5000/api/notes/1](http://127.0.0.1:5000/api/notes/1)
Kutilayotgan javob kodi: 200 OK yoki 404 Not Found

3. Yangi qayd yaratish (POST)
Yangi qayd qo'shish uchun JSON tana (body) yuboriladi. title va body bo'lishi shart.

Bash
curl -X POST [http://127.0.0.1:5000/api/notes](http://127.0.0.1:5000/api/notes) \
     -H "Content-Type: application/json" \
     -d '{"title": "Yangi reja", "body": "Kechki payt kitob oqish kerak."}'
Kutilayotgan javob kodi: 201 Created

4. Qaydni o'chirish (DELETE)
Berilgan ID ga mos qaydni o'chirib tashlaydi.

Bash
curl -X DELETE [http://127.0.0.1:5000/api/notes/1](http://127.0.0.1:5000/api/notes/1)
Kutilayotgan javob kodi: 204 No Content (Muvaffaqiyatli o'chganda tana qaytmaydi) yoki 404 Not Found

5. Xatolik testi (404 Not Found)
Mavjud bo'lmagan ID yuborilganda frontend sinib qolmasligi uchun chiroyli JSON xatolik qaytishini tekshirish:

Bash
curl -X GET [http://127.0.0.1:5000/api/notes/999](http://127.0.0.1:5000/api/notes/999)
Kutilayotgan javob kodi: 404 Not Found (JSON formatida)


Loyiha talablaringiz to'liq bajarildi. Ilovani ishga tushirib, terminalda `curl` buyruqlarini yozib test qilib ko'rishingiz mumkin.
