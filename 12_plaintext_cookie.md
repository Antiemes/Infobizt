# Cookie-k tárolása nyílt szövegként

## Rendszerterv

A következő sérülékeny webalkalmazás fő hibája, hogy a sütiket (cookie-kat)
nyílt szövegként tárolja, bármiféle ellenőrzés nélkül. A hiba demonstrálása
most egy PHP webalkalmazássa történik, amihez a már ismert módon telepíthetjük
az Apache webszervert, a PHP modullal (vagy feltételezhetjük, hogy az még
fut). Ebben az esetben a PHP tartalmat elegendő egyszerűen a célhelyre másolni.

## A webalkalmazás létrehozása

A webalkalmazást az `index.php` fájlban helyezzük el:

```bash
cat > index.php
```

Ez lesz a tartalma:

```php
<?php

// Hardcoded users
$users = [
    'admin'     => '$2y$12$amDAD2ukCwoeFqWKFV2OwOyLxV/pFuwRAzNJt.xj7xGFDwMVkQhuq', //supersecret
    'user'      => '$2y$12$aYrWTFZH4aOYwFbnSwcKieWYgKJc4204to0fk1TwrBjWVOXfK6cEG', //123
];

// Get current logged-in user from plaintext cookie
function current_user() {
    if (empty($_COOKIE['username'])) {
        return false;
    }
    $username = $_COOKIE['username'];
    return isset($GLOBALS['users'][$username]) ? $username : false;
}

// Get action from GET parameter (default = home)
$action = $_GET['action'] ?? 'home';

// =============================================
//  Handle actions
// =============================================

if ($action === 'logout') {
    // Clear the plaintext session cookie
    setcookie('username', '', time() - 3600 * 24 * 365, '/');
    header('Location: index.php');
    exit;
}

if ($action === 'login') {
    // -------------------
    //  LOGIN PROCESSING
    // -------------------
    $error = '';
    $user = current_user();

    // If already logged in, go home
    if ($user) {
        header('Location: index.php');
        exit;
    }

    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
        $username = trim($_POST['username'] ?? '');
        $password = $_POST['password'] ?? '';

        if ($username === '' || $password === '') {
            $error = 'Both fields are required.';
        }
        elseif (!isset($users[$username])) {
            $error = 'Invalid username or password.';
        }
        elseif (!password_verify($password, $users[$username])) {
            $error = 'Invalid username or password.';
        }
        else {
            // SUCCESS: Set plaintext cookie
            setcookie('username', $username, 0, '/');   // session cookie
            header('Location: index.php');
            exit;
        }
    }

    // Show login form
    ?>
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Login</title>
        <style>
            body { font-family: system-ui, sans-serif; background:#f8fafc; display:flex; min-height:100vh; align-items:center; justify-content:center; }
            .card { background:white; padding:2.5rem; border-radius:12px; box-shadow:0 10px 25px rgba(0,0,0,0.08); width:100%; max-width:380px; }
            h1 { text-align:center; color:#1e40af; margin-bottom:1.8rem; }
            label { display:block; margin:1.2rem 0 0.4rem; font-weight:500; color:#334155; }
            input { width:100%; padding:0.75rem; border:1px solid #cbd5e1; border-radius:6px; font-size:1rem; box-sizing:border-box; }
            input:focus { outline:none; border-color:#3b82f6; box-shadow:0 0 0 3px rgba(59,130,246,0.2); }
            button { width:100%; margin-top:1.8rem; padding:0.9rem; background:#2563eb; color:white; border:none; border-radius:6px; font-size:1.05rem; cursor:pointer; font-weight:600; }
            button:hover { background:#1d4ed8; }
            .error { color:#dc2626; text-align:center; margin:1rem 0; font-weight:500; }
        </style>
    </head>
    <body>
        <div class="card">
            <h1>Sign In</h1>

            <?php if ($error): ?>
                <div class="error"><?= htmlspecialchars($error) ?></div>
            <?php endif; ?>

            <form method="post" action="index.php?action=login">
                <label for="username">Username</label>
                <input type="text" id="username" name="username" autocomplete="username" required>

                <label for="password">Password</label>
                <input type="password" id="password" name="password" autocomplete="current-password" required>

                <button type="submit">Log In</button>
            </form>
        </div>
    </body>
    </html>
    <?php
    exit;
}

// -------------------
//  HOME / DEFAULT
// -------------------
$user = current_user();

if (!$user) {
    // Not logged in → redirect to login
    header('Location: index.php?action=login');
    exit;
}

// Show welcome page
header('Content-Type: text/html; charset=utf-8');
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
    <style>
        body { font-family: system-ui, sans-serif; text-align:center; padding: 80px 20px; background:#f8fafc; }
        h1   { color: #2563eb; }
        .box { background: #f1f5f9; padding: 2.5rem; border-radius: 12px; max-width: 420px; margin: 0 auto; }
        a.logout { color: #dc2626; text-decoration: none; font-weight: bold; }
        a.logout:hover { text-decoration: underline; }
    </style>
</head>
<body>
    <div class="box">
        <h1>Welcome, <?= htmlspecialchars($user) ?>!</h1>
        <p>You are successfully logged in.</p>
        <p>
            <a href="index.php?action=logout" class="logout">Log out</a>
        </p>
    </div>
</body>
</html>
```

Ezután az `index.php` fájlt helyezzük a `/var/www/html` könyvtárba.

```bash
sudo cp index.php /var/www/html
```

## A sérülékenység bemutatása

A weboldalra a `user` nevű felhasználóval tudunk belépni, akinek a jelszava `123`. Ekkor a
webalkalmazás elhelyez a kliensnél egy cookie-t és arról azonosítja. (Kijelentkezésnél a
cookie eltűnik.) Ha a cookie-t átírjuk egy másik felhasználóra, akkor vele is be tudunk lépni.

A webalkalmazás alapvető hibája, hogy megbízik a kliens által küldött adatokkal. Hasonló
jellegű hiba alkalmazások széles körét érintheti (például képzeljünk el egy olyan esetet, amikor
egy banki alkalmazás a kliensre bízza a fennmaradó egyenleg kiszámítását.)

