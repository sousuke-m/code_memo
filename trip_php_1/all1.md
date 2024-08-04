## PHPとSQLを使って、旅行の計画を自由自在に細かくドラックアンドドロップで組むことのでき、メモや写真も1つの予定ごとに追加できるページを作ってください。また、旅行の日数は自由に決めることができるようにしてください。なお、万全なセキュリティ対策もコードの中に含めてくたさい。そして、ログイン、ログアウト、新規登録、退会の機能も備えてください。あと、複数の旅行を管理できるようにしてください。

作成するページの要件を整理します。

1. **旅行計画ページ**:
   - PHPとSQLを使用して、ドラックアンドドロップで旅行の計画を自由に組むことができるページを作成します。
   - メモや写真を各予定に追加できるようにします。

2. **旅行の日数の自由設定**:
   - ユーザーが旅行の日数を自由に設定できるようにします。

3. **セキュリティ対策**:
   - セキュリティ対策を万全に行い、SQLインジェクションやクロスサイトスクリプティング（XSS）などの脆弱性に対処します。

4. **ユーザーアカウント管理**:
   - ユーザーのログイン、ログアウト、新規登録、退会の機能を実装します。
   - 複数の旅行を管理できるように、ユーザーごとに複数の旅行計画を作成できるようにします。

これらの要件に基づいて、以下に基本的な実装ステップを示します。

### 技術的な実装:

#### 1. データベース設計 (MySQLを例として)

- **usersテーブル**:
  - id (INT, 主キー)
  - username (VARCHAR)
  - password (VARCHAR, ハッシュ化して保存する)
  - email (VARCHAR)
  - created_at (TIMESTAMP)

- **tripsテーブル**:
  - id (INT, 主キー)
  - user_id (INT, usersテーブルの外部キー)
  - trip_name (VARCHAR)
  - start_date (DATE)
  - end_date (DATE)
  - created_at (TIMESTAMP)

- **plansテーブル**:
  - id (INT, 主キー)
  - trip_id (INT, tripsテーブルの外部キー)
  - plan_title (VARCHAR)
  - plan_description (TEXT)
  - plan_order (INT, ドラックアンドドロップの順序を保存するため)

- **photosテーブル**:
  - id (INT, 主キー)
  - plan_id (INT, plansテーブルの外部キー)
  - photo_path (VARCHAR)
  - photo_description (TEXT)

#### 2. PHPコードの基本構成:

- **ログイン**:
  - ユーザーが提供したユーザー名とパスワードをチェックし、セッションを開始します。

- **ログアウト**:
  - 現在のセッションを破棄し、ログアウトします。

- **新規登録**:
  - ユーザー名、パスワード（ハッシュ化）、メールアドレスを保存します。

- **退会**:
  - ユーザーアカウントを削除します。それに伴って、関連する旅行データも削除します。

- **旅行計画管理**:
  - 旅行の追加、編集、削除を行います。
  - 各旅行に対して、ドラックアンドドロップで計画を管理するためのインターフェースを提供します。

- **セキュリティ対策**:
  - PDO（PHP Data Objects）を使用してSQLインジェクションを防ぎます。
  - ユーザー入力を適切にエスケープまたはフィルタリングしてXSS攻撃を防ぎます。
  - パスワードはハッシュ化して保存し、セキュアな方法で認証を行います。

#### 3. フロントエンド:

- **HTML/CSS/JavaScript**:
  - ドラックアンドドロップのインターフェースを実装します（例えば、jQuery UIなどを使用）。
  - 旅行計画ページには、予定ごとにメモを追加するフォームや写真をアップロードする機能を備えます。

### 結論:

上記の要件を満たすために、PHPでバックエンドを実装し、MySQLでデータベースを設計・管理します。セキュリティを重視したコーディングを行い、ユーザーが使いやすいインターフェースを提供します。それぞれの機能を段階的に実装し、テストしながら完成度を高めていくことが重要です。

## これをコピペするだけで良いコードにしてください。

全体を網羅する詳細なコードは多くの要素が含まれるため、ここでは主要な部分を簡潔に示します。以下のコードは基本的な機能を実装するためのものです。セキュリティ、エラーハンドリング、ユーザーインターフェースの完成度には注意が必要です。これを基に、さらにカスタマイズと改善を行う必要があります。

### 1. データベース設計 (SQL)

```sql
CREATE DATABASE travel_planner;

USE travel_planner;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE trips (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    trip_name VARCHAR(100) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE TABLE plans (
    id INT AUTO_INCREMENT PRIMARY KEY,
    trip_id INT NOT NULL,
    plan_title VARCHAR(100) NOT NULL,
    plan_description TEXT,
    plan_order INT,
    FOREIGN KEY (trip_id) REFERENCES trips(id) ON DELETE CASCADE
);

CREATE TABLE photos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    plan_id INT NOT NULL,
    photo_path VARCHAR(255) NOT NULL,
    photo_description TEXT,
    FOREIGN KEY (plan_id) REFERENCES plans(id) ON DELETE CASCADE
);
```

### 2. PHPコード

#### `db.php` (データベース接続)

```php
<?php
$host = 'localhost';
$db = 'travel_planner';
$user = 'root'; // MySQLのユーザー名
$pass = ''; // MySQLのパスワード

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Database connection failed: " . $e->getMessage());
}
?>
```

#### `register.php` (新規登録)

```php
<?php
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'];
    $password = password_hash($_POST['password'], PASSWORD_DEFAULT);
    $email = $_POST['email'];

    $stmt = $pdo->prepare("INSERT INTO users (username, password, email) VALUES (?, ?, ?)");
    $stmt->execute([$username, $password, $email]);

    header("Location: login.php");
}
?>
<form method="post" action="">
    Username: <input type="text" name="username" required><br>
    Password: <input type="password" name="password" required><br>
    Email: <input type="email" name="email" required><br>
    <input type="submit" value="Register">
</form>
```

#### `login.php` (ログイン)

```php
<?php
session_start();
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'];
    $password = $_POST['password'];

    $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
    $stmt->execute([$username]);
    $user = $stmt->fetch(PDO::FETCH_ASSOC);

    if ($user && password_verify($password, $user['password'])) {
        $_SESSION['user_id'] = $user['id'];
        header("Location: index.php");
    } else {
        echo "Invalid credentials.";
    }
}
?>
<form method="post" action="">
    Username: <input type="text" name="username" required><br>
    Password: <input type="password" name="password" required><br>
    <input type="submit" value="Login">
</form>
```

#### `index.php` (旅行計画の管理)

```php
<?php
session_start();
include 'db.php';

if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['trip_name'])) {
    $trip_name = $_POST['trip_name'];
    $start_date = $_POST['start_date'];
    $end_date = $_POST['end_date'];
    $user_id = $_SESSION['user_id'];

    $stmt = $pdo->prepare("INSERT INTO trips (user_id, trip_name, start_date, end_date) VALUES (?, ?, ?, ?)");
    $stmt->execute([$user_id, $trip_name, $start_date, $end_date]);
}

$trips = $pdo->prepare("SELECT * FROM trips WHERE user_id = ?");
$trips->execute([$_SESSION['user_id']]);
$trips = $trips->fetchAll(PDO::FETCH_ASSOC);
?>
<h1>Welcome to Travel Planner</h1>
<form method="post" action="">
    Trip Name: <input type="text" name="trip_name" required><br>
    Start Date: <input type="date" name="start_date" required><br>
    End Date: <input type="date" name="end_date" required><br>
    <input type="submit" value="Add Trip">
</form>

<h2>Your Trips</h2>
<ul>
    <?php foreach ($trips as $trip): ?>
        <li>
            <?php echo htmlspecialchars($trip['trip_name']); ?> 
            (<?php echo htmlspecialchars($trip['start_date']); ?> - <?php echo htmlspecialchars($trip['end_date']); ?>)
        </li>
    <?php endforeach; ?>
</ul>
<a href="logout.php">Logout</a>
```

#### `logout.php` (ログアウト)

```php
<?php
session_start();
session_unset();
session_destroy();
header("Location: login.php");
?>
```

### 注意点

1. **セキュリティ**: 上記のコードは基本的な実装です。実際には、XSS、CSRF攻撃対策や、セキュリティ強化のためのさらなる検討が必要です。特に、入力値の検証とサニタイズを行うことが重要です。

2. **フロントエンド**: ドラッグ＆ドロップ機能や画像アップロード機能を実装するには、JavaScript（例: jQuery UI）を使う必要があります。これらのフロントエンド機能は、別途実装する必要があります。

3. **エラーハンドリング**: 上記のコードは基本的なエラーハンドリングを含んでいますが、実際の運用ではより詳細なエラーハンドリングが必要です。

これらを基にして、機能を追加・拡張し、セキュリティ対策を講じてください。

## ディレクトリ構造を見せてください。

以下は、あなたの旅行計画アプリケーションの基本的なディレクトリ構造の一例です。この構造は、コードの管理と拡張をしやすくするためのものです。セキュリティと保守性を考慮して、各コンポーネントを分離しています。

```
/travel-planner
├── /public
│   ├── /css
│   │   └── styles.css           # スタイルシート
│   ├── /js
│   │   └── scripts.js           # JavaScriptファイル (例: ドラッグ＆ドロップ機能など)
│   ├── /uploads
│   │   └── (画像アップロード用ディレクトリ)
│   ├── index.php                # トップページ、旅行計画管理
│   ├── login.php                # ログインページ
│   ├── register.php             # 新規登録ページ
│   ├── logout.php               # ログアウト処理
│   └── (その他の公開用ファイル)
├── /src
│   ├── /controllers
│   │   ├── AuthController.php   # 認証関連のコントローラ
│   │   ├── TripController.php   # 旅行関連のコントローラ
│   │   └── PlanController.php   # 予定関連のコントローラ
│   ├── /models
│   │   ├── User.php             # ユーザーモデル
│   │   ├── Trip.php             # 旅行モデル
│   │   └── Plan.php             # 予定モデル
│   └── /views
│       ├── header.php           # ヘッダー部分のビュー
│       ├── footer.php           # フッター部分のビュー
│       ├── trip_form.php        # 旅行追加・編集フォーム
│       └── (その他のビュー)
├── /config
│   └── database.php             # データベース接続設定
├── /logs
│   └── (ログファイル用ディレクトリ)
├── /tests
│   └── (ユニットテスト用ディレクトリ)
├── .htaccess                    # Apache設定ファイル (URLリライトなど)
├── composer.json                # Composer設定ファイル (依存管理)
└── README.md                    # プロジェクト説明ファイル
```

### ディレクトリとファイルの説明

- **/public**: ウェブサーバーが公開するディレクトリ。CSS、JavaScript、画像ファイルなどが含まれます。`index.php`やログイン、登録、ログアウトなどの公開用PHPファイルもここに置きます。

- **/src**: アプリケーションのコアロジックが含まれるディレクトリ。MVC（Model-View-Controller）パターンに基づいて、コントローラ、モデル、ビューを分けて管理します。

- **/config**: アプリケーションの設定ファイル。データベース接続情報などが含まれます。

- **/logs**: エラーログやデバッグログを保存するディレクトリ。

- **/tests**: ユニットテストや機能テストのためのファイルを含むディレクトリ。

- **.htaccess**: Apacheウェブサーバー用の設定ファイル。URLのリライトやセキュリティ設定などに使用します。

- **composer.json**: PHPの依存関係を管理するためのComposer設定ファイル。

- **README.md**: プロジェクトの概要やインストール手順を記述するファイル。

この構造を基にして、アプリケーションを開発・拡張していくことができます。

## 全てのファイルを見せてください。

すべてのファイルを一度に見せるのは長大になるため、主要なファイルとディレクトリを含む完全な例を示します。以下は、基本的な機能を持つ旅行計画アプリケーションのディレクトリ構造とそのファイルの内容です。

### ディレクトリ構造

```
/travel-planner
├── /public
│   ├── /css
│   │   └── styles.css
│   ├── /js
│   │   └── scripts.js
│   ├── /uploads
│   ├── index.php
│   ├── login.php
│   ├── register.php
│   ├── logout.php
│   └── trip.php
├── /src
│   ├── /controllers
│   │   ├── AuthController.php
│   │   ├── TripController.php
│   │   └── PlanController.php
│   ├── /models
│   │   ├── User.php
│   │   ├── Trip.php
│   │   └── Plan.php
│   └── /views
│       ├── header.php
│       ├── footer.php
│       ├── trip_form.php
│       └── plan_form.php
├── /config
│   └── database.php
├── /logs
├── /tests
├── .htaccess
├── composer.json
└── README.md
```

### 各ファイルの内容

#### `/public/index.php` (トップページ)

```php
<?php
session_start();
include '../config/database.php';
include '../src/controllers/TripController.php';

$tripController = new TripController($pdo);
$trips = $tripController->getUserTrips($_SESSION['user_id']);

?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Travel Planner</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
<?php include '../src/views/header.php'; ?>
<h1>Your Trips</h1>
<form method="post" action="trip.php">
    <input type="text" name="trip_name" placeholder="Trip Name" required>
    <input type="date" name="start_date" required>
    <input type="date" name="end_date" required>
    <input type="submit" value="Add Trip">
</form>
<ul>
    <?php foreach ($trips as $trip): ?>
        <li>
            <?php echo htmlspecialchars($trip['trip_name']); ?> 
            (<?php echo htmlspecialchars($trip['start_date']); ?> - <?php echo htmlspecialchars($trip['end_date']); ?>)
        </li>
    <?php endforeach; ?>
</ul>
<a href="logout.php">Logout</a>
<?php include '../src/views/footer.php'; ?>
</body>
</html>
```

#### `/public/login.php` (ログインページ)

```php
<?php
session_start();
include '../config/database.php';
include '../src/controllers/AuthController.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $authController = new AuthController($pdo);
    $result = $authController->login($_POST['username'], $_POST['password']);
    if ($result) {
        header('Location: index.php');
    } else {
        echo "Invalid credentials.";
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
<form method="post" action="">
    Username: <input type="text" name="username" required><br>
    Password: <input type="password" name="password" required><br>
    <input type="submit" value="Login">
</form>
<a href="register.php">Register</a>
</body>
</html>
```

#### `/public/register.php` (新規登録ページ)

```php
<?php
include '../config/database.php';
include '../src/controllers/AuthController.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $authController = new AuthController($pdo);
    $authController->register($_POST['username'], $_POST['password'], $_POST['email']);
    header('Location: login.php');
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Register</title>
</head>
<body>
<form method="post" action="">
    Username: <input type="text" name="username" required><br>
    Password: <input type="password" name="password" required><br>
    Email: <input type="email" name="email" required><br>
    <input type="submit" value="Register">
</form>
<a href="login.php">Login</a>
</body>
</html>
```

#### `/public/logout.php` (ログアウト処理)

```php
<?php
session_start();
session_unset();
session_destroy();
header('Location: login.php');
?>
```

#### `/public/trip.php` (旅行追加処理)

```php
<?php
session_start();
include '../config/database.php';
include '../src/controllers/TripController.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $tripController = new TripController($pdo);
    $tripController->addTrip($_SESSION['user_id'], $_POST['trip_name'], $_POST['start_date'], $_POST['end_date']);
    header('Location: index.php');
}
?>
```

#### `/src/controllers/AuthController.php` (認証コントローラ)

```php
<?php
class AuthController {
    private $pdo;

    public function __construct($pdo) {
        $this->pdo = $pdo;
    }

    public function register($username, $password, $email) {
        $passwordHash = password_hash($password, PASSWORD_DEFAULT);
        $stmt = $this->pdo->prepare("INSERT INTO users (username, password, email) VALUES (?, ?, ?)");
        $stmt->execute([$username, $passwordHash, $email]);
    }

    public function login($username, $password) {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE username = ?");
        $stmt->execute([$username]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        return $user && password_verify($password, $user['password']);
    }
}
?>
```

#### `/src/controllers/TripController.php` (旅行コントローラ)

```php
<?php
class TripController {
    private $pdo;

    public function __construct($pdo) {
        $this->pdo = $pdo;
    }

    public function addTrip($userId, $tripName, $startDate, $endDate) {
        $stmt = $this->pdo->prepare("INSERT INTO trips (user_id, trip_name, start_date, end_date) VALUES (?, ?, ?, ?)");
        $stmt->execute([$userId, $tripName, $startDate, $endDate]);
    }

    public function getUserTrips($userId) {
        $stmt = $this->pdo->prepare("SELECT * FROM trips WHERE user_id = ?");
        $stmt->execute([$userId]);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}
?>
```

#### `/src/models/User.php` (ユーザーモデル)

```php
<?php
class User {
    private $pdo;
    
    public function __construct($pdo) {
        $this->pdo = $pdo;
    }
    
    // ユーザー関連のメソッドを追加
}
?>
```

#### `/src/models/Trip.php` (旅行モデル)

```php
<?php
class Trip {
    private $pdo;
    
    public function __construct($pdo) {
        $this->pdo = $pdo;
    }
    
    // 旅行関連のメソッドを追加
}
?>
```

#### `/src/views/header.php` (ヘッダー)

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Travel Planner</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
<header>
    <h1>Travel Planner</h1>
    <nav>
        <a href="index.php">Home</a>
        <a href="logout.php">Logout</a>
    </nav>
</header>
```

#### `/src/views/footer.php` (フッター)

```php
<footer>
    <p>&copy; <?php echo date("Y"); ?> Travel Planner</p>
</footer>
</body>
</html>
```

#### `/src/views/trip_form.php` (旅行フォームビュー)

```php
<form method="post" action="trip.php">
    <input type="text" name="trip_name" placeholder="Trip Name" required>
    <input type="date" name="start_date" required>
    <input type="date" name="end_date" required>
    <input type="submit" value="Add Trip">
</form>
```

#### `/config/database.php` (データベース接続設定)

```php
<?php
$host = 'localhost';
$db = 'travel_planner';
$user = 'root'; // MySQLのユーザー名
$pass = ''; // MySQLのパスワード

try {
    $pdo = new PDO("mysql:host=$host;

dbname=$db;charset=utf8", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Database connection failed: " . $e->getMessage());
}
?>
```

#### `/public/css/styles.css` (スタイルシート)

```css
body {
    font-family: Arial, sans-serif;
}

header {
    background: #f4f4f4;
    padding: 10px;
}

nav a {
    margin: 0 10px;
    text-decoration: none;
}

form {
    margin: 20px 0;
}

footer {
    background: #f4f4f4;
    padding: 10px;
    text-align: center;
}
```

#### `/public/js/scripts.js` (JavaScriptファイル)

```javascript
// JavaScriptコード（例: ドラッグ＆ドロップの機能）
// このファイルは後で必要に応じて追加できます。
```

#### `.htaccess` (Apache設定ファイル)

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ /public/$1 [L]
```

#### `composer.json` (Composer設定ファイル)

```json
{
    "require": {
        "php": ">=7.4",
        "monolog/monolog": "^2.0"
    }
}
```

#### `README.md` (プロジェクト説明)

```markdown
# Travel Planner

## Overview

A simple travel planning application that allows users to manage multiple trips, add plans to each trip, and upload photos.

## Installation

1. Clone the repository.
2. Run `composer install` to install dependencies.
3. Set up the database using the provided SQL script.
4. Configure database settings in `config/database.php`.

## Usage

- Register a new account.
- Log in and start adding trips.
- Manage and view your trips and plans.

## License

This project is licensed under the MIT License.
```

これで、基本的な旅行計画アプリケーションのディレクトリ構造と主要なファイルが整いました。これをベースに、さらに機能を追加したり、デザインをカスタマイズしたりできます。

## /public/js/scripts.jsを教えてください。

`/public/js/scripts.js` は、旅行計画アプリケーションのインタラクティブな要素を追加するためのJavaScriptファイルです。以下は、基本的なドラッグ＆ドロップ機能や、予定の順序を変更するためのスクリプトの例です。このスクリプトは、ドラッグ＆ドロップのライブラリである `SortableJS` を利用して、予定をドラッグ＆ドロップで並び替えられるようにするものです。

### `/public/js/scripts.js`

```javascript
document.addEventListener('DOMContentLoaded', (event) => {
    // SortableJSライブラリを利用して、予定のリストをドラッグ＆ドロップで並び替え可能にする
    const sortable = new Sortable(document.querySelector('#plans-list'), {
        animation: 150,
        onEnd: function (evt) {
            // 並び替え後の処理
            const order = Array.from(evt.from.children).map(child => child.dataset.planId);
            // 新しい順序をサーバーに送信して保存
            fetch('/update_plan_order.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ order: order }),
            }).then(response => response.json())
              .then(data => {
                  if (!data.success) {
                      console.error('Order update failed:', data.message);
                  }
              })
              .catch(error => {
                  console.error('Error:', error);
              });
        }
    });

    // フォームの動的追加（例: 予定の追加）
    document.querySelector('#add-plan-form').addEventListener('submit', function (e) {
        e.preventDefault();
        const formData = new FormData(this);

        fetch('/add_plan.php', {
            method: 'POST',
            body: formData
        }).then(response => response.json())
          .then(data => {
              if (data.success) {
                  // 予定をリストに追加する処理
                  const newPlan = document.createElement('li');
                  newPlan.dataset.planId = data.planId;
                  newPlan.innerHTML = `
                      <span>${data.planTitle}</span>
                      <p>${data.planDescription}</p>
                  `;
                  document.querySelector('#plans-list').appendChild(newPlan);
                  this.reset(); // フォームをリセット
              } else {
                  console.error('Plan addition failed:', data.message);
              }
          })
          .catch(error => {
              console.error('Error:', error);
          });
    });
});
```

### 説明

1. **SortableJSの利用**: `SortableJS` ライブラリを使って、リストアイテム（予定）をドラッグ＆ドロップで並び替え可能にします。`#plans-list` は、予定のリストを含むHTML要素のIDです。`animation` は、ドラッグ中のアニメーションの速度です。

2. **順序の保存**: 並び替えた後の予定の順序をサーバーに送信して保存します。`fetch` を使って、`/update_plan_order.php` にPOSTリクエストを送り、新しい順序をサーバーで処理します。

3. **フォームの処理**: 新しい予定を追加するフォーム（`#add-plan-form`）をサーバーに送信し、追加された予定をリストに動的に追加します。サーバーからの応答を基に、新しい予定をリストに追加し、フォームをリセットします。

### 必要なライブラリ

- **SortableJS**: このライブラリを使用するには、CDNからライブラリを追加するか、プロジェクトにダウンロードして含める必要があります。CDNを使用する例は以下の通りです。

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/Sortable/1.14.0/Sortable.min.js"></script>
```

このスクリプトをプロジェクトに追加することで、ユーザーは旅行の予定をドラッグ＆ドロップで並び替え、動的に追加できるようになります。必要に応じて、さらに機能を追加したり、ユーザーインターフェースをカスタマイズしたりしてください。

## /src/models/User.phpと/src/models/Trip.phpを全て教えてください。

以下は、基本的なユーザーと旅行のモデルクラスの実装例です。これらのクラスは、データベースとのやり取りを抽象化し、ビジネスロジックを処理します。

### `/src/models/User.php`

```php
<?php

class User {
    private $pdo;

    public function __construct($pdo) {
        $this->pdo = $pdo;
    }

    /**
     * ユーザーを登録する
     *
     * @param string $username
     * @param string $password
     * @param string $email
     * @return bool
     */
    public function register($username, $password, $email) {
        $passwordHash = password_hash($password, PASSWORD_DEFAULT);
        $stmt = $this->pdo->prepare("INSERT INTO users (username, password, email) VALUES (?, ?, ?)");
        return $stmt->execute([$username, $passwordHash, $email]);
    }

    /**
     * ユーザー名でユーザーを検索する
     *
     * @param string $username
     * @return array|false
     */
    public function findByUsername($username) {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE username = ?");
        $stmt->execute([$username]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    /**
     * ユーザーIDでユーザーを検索する
     *
     * @param int $userId
     * @return array|false
     */
    public function findById($userId) {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$userId]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
}
```

#### 説明

- `register($username, $password, $email)`: ユーザーを新規登録します。パスワードはハッシュ化して保存します。
- `findByUsername($username)`: ユーザー名でユーザーを検索し、情報を返します。
- `findById($userId)`: ユーザーIDでユーザーを検索し、情報を返します。

### `/src/models/Trip.php`

```php
<?php

class Trip {
    private $pdo;

    public function __construct($pdo) {
        $this->pdo = $pdo;
    }

    /**
     * ユーザーの旅行リストを取得する
     *
     * @param int $userId
     * @return array
     */
    public function getUserTrips($userId) {
        $stmt = $this->pdo->prepare("SELECT * FROM trips WHERE user_id = ?");
        $stmt->execute([$userId]);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * 新しい旅行を追加する
     *
     * @param int $userId
     * @param string $tripName
     * @param string $startDate
     * @param string $endDate
     * @return bool
     */
    public function addTrip($userId, $tripName, $startDate, $endDate) {
        $stmt = $this->pdo->prepare("INSERT INTO trips (user_id, trip_name, start_date, end_date) VALUES (?, ?, ?, ?)");
        return $stmt->execute([$userId, $tripName, $startDate, $endDate]);
    }

    /**
     * 旅行の情報を更新する
     *
     * @param int $tripId
     * @param string $tripName
     * @param string $startDate
     * @param string $endDate
     * @return bool
     */
    public function updateTrip($tripId, $tripName, $startDate, $endDate) {
        $stmt = $this->pdo->prepare("UPDATE trips SET trip_name = ?, start_date = ?, end_date = ? WHERE id = ?");
        return $stmt->execute([$tripName, $startDate, $endDate, $tripId]);
    }

    /**
     * 旅行を削除する
     *
     * @param int $tripId
     * @return bool
     */
    public function deleteTrip($tripId) {
        $stmt = $this->pdo->prepare("DELETE FROM trips WHERE id = ?");
        return $stmt->execute([$tripId]);
    }
}
```

#### 説明

- `getUserTrips($userId)`: 指定されたユーザーIDに関連する旅行リストを取得します。
- `addTrip($userId, $tripName, $startDate, $endDate)`: 新しい旅行をデータベースに追加します。
- `updateTrip($tripId, $tripName, $startDate, $endDate)`: 既存の旅行の情報を更新します。
- `deleteTrip($tripId)`: 指定された旅行をデータベースから削除します。

これらのモデルクラスは、データベースとのやり取りを簡単にし、コントローラや他のクラスで再利用するためのものです。必要に応じて、さらにメソッドを追加して機能を拡張できます。

## /src/controllers/PlanController.phpと/src/models/Plan.phpを見せてください

以下は、`/src/controllers/PlanController.php` と `/src/models/Plan.php` の基本的な実装例です。これらのファイルは、旅行の予定に関連する機能を管理するためのものです。

### `/src/controllers/PlanController.php`

```php
<?php

class PlanController {
    private $pdo;

    public function __construct($pdo) {
        $this->pdo = $pdo;
    }

    /**
     * 予定を追加する
     *
     * @param int $tripId
     * @param string $title
     * @param string $description
     * @param string $date
     * @return bool
     */
    public function addPlan($tripId, $title, $description, $date) {
        $stmt = $this->pdo->prepare("INSERT INTO plans (trip_id, title, description, date) VALUES (?, ?, ?, ?)");
        return $stmt->execute([$tripId, $title, $description, $date]);
    }

    /**
     * 特定の旅行に関連する予定を取得する
     *
     * @param int $tripId
     * @return array
     */
    public function getPlansByTrip($tripId) {
        $stmt = $this->pdo->prepare("SELECT * FROM plans WHERE trip_id = ? ORDER BY date ASC");
        $stmt->execute([$tripId]);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * 予定の順序を更新する
     *
     * @param array $order
     * @return bool
     */
    public function updatePlanOrder($order) {
        $pdo->beginTransaction();
        try {
            $stmt = $this->pdo->prepare("UPDATE plans SET sort_order = ? WHERE id = ?");
            foreach ($order as $sortOrder => $planId) {
                $stmt->execute([$sortOrder, $planId]);
            }
            $pdo->commit();
            return true;
        } catch (Exception $e) {
            $pdo->rollBack();
            return false;
        }
    }

    /**
     * 予定を削除する
     *
     * @param int $planId
     * @return bool
     */
    public function deletePlan($planId) {
        $stmt = $this->pdo->prepare("DELETE FROM plans WHERE id = ?");
        return $stmt->execute([$planId]);
    }
}
```

#### 説明

- **addPlan($tripId, $title, $description, $date)**: 指定した旅行IDに関連する新しい予定を追加します。
- **getPlansByTrip($tripId)**: 指定した旅行IDに関連する予定を取得し、日付順に並べます。
- **updatePlanOrder($order)**: 予定の順序を更新します。`$order` は、予定IDとその新しい順序を含む配列です。
- **deletePlan($planId)**: 指定した予定IDに関連する予定を削除します。

### `/src/models/Plan.php`

```php
<?php

class Plan {
    private $pdo;

    public function __construct($pdo) {
        $this->pdo = $pdo;
    }

    /**
     * 予定を追加する
     *
     * @param int $tripId
     * @param string $title
     * @param string $description
     * @param string $date
     * @return bool
     */
    public function add($tripId, $title, $description, $date) {
        $stmt = $this->pdo->prepare("INSERT INTO plans (trip_id, title, description, date) VALUES (?, ?, ?, ?)");
        return $stmt->execute([$tripId, $title, $description, $date]);
    }

    /**
     * 特定の旅行に関連する予定を取得する
     *
     * @param int $tripId
     * @return array
     */
    public function getByTrip($tripId) {
        $stmt = $this->pdo->prepare("SELECT * FROM plans WHERE trip_id = ? ORDER BY date ASC");
        $stmt->execute([$tripId]);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

    /**
     * 予定の順序を更新する
     *
     * @param array $order
     * @return bool
     */
    public function updateOrder($order) {
        $this->pdo->beginTransaction();
        try {
            $stmt = $this->pdo->prepare("UPDATE plans SET sort_order = ? WHERE id = ?");
            foreach ($order as $sortOrder => $planId) {
                $stmt->execute([$sortOrder, $planId]);
            }
            $this->pdo->commit();
            return true;
        } catch (Exception $e) {
            $this->pdo->rollBack();
            return false;
        }
    }

    /**
     * 予定を削除する
     *
     * @param int $planId
     * @return bool
     */
    public function delete($planId) {
        $stmt = $this->pdo->prepare("DELETE FROM plans WHERE id = ?");
        return $stmt->execute([$planId]);
    }
}
```

#### 説明

- **add($tripId, $title, $description, $date)**: 指定した旅行IDに新しい予定を追加します。
- **getByTrip($tripId)**: 特定の旅行IDに関連する予定を取得し、日付順に並べます。
- **updateOrder($order)**: 予定の順序を更新します。`$order` は予定IDとその新しい順序を含む配列です。
- **delete($planId)**: 指定した予定IDに関連する予定を削除します。

これらのクラスを使うことで、旅行に関連する予定の管理が容易になります。コントローラとモデルの分離により、ビジネスロジックとデータ処理が整理され、コードの保守性が向上します。