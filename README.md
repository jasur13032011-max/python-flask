# python-flask
Talablaringiz asosida yaratilgan, kamida 3 ta routeni o'z ichiga olgan, dinamik parametrli va url_for() yordamida o'zaro bog'langan Flask loyihasi tayyor.

Loyihani oson sinab ko'rish uchun quyidagi fayl strukturasidan foydalanishingiz mumkin:

1. Dastur kodi (app.py)
Ushbu kodda 3 ta route va sahifalar orasida navigatsiya qilish uchun url_for() funksiyasidan foydalanilgan. Shuningdek, dinamik <name> parametri ham kiritilgan.

Python
from flask import Flask, url_for

app = Flask(__name__)

# 1-route: Bosh sahifa
@app.route('/')
def home():
    # url_for orqali boshqa sahifalarga link yaratamiz
    about_url = url_for('about')
    greet_url = url_for('greet', name='Ali')
    
    html = f"""
    <h1>Bosh sahifa</h1>
    <p>Flask ilovasiga xush kelibsiz!</p>
    <ul>
        <li><a href="{about_url}">Biz haqimizda sahifasiga o'tish</a></li>
        <li><a href="{greet_url}">Ali bilan salomlashish sahifasiga o'tish (Dinamik)</a></li>
    </ul>
    """
    return html

# 2-route: Biz haqimizda sahifasi
@app.route('/about')
def about():
    home_url = url_for('home')
    html = f"""
    <h1>Biz haqimizda</h1>
    <p>Ushbu loyiha Flask routerlarini va url_for() funksiyasini tushunish uchun yaratildi.</p>
    <a href="{home_url}">Bosh sahifaga qaytish</a>
    """
    return html

# 3-route: Dinamik <name> parametri bilan salomlashish sahifasi
@app.route('/user/<name>')
def greet(name):
    home_url = url_for('home')
    html = f"""
    <h1>Salom, {name}!</h1>
    <p>Sizning ismingiz URL orqali dinamik tarzda qabul qilindi.</p>
    <a href="{home_url}">Bosh sahifaga qaytish</a>
    """
    return html

if __name__ == '__main__':
    app.run(debug=True)
2. Loyiha qo'llanmasi (README.md)
GitHub yoki loyiha papkasida qoldirish uchun har bir routening misol URL manzillari ko'rsatilgan qo'llanma:

Markdown
# Flask Dinamik Router va Navigatsiya Loyihasi

Ushbu loyihada Flask freymvorkida sahifalararo bog'lanish (`url_for`) va dinamik URL parametrlaridan foydalanish ko'rsatilgan.

## Loyihani ishga tushirish

1. Virtual muhitni faollashtiring va Flask-ni o'rnating:
   ```bash
   pip install flask
Dasturni ishga tushiring:

Bash
python app.py
Ilova standart holatda http://127.0.0.1:5000 manzilida ishlaydi.

Route-lar ro'yxati va misol URL-manzillar
Ilova ishga tushgach, brauzer orqali quyidagi manzillarga kirib tekshirishingiz mumkin:

Bosh sahifa (Home):

URL: http://127.0.0.1:5000/

Tavsif: Saytning asosiy sahifasi. Ichida boshqa sahifalarga o'tish uchun dinamik linklar mavjud.

Biz haqimizda sahifasi (About):

URL: http://127.0.0.1:5000/about

Tavsif: Loyiha haqida ma'lumot beruvchi statik route.

Dinamik salomlashish sahifasi (Greet):

Misol URL 1: http://127.0.0.1:5000/user/Ali (Ekranga "Salom, Ali!" deb chiqaradi)

Misol URL 2: http://127.0.0.1:5000/user/Zilola (Ekranga "Salom, Zilola!" deb chiqaradi)

Tavsif: URL tarkibidagi <name> o'zgaruvchisini olib, sahifada dinamik aks ettiradi.
