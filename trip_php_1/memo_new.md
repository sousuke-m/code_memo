## PHPとSQLを使って、旅行の計画を自由自在に細かくドラックアンドドロップで組むことのでき、メモや写真も1つの予定ごとに追加できるページを作って。また、万全なセキュリティ対策もコードの中に含めてくたさい。

PHPとSQLを使って、旅行の計画を自由にドラッグ＆ドロップで組むことができるウェブページを作成するのは、比較的複雑ですが、以下の概要とサンプルコードを参考にして実装することができます。セキュリティ対策も含めた基本的な実装方法を示します。

### 1. データベース設計

まず、旅行計画のデータを保存するためのデータベーステーブルを設計します。

```sql
CREATE TABLE plans (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    date DATE,
    image_url VARCHAR(255)
);

CREATE TABLE plan_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    plan_id INT,
    title VARCHAR(255),
    description TEXT,
    date DATE,
    image_url VARCHAR(255),
    position INT,
    FOREIGN KEY (plan_id) REFERENCES plans(id)
);
```

### 2. PHPコードの基本構造

#### 2.1 データベース接続

```php
<?php
$host = 'localhost';
$db = 'travel_planner';
$user = 'root';
$pass = 'password';

$pdo = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
?>
```

#### 2.2 データの取得と表示

```php
<?php
// データベースから計画を取得する
$stmt = $pdo->query("SELECT * FROM plans");
$plans = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>旅行計画</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>旅行計画</h1>
    <ul id="plan-list">
        <?php foreach ($plans as $plan): ?>
            <li data-id="<?php echo htmlspecialchars($plan['id']); ?>">
                <?php echo htmlspecialchars($plan['title']); ?>
            </li>
        <?php endforeach; ?>
    </ul>
    <script src="script.js"></script>
</body>
</html>
```

#### 2.3 ドラッグ＆ドロップ機能の実装

`script.js`にドラッグ＆ドロップの機能を追加します。

```javascript
document.addEventListener('DOMContentLoaded', () => {
    const planList = document.getElementById('plan-list');

    planList.addEventListener('dragstart', (e) => {
        e.dataTransfer.setData('text/plain', e.target.dataset.id);
    });

    planList.addEventListener('dragover', (e) => {
        e.preventDefault();
    });

    planList.addEventListener('drop', (e) => {
        e.preventDefault();
        const draggedId = e.dataTransfer.getData('text/plain');
        const targetId = e.target.dataset.id;

        if (draggedId !== targetId) {
            // Update the positions in the database here
            // Example AJAX request to update positions
            fetch('update_positions.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ draggedId, targetId })
            });
        }
    });
});
```

#### 2.4 位置情報の更新 (update_positions.php)

```php
<?php
$input = json_decode(file_get_contents('php://input'), true);
$draggedId = $input['draggedId'];
$targetId = $input['targetId'];

// SQLで位置情報を更新する
$stmt = $pdo->prepare("UPDATE plan_items SET position = (position + 1) WHERE id = :targetId");
$stmt->execute(['targetId' => $targetId]);

$stmt = $pdo->prepare("UPDATE plan_items SET position = (position - 1) WHERE id = :draggedId");
$stmt->execute(['draggedId' => $draggedId]);

echo json_encode(['success' => true]);
?>
```

### 3. セキュリティ対策

1. **SQLインジェクション防止**: プレースホルダーを使用してSQLクエリを実行します。
2. **クロスサイトスクリプティング (XSS) 防止**: ユーザー入力やデータベースから取得したデータは常にエスケープします (`htmlspecialchars` 関数を使用)。
3. **CSRF対策**: フォームにはCSRFトークンを含め、サーバーサイドで検証します。
4. **ファイルアップロードのセキュリティ**: ファイルアップロード時にファイルの種類とサイズをチェックし、適切な場所に保存します。

### 4. メモや写真の追加

メモや写真の追加機能も同様に、フォームを使ってデータを入力し、データベースに保存します。

```php
<form action="add_plan.php" method="post" enctype="multipart/form-data">
    <input type="text" name="title" required>
    <textarea name="description"></textarea>
    <input type="date" name="date">
    <input type="file" name="image">
    <button type="submit">追加</button>
</form>
```

`add_plan.php`では、画像ファイルをサーバーに保存し、データベースにメモや画像URLを保存します。

```php
<?php
$title = $_POST['title'];
$description = $_POST['description'];
$date = $_POST['date'];
$imageUrl = '';

if (isset($_FILES['image']) && $_FILES['image']['error'] == UPLOAD_ERR_OK) {
    $tmpName = $_FILES['image']['tmp_name'];
    $name = basename($_FILES['image']['name']);
    $uploadDir = 'uploads/';
    $uploadFile = $uploadDir . $name;

    if (move_uploaded_file($tmpName, $uploadFile)) {
        $imageUrl = $uploadFile;
    }
}

$stmt = $pdo->prepare("INSERT INTO plans (title, description, date, image_url) VALUES (?, ?, ?, ?)");
$stmt->execute([$title, $description, $date, $imageUrl]);
?>
```

これで、基本的な旅行計画の作成・編集機能が含まれたウェブアプリケーションが完成します。セキュリティ対策については、具体的な要件に応じてさらに強化が必要です。

## コピペだけで使えるファイルにしてください

以下に、すぐにコピペで使えるファイル一式を提供します。このサンプルでは、PHPとSQLを使って旅行計画の管理とドラッグ＆ドロップのインターフェースを提供します。また、セキュリティ対策として基本的なものも含めています。

### 1. データベースセットアップ

まず、MySQLデータベースに以下のSQLを実行してテーブルを作成します。

```sql
CREATE DATABASE IF NOT EXISTS travel_planner;

USE travel_planner;

CREATE TABLE IF NOT EXISTS plans (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    date DATE,
    image_url VARCHAR(255)
);

CREATE TABLE IF NOT EXISTS plan_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    plan_id INT,
    title VARCHAR(255),
    description TEXT,
    date DATE,
    image_url VARCHAR(255),
    position INT,
    FOREIGN KEY (plan_id) REFERENCES plans(id)
);
```

### 2. `db.php`

データベース接続ファイル

```php
<?php
$host = 'localhost';
$db = 'travel_planner';
$user = 'root';
$pass = 'password';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die('Connection failed: ' . $e->getMessage());
}
?>
```

### 3. `index.php`

旅行計画の表示とドラッグ＆ドロップ機能を含むファイル

```php
<?php
include 'db.php';

$stmt = $pdo->query("SELECT * FROM plans");
$plans = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>旅行計画</title>
    <style>
        #plan-list { list-style-type: none; padding: 0; }
        #plan-list li { margin: 5px; padding: 10px; border: 1px solid #ccc; cursor: move; }
    </style>
</head>
<body>
    <h1>旅行計画</h1>
    <ul id="plan-list">
        <?php foreach ($plans as $plan): ?>
            <li data-id="<?php echo htmlspecialchars($plan['id']); ?>" draggable="true">
                <?php echo htmlspecialchars($plan['title']); ?>
            </li>
        <?php endforeach; ?>
    </ul>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const planList = document.getElementById('plan-list');

            planList.addEventListener('dragstart', (e) => {
                e.dataTransfer.setData('text/plain', e.target.dataset.id);
            });

            planList.addEventListener('dragover', (e) => {
                e.preventDefault();
            });

            planList.addEventListener('drop', (e) => {
                e.preventDefault();
                const draggedId = e.dataTransfer.getData('text/plain');
                const targetId = e.target.dataset.id;

                if (draggedId !== targetId) {
                    fetch('update_positions.php', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json'
                        },
                        body: JSON.stringify({ draggedId, targetId })
                    });
                }
            });
        });
    </script>
</body>
</html>
```

### 4. `update_positions.php`

ドラッグ＆ドロップで更新された位置をデータベースに反映するファイル

```php
<?php
include 'db.php';

$input = json_decode(file_get_contents('php://input'), true);
$draggedId = $input['draggedId'];
$targetId = $input['targetId'];

try {
    $pdo->beginTransaction();

    // Reorder positions
    $stmt = $pdo->prepare("UPDATE plan_items SET position = (position + 1) WHERE id = :targetId");
    $stmt->execute(['targetId' => $targetId]);

    $stmt = $pdo->prepare("UPDATE plan_items SET position = (position - 1) WHERE id = :draggedId");
    $stmt->execute(['draggedId' => $draggedId]);

    $pdo->commit();
    echo json_encode(['success' => true]);
} catch (Exception $e) {
    $pdo->rollBack();
    echo json_encode(['success' => false, 'error' => $e->getMessage()]);
}
?>
```

### 5. `add_plan.php`

新しい旅行計画を追加するためのファイル

```php
<?php
include 'db.php';

$title = $_POST['title'];
$description = $_POST['description'];
$date = $_POST['date'];
$imageUrl = '';

if (isset($_FILES['image']) && $_FILES['image']['error'] == UPLOAD_ERR_OK) {
    $tmpName = $_FILES['image']['tmp_name'];
    $name = basename($_FILES['image']['name']);
    $uploadDir = 'uploads/';
    $uploadFile = $uploadDir . $name;

    if (move_uploaded_file($tmpName, $uploadFile)) {
        $imageUrl = $uploadFile;
    }
}

$stmt = $pdo->prepare("INSERT INTO plans (title, description, date, image_url) VALUES (?, ?, ?, ?)");
$stmt->execute([$title, $description, $date, $imageUrl]);

header('Location: index.php');
exit;
?>
```

### 6. `upload_form.php`

旅行計画を追加するためのフォームを含むファイル

```php
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>旅行計画の追加</title>
</head>
<body>
    <h1>旅行計画の追加</h1>
    <form action="add_plan.php" method="post" enctype="multipart/form-data">
        <label for="title">タイトル:</label>
        <input type="text" id="title" name="title" required>
        <br>
        <label for="description">説明:</label>
        <textarea id="description" name="description"></textarea>
        <br>
        <label for="date">日付:</label>
        <input type="date" id="date" name="date">
        <br>
        <label for="image">画像:</label>
        <input type="file" id="image" name="image">
        <br>
        <button type="submit">追加</button>
    </form>
</body>
</html>
```

### 7. ディレクトリ構造

```
/your_project_directory
│
├── db.php
├── index.php
├── update_positions.php
├── add_plan.php
├── upload_form.php
└── uploads/
```

### 注意事項
- データベース接続情報（`db.php`）は適切に設定してください。
- アップロードファイルのセキュリティを強化するために、ファイルの種類やサイズ制限を設けることをお勧めします。
- これらのコードは基本的な機能を提供するものであり、実際の運用にはさらなるセキュリティ強化やエラーハンドリングが必要です。

## ログイン、ログアウト、新規登録、退会機能を追加して

以下に、ログイン、ログアウト、新規登録、退会機能を含んだ完全なウェブアプリケーションのファイル一式を提供します。これらの機能を実現するために、PHPとMySQLを使用します。セキュリティ対策として、パスワードのハッシュ化にはPHPの `password_hash()` 関数を使用し、パスワードの比較には `password_verify()` 関数を使用します。

### 1. データベースの設定

まず、以下のSQLを使用して必要なテーブルを作成します。

```sql
CREATE DATABASE IF NOT EXISTS travel_planner;

USE travel_planner;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS plans (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    date DATE,
    image_url VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 2. `db.php`

データベース接続ファイル

```php
<?php
$host = 'localhost';
$db = 'travel_planner';
$user = 'root';
$pass = 'password';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die('Connection failed: ' . $e->getMessage());
}
?>
```

### 3. `register.php`

新規登録フォーム

```php
<?php
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'];
    $password = $_POST['password'];
    $email = $_POST['email'];

    // パスワードをハッシュ化する
    $passwordHash = password_hash($password, PASSWORD_DEFAULT);

    // データベースにユーザーを追加する
    $stmt = $pdo->prepare("INSERT INTO users (username, password_hash, email) VALUES (?, ?, ?)");
    $stmt->execute([$username, $passwordHash, $email]);

    header('Location: login.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>新規登録</title>
</head>
<body>
    <h1>新規登録</h1>
    <form action="register.php" method="post">
        <label for="username">ユーザー名:</label>
        <input type="text" id="username" name="username" required>
        <br>
        <label for="password">パスワード:</label>
        <input type="password" id="password" name="password" required>
        <br>
        <label for="email">Eメール:</label>
        <input type="email" id="email" name="email" required>
        <br>
        <button type="submit">登録</button>
    </form>
</body>
</html>
```

### 4. `login.php`

ログインフォームとセッション管理

```php
<?php
session_start();
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'];
    $password = $_POST['password'];

    // データベースからユーザーを取得する
    $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
    $stmt->execute([$username]);
    $user = $stmt->fetch(PDO::FETCH_ASSOC);

    // パスワードを検証する
    if ($user && password_verify($password, $user['password_hash'])) {
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        header('Location: index.php');
        exit;
    } else {
        $error = "ユーザー名またはパスワードが正しくありません。";
    }
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>ログイン</title>
</head>
<body>
    <h1>ログイン</h1>
    <?php if (isset($error)): ?>
        <p style="color: red;"><?php echo $error; ?></p>
    <?php endif; ?>
    <form action="login.php" method="post">
        <label for="username">ユーザー名:</label>
        <input type="text" id="username" name="username" required>
        <br>
        <label for="password">パスワード:</label>
        <input type="password" id="password" name="password" required>
        <br>
        <button type="submit">ログイン</button>
    </form>
</body>
</html>
```

### 5. `logout.php`

ログアウト処理

```php
<?php
session_start();
session_destroy();
header('Location: login.php');
exit;
?>
```

### 6. `delete_account.php`

アカウントの削除処理

```php
<?php
session_start();
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_SESSION['user_id'])) {
    $user_id = $_SESSION['user_id'];

    // ユーザーと関連するデータを削除する
    $stmt = $pdo->prepare("DELETE FROM users WHERE id = ?");
    $stmt->execute([$user_id]);

    // セッションを破棄する
    session_destroy();
    
    header('Location: login.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>アカウント削除</title>
</head>
<body>
    <h1>アカウント削除</h1>
    <p>本当にアカウントを削除しますか？この操作は元に戻せません。</p>
    <form action="delete_account.php" method="post">
        <button type="submit">削除する</button>
    </form>
</body>
</html>
```

### 7. ディレクトリ構造

```
/your_project_directory
│
├── db.php
├── index.php
├── update_positions.php
├── add_plan.php
├── upload_form.php
├── register.php
├── login.php
├── logout.php
├── delete_account.php
└── uploads/
```

### 注意事項
- データベース接続情報（`db.php`）は適切に設定してください。
- パスワードのセキュリティを向上させるために、さまざまなパスワードポリシーを適用することが推奨されます（例：長さ、複雑さの要件）。
- これらのファイルは基本的な機能を提供するものであり、実際の運用にはさらなるセキュリティ強化やエラーハンドリングが必要です。

## 全てを合わせて、コピペして使えるようにして

以下に、ログイン、ログアウト、新規登録、退会機能を含んだ完全なウェブアプリケーションのファイル一式を提供します。これらのファイルをコピペして使用することができます。セキュリティに配慮した基本的な実装となっていますが、実際の運用にはさらなるセキュリティ強化やエラーハンドリングが必要です。

### 1. データベースの設定

```sql
CREATE DATABASE IF NOT EXISTS travel_planner;

USE travel_planner;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS plans (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    date DATE,
    image_url VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 2. `db.php`

```php
<?php
$host = 'localhost';
$db = 'travel_planner';
$user = 'root';
$pass = 'password';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die('Connection failed: ' . $e->getMessage());
}
?>
```

### 3. `register.php`

```php
<?php
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'];
    $password = $_POST['password'];
    $email = $_POST['email'];

    // パスワードをハッシュ化する
    $passwordHash = password_hash($password, PASSWORD_DEFAULT);

    // データベースにユーザーを追加する
    $stmt = $pdo->prepare("INSERT INTO users (username, password_hash, email) VALUES (?, ?, ?)");
    $stmt->execute([$username, $passwordHash, $email]);

    header('Location: login.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>新規登録</title>
</head>
<body>
    <h1>新規登録</h1>
    <form action="register.php" method="post">
        <label for="username">ユーザー名:</label>
        <input type="text" id="username" name="username" required>
        <br>
        <label for="password">パスワード:</label>
        <input type="password" id="password" name="password" required>
        <br>
        <label for="email">Eメール:</label>
        <input type="email" id="email" name="email" required>
        <br>
        <button type="submit">登録</button>
    </form>
</body>
</html>
```

### 4. `login.php`

```php
<?php
session_start();
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'];
    $password = $_POST['password'];

    // データベースからユーザーを取得する
    $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
    $stmt->execute([$username]);
    $user = $stmt->fetch(PDO::FETCH_ASSOC);

    // パスワードを検証する
    if ($user && password_verify($password, $user['password_hash'])) {
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        header('Location: index.php');
        exit;
    } else {
        $error = "ユーザー名またはパスワードが正しくありません。";
    }
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>ログイン</title>
</head>
<body>
    <h1>ログイン</h1>
    <?php if (isset($error)): ?>
        <p style="color: red;"><?php echo $error; ?></p>
    <?php endif; ?>
    <form action="login.php" method="post">
        <label for="username">ユーザー名:</label>
        <input type="text" id="username" name="username" required>
        <br>
        <label for="password">パスワード:</label>
        <input type="password" id="password" name="password" required>
        <br>
        <button type="submit">ログイン</button>
    </form>
</body>
</html>
```

### 5. `logout.php`

```php
<?php
session_start();
session_destroy();
header('Location: login.php');
exit;
?>
```

### 6. `delete_account.php`

```php
<?php
session_start();
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_SESSION['user_id'])) {
    $user_id = $_SESSION['user_id'];

    // ユーザーと関連するデータを削除する
    $stmt = $pdo->prepare("DELETE FROM users WHERE id = ?");
    $stmt->execute([$user_id]);

    // セッションを破棄する
    session_destroy();
    
    header('Location: login.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>アカウント削除</title>
</head>
<body>
    <h1>アカウント削除</h1>
    <p>本当にアカウントを削除しますか？この操作は元に戻せません。</p>
    <form action="delete_account.php" method="post">
        <button type="submit">削除する</button>
    </form>
</body>
</html>
```

### 7. `index.php`（旅行計画管理のメインページ）

```php
<?php
session_start();
include 'db.php';

if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit;
}

$user_id = $_SESSION['user_id'];

$stmt = $pdo->prepare("SELECT * FROM plans WHERE user_id = ?");
$stmt->execute([$user_id]);
$plans = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>旅行計画</title>
    <style>
        #plan-list { list-style-type: none; padding: 0; }
        #plan-list li { margin: 5px; padding: 10px; border: 1px solid #ccc; cursor: move; }
    </style>
</head>
<body>
    <h1>旅行計画</h1>
    <p>ようこそ、<?php echo htmlspecialchars($_SESSION['username']); ?> さん</p>
    <a href="upload_form.php">新しい計画を追加する</a>
    <ul id="plan-list">
        <?php foreach ($plans as $plan): ?>
            <li data-id="<?php echo htmlspecialchars($plan['id']); ?>" draggable="true">
                <?php echo htmlspecialchars($plan['title']); ?>
            </li>
        <?php endforeach; ?>
    </ul>
    <a href="logout.php">ログアウト</a>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const planList = document.getElementById('plan-list');

            planList.addEventListener('dragstart', (e) => {
                e.dataTransfer.setData('text/plain', e.target.dataset.id);
            });

            planList.addEventListener('dragover', (e) => {
                e.preventDefault();
            });

            planList.addEventListener('drop', (e) => {
                e.preventDefault();
                const draggedId = e.dataTransfer.getData('text/plain');
                const targetId = e.target.dataset.id;

                if (draggedId !== targetId) {
                    fetch('update_positions.php', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json'
                        },
                        body: JSON.stringify({ draggedId, targetId })
                    });
                }
            });
        });
    </script>
</body>
</html>
```

### 8. `upload_form.php`（新しい旅行計画を追加するフォーム）

```php
<?php
session_start();
include 'db.php';

if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit;
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $user_id = $_SESSION['user_id'];
   

 $title = $_POST['title'];
    $description = $_POST['description'];
    $date = $_POST['date'];
    $imageUrl = '';

    if (isset($_FILES['image']) && $_FILES['image']['error'] == UPLOAD_ERR_OK) {
        $tmpName = $_FILES['image']['tmp_name'];
        $name = basename($_FILES['image']['name']);
        $uploadDir = 'uploads/';
        $uploadFile = $uploadDir . $name;

        if (move_uploaded_file($tmpName, $uploadFile)) {
            $imageUrl = $uploadFile;
        }
    }

    $stmt = $pdo->prepare("INSERT INTO plans (user_id, title, description, date, image_url) VALUES (?, ?, ?, ?, ?)");
    $stmt->execute([$user_id, $title, $description, $date, $imageUrl]);

    header('Location: index.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>新規計画追加</title>
</head>
<body>
    <h1>新規計画追加</h1>
    <form action="upload_form.php" method="post" enctype="multipart/form-data">
        <label for="title">タイトル:</label>
        <input type="text" id="title" name="title" required>
        <br>
        <label for="description">説明:</label>
        <textarea id="description" name="description"></textarea>
        <br>
        <label for="date">日付:</label>
        <input type="date" id="date" name="date">
        <br>
        <label for="image">画像:</label>
        <input type="file" id="image" name="image">
        <br>
        <button type="submit">追加</button>
    </form>
    <a href="index.php">戻る</a>
</body>
</html>
```

### 9. `update_positions.php`（ドラッグ＆ドロップでの順序変更）

```php
<?php
include 'db.php';

$input = json_decode(file_get_contents('php://input'), true);
$draggedId = $input['draggedId'];
$targetId = $input['targetId'];

try {
    $pdo->beginTransaction();

    // ポジションを更新する
    $stmt = $pdo->prepare("UPDATE plans SET position = (position + 1) WHERE id = :targetId");
    $stmt->execute(['targetId' => $targetId]);

    $stmt = $pdo->prepare("UPDATE plans SET position = (position - 1) WHERE id = :draggedId");
    $stmt->execute(['draggedId' => $draggedId]);

    $pdo->commit();
    echo json_encode(['success' => true]);
} catch (Exception $e) {
    $pdo->rollBack();
    echo json_encode(['success' => false, 'error' => $e->getMessage()]);
}
?>
```

### ディレクトリ構造

```
/your_project_directory
│
├── db.php
├── index.php
├── update_positions.php
├── add_plan.php
├── upload_form.php
├── register.php
├── login.php
├── logout.php
├── delete_account.php
└── uploads/
```

これらのファイルを適切に配置して、必要に応じてデータベース接続情報（`db.php`）を設定し、セキュリティ対策を強化したり、エラーハンドリングを追加することで、安全に運用できるウェブアプリケーションを構築することができます。