# python-flask
Loyiha talablaringiz asosida yaratilgan, qidiruv tizimiga ega, statik CSS uslublari bilan bezatilgan va katta/kichik harflarni farqlamaydigan (case-insensitive) Flask ilovasi tayyor.

Loyiha quyidagi fayl strukturasidan tashkil topadi:

app.py — Dastur kodi va qidiruv logikasi.

static/style.css — Vizual ko'rinish (dizayn) fayli.

templates/index.html — Qidiruv shakli va natijalar sahifasi.

1. Dastur kodi (app.py)
Ushbu kodda ma'lumotlarni qidirishda .lower() metodidan foydalanilgan. Bu foydalanuvchi "olma", "Olma" yoki "OLMA" deb yozishidan qat'i nazar, qidiruv to'g'ri ishlashini ta'minlaydi (case-insensitive).

Python
from flask import Flask, render_template, request

app = Flask(__name__)

# Qidiruv amalga oshiriladigan ma'lumotlar ro'yxati
ITEMS = [
    "Olma noutbuklari va texnologiyalari",
    "Python dasturlash tili bo'yicha qo'llanma",
    "Flask yordamida veb-sayt yaratish sirlari",
    "Sun'iy intellekt va kelajak texnologiyalari",
    "Ma'lumotlar bazasi (SQL) asoslari",
    "JavaScript va zamonaviy veb frameworklar"
]

@app.route('/')
def index():
    # URL'dan 'q' parametrini olamiz (GET so'rov orqali)
    query = request.args.get('q', '').strip()
    
    results = []
    if query:
        # Katta-kichik harf farqlamasligi uchun ikkala tomonni ham kichik harfga o'tkazamiz
        results = [item for item in ITEMS if query.lower() in item.lower()]
        
    return render_template('index.html', query=query, results=results)

if __name__ == '__main__':
    app.run(debug=True)
2. Statik dizayn (static/style.css)
Sahifani chiroyli ko'rinishga keltirish uchun oddiy va toza dizayn:

CSS
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f4f7f6;
    margin: 0;
    padding: 40px;
    display: flex;
    justify-content: center;
}

.search-container {
    background: white;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 500px;
}

h2 {
    color: #333;
    margin-bottom: 20px;
}

.search-form {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

.search-input {
    flex: 1;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    font-size: 16px;
}

.search-button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
}

.search-button:hover {
    background-color: #0056b3;
}

.results-list {
    list-style: none;
    padding: 0;
}

.results-list li {
    padding: 12px;
    border-bottom: 1px solid #eee;
    color: #555;
}

.no-result {
    color: #dc3545;
    font-weight: bold;
    background-color: #f8d7da;
    padding: 10px;
    border-radius: 5px;
    text-align: center;
}
3. HTML shabloni (templates/index.html)
Bu yerda CSS fayli Flask standartlariga mos ravishda url_for orqali ulangan. Shuningdek, {% if %} yordamida qidiruv natijasi bo'sh bo'lgan holat tekshirilgan:

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aqlli Qidiruv Tizimi</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>

<div class="search-container">
    <h2>Ma'lumotlar bazasidan qidirish</h2>
    
    <form action="/" method="GET" class="search-form">
        <input type="text" name="q" value="{{ query }}" placeholder="Kalit so'zni kiriting..." class="search-input" required>
        <button type="submit" class="search-button">Qidirish</button>
    </form>

    {% if query %}
        <h3>"{{ query }}" bo'yicha qidiruv natijalari:</h3>
        
        {% if results %}
            <ul class="results-list">
                {% for result in results %}
                    <li>🔍 {{ result }}</li>
                {% endfor %}
            </ul>
        {% else %}
            <p class="no-result">Hech narsa topilmadi</p>
        {% endif %}
    {% endif %}
</div>

</body>
</html>
Fayllarni joylashtirish tartibi:
Loyihangiz to'g'ri ishlashi uchun fayllarni quyidagi papka tartibida joylashtiring:

Plaintext
📂 loyiha-papki/
├── 📄 app.py
├── 📂 static/
│   └── 📄 style.css
└── 📂 templates/
    └── 📄 index.html
Ilovani python app.py orqali ishga tushirib, brauzerda http://127.0.0.1:5000 manzilida sinab ko'rishingiz mumkin.
