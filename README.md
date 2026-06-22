# python-flask
Loyiha quyidagi fayllardan tashkil topadi:

app.py — Dastur kodi va retseptlar ma'lumotlar bazasi (dict).

templates/base.html — Asosiy shablon (ota shablon).

templates/index.html — Retseptlar ro'yxati sahifasi.

templates/detail.html — Har bir retseptning batafsil sahifasi.

1. Dastur kodi (app.py)
Python
from flask import Flask, render_template, abort

app = Flask(__name__)

# Kamida 5 ta retseptdan iborat ma'lumotlar (dict)
# Narxi 0 bo'lgan retseptlar shart bajarilishini tekshirish uchun kiritildi
recipes = {
    1: {"title": "Somsa", "description": "O'zbekona tansiq taom, pechda pishiriladi.", "price": 15000, "ingredients": ["Go'sht", "Piyoz", "Xamir", "Dumba yog'i"]},
    2: {"title": "Osh (Palov)", "description": "Toshkentcha bayramona osh retsepti.", "price": 25000, "ingredients": ["Guruch", "Go'sht", "Sabzi", "Piyoz", "Yog'"]},
    3: {"title": "Chuchvara suvi", "description": "Uydagilar uchun foydali va mazali sho'rva tayyorlash usuli.", "price": 0, "ingredients": ["Xamir", "Qiymali go'sht", "Ziravorlar", "Suv"]},
    4: {"title": "Meva salati", "description": "Tez va oson tayyorlanadigan vitaminli salat.", "price": 12000, "ingredients": ["Olma", "Banan", "Apelsin", "Yogurt"]},
    5: {"title": "Muzdek Limonad", "description": "Yoz kunlari uchun tetiklashtiruvchi bepul uy sharoitidagi retsept.", "price": 0, "ingredients": ["Suv", "Limon", "Yalpiz", "Shakar"]}
}

# 1-route: Barcha retseptlar ro'yxati
@app.route('/')
def index():
    return render_template('index.html', recipes=recipes)

# 2-route: Har bir retsept uchun alohida batafsil sahifa
@app.route('/recipes/<int:recipe_id>')
def recipe_detail(recipe_id):
    recipe = recipes.get(recipe_id)
    if recipe is None:
        abort(404)  # Agar retsept topilmasa, 404 xatolik qaytaradi
    return render_template('detail.html', recipe=recipe)

if __name__ == '__main__':
    app.run(debug=True)
2. Ota shablon (templates/base.html)
Barcha sahifalar uchun umumiy bo'lgan dizayn asosi:

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Pazandachilik Sayti{% endblock %}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f9f9f9; }
        .container { max-width: 800px; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        h1 { color: #333; }
        .recipe-card { border-bottom: 1px solid #eee; padding: 15px 0; }
        .badge-free { background-color: #28a745; color: white; padding: 3px 8px; border-radius: 4px; font-size: 0.9em; }
        .price { font-weight: bold; color: #e67e22; }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h2>👨‍🍳 Pazandalik Sirlari</h2>
            <hr>
        </header>

        <main>
            {% block content %}
            {% endblock %}
        </main>
    </div>
</body>
</html>
3. Ro'yxat sahifasi (templates/index.html)
Ushbu sahifada {% for %} tsikli hamda narxni tekshirish uchun {% if %} sharti ishlatilgan:

HTML
{% extends 'base.html' %}

{% block title %}Barcha Retseptlar{% endblock %}

{% block content %}
    <h1>Bizning retseptlar ro'yxati</h1>
    
    <div>
        {% for id, recipe in recipes.items() %}
            <div class="recipe-card">
                <h3><a href="/recipes/{{ id }}">{{ recipe.title }}</a></h3>
                <p>{{ recipe.description }}</p>
                <p>
                    Narxi: 
                    {% if recipe.price == 0 %}
                        <span class="badge-free">Bepul</span>
                    {% else %}
                        <span class="price">{{ recipe.price }} so'm</span>
                    {% endif %}
                </p>
            </div>
        {% endfor %}
    </div>
{% endblock %}
4. Batafsil sahifa (templates/detail.html)
Har bir retseptning tarkibiy mahsulotlarini ko'rsatuvchi sahifa:

HTML
{% extends 'base.html' %}

{% block title %}{{ recipe.title }} Retsepti{% endblock %}

{% block content %}
    <h1>{{ recipe.title }}</h1>
    <p><strong>Tavsif:</strong> {{ recipe.description }}</p>
    
    <p><strong>Narxi:</strong> 
        {% if recipe.price == 0 %}
            <span class="badge-free">Bepul uchrashuv / Maslahat</span>
        {% else %}
            <span class="price">{{ recipe.price }} so'm</span>
        {% endif %}
    </p>

    <h3>Kerakli mahsulotlar:</h3>
    <ul>
        {% for ingredient in recipe.ingredients %}
            <li>{{ ingredient }}</li>
        {% endfor %}
    </ul>

    <p style="margin-top: 30px;">
        <a href="/">← Orqaga (Ro'yxatga qaytish)</a>
    </p>
{% endblock %}
Loyihani ishga tushirish qadamlari:
Loyiha papkasida templates nomli yangi jild (folder) oching.

app.py faylini tashqariga, qolgan 3 ta .html shablonlarini esa templates papkasining ichiga joylashtiring.

Terminalda python app.py buyrug'ini bering va http://127.0.0.1:5000 manzili orqali brauzerda tekshiring.
