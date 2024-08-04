!!! info
    PHPとSQLを使って、旅行の計画を自由自在に細かくドラックアンドドロップで組むことのでき、メモや写真も1つの予定ごとに追加できるページを作って

PHPとSQLを使って、旅行の計画をドラッグ＆ドロップで組むことができるページを作成するには、以下のような手順を考慮する必要があります。

### 1. **データベース設計**

まず、旅行の計画を管理するためのデータベースを設計します。以下のテーブルが必要になるでしょう。

- **trips**: 旅行全般の情報を保存
  - id (INT, PK)
  - name (VARCHAR)
  - description (TEXT)
  - start_date (DATE)
  - end_date (DATE)
  
- **plans**: 各旅行に対する個別の計画
  - id (INT, PK)
  - trip_id (INT, FK)
  - title (VARCHAR)
  - description (TEXT)
  - date (DATE)
  - time (TIME)

- **photos**: 写真情報
  - id (INT, PK)
  - plan_id (INT, FK)
  - file_path (VARCHAR)

- **notes**: メモ情報
  - id (INT, PK)
  - plan_id (INT, FK)
  - content (TEXT)

### 2. **PHPとSQLによるバックエンド実装**

- **データの取得**: PHPスクリプトを作成して、旅行プランやメモ、写真などのデータをデータベースから取得します。

  ```php
  // データベース接続
  $pdo = new PDO('mysql:host=localhost;dbname=travel_planner', 'username', 'password');
  
  // 旅行プランの取得
  $stmt = $pdo->prepare('SELECT * FROM plans WHERE trip_id = ? ORDER BY date, time');
  $stmt->execute([$trip_id]);
  $plans = $stmt->fetchAll(PDO::FETCH_ASSOC);
  ```

- **データの追加・更新・削除**: ユーザーが新しいプランやメモ、写真を追加したり、既存のデータを更新したりするためのPHPスクリプトを作成します。

  ```php
  // プランの追加
  $stmt = $pdo->prepare('INSERT INTO plans (trip_id, title, description, date, time) VALUES (?, ?, ?, ?, ?)');
  $stmt->execute([$trip_id, $title, $description, $date, $time]);
  ```

### 3. **フロントエンドの実装**

- **ドラッグ＆ドロップ**: JavaScriptライブラリ（例: jQuery UI、Sortable.js）を使って、計画をドラッグ＆ドロップで移動できるようにします。

  ```html
  <ul id="sortable">
    <li class="ui-state-default">Plan 1</li>
    <li class="ui-state-default">Plan 2</li>
  </ul>

  <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
  <script>
    $(function() {
      $("#sortable").sortable();
      $("#sortable").disableSelection();
    });
  </script>
  ```

- **メモと写真の管理**: 各プランに対してメモや写真を追加するインターフェースを作成します。

  ```html
  <form action="upload.php" method="post" enctype="multipart/form-data">
    <input type="file" name="photo">
    <input type="submit" value="Upload">
  </form>

  <textarea name="note"></textarea>
  <button onclick="saveNote()">Save Note</button>
  ```

### 4. **ユーザーインターフェース**

- **プラン表示**: PHPとHTMLを組み合わせて、旅行プランを一覧表示します。
- **メモ・写真の追加**: プランに関連するメモや写真を追加するためのフォームを提供します。
- **ドラッグ＆ドロップ**: JavaScriptを使って、プランの順序を変更できるようにします。

### 5. **セキュリティとバリデーション**

- **ユーザー入力のサニタイズ**: SQLインジェクションやクロスサイトスクリプティング（XSS）を防ぐために、ユーザーからの入力をサニタイズします。
- **ファイルアップロードの検証**: アップロードするファイルのタイプやサイズを制限し、セキュリティリスクを軽減します。

このように、PHPとSQLを使って旅行の計画を作成し、メモや写真を管理するページを実装することができます。フロントエンドとバックエンドの統合が重要であり、ユーザー体験を高めるための工夫が必要です。

!!! info
    全てをファイルにして

以下に、旅行計画を管理するための簡単なウェブアプリケーションのコード例を示します。この例では、PHPとMySQLを使ってバックエンドを構築し、HTMLとJavaScriptを使ってフロントエンドを実装します。

### 1. **データベースのセットアップ**

まず、MySQLで以下のスクリプトを実行してデータベースとテーブルを作成します。

```sql
CREATE DATABASE travel_planner;

USE travel_planner;

CREATE TABLE trips (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    start_date DATE,
    end_date DATE
);

CREATE TABLE plans (
    id INT AUTO_INCREMENT PRIMARY KEY,
    trip_id INT,
    title VARCHAR(255),
    description TEXT,
    date DATE,
    time TIME,
    FOREIGN KEY (trip_id) REFERENCES trips(id)
);

CREATE TABLE photos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    plan_id INT,
    file_path VARCHAR(255),
    FOREIGN KEY (plan_id) REFERENCES plans(id)
);

CREATE TABLE notes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    plan_id INT,
    content TEXT,
    FOREIGN KEY (plan_id) REFERENCES plans(id)
);
```

### 2. **PHPスクリプト**

#### `db.php` - データベース接続

```php
<?php
$host = 'localhost';
$db = 'travel_planner';
$user = 'root';
$pass = '';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}
?>
```

#### `index.php` - メインページ

```php
<?php
include 'db.php';

// データ取得
$stmt = $pdo->query('SELECT * FROM trips');
$trips = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Travel Planner</title>
    <link rel="stylesheet" href="styles.css">
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
    <script src="script.js"></script>
</head>
<body>
    <h1>Travel Planner</h1>
    <ul id="trip-list">
        <?php foreach ($trips as $trip): ?>
            <li data-trip-id="<?php echo $trip['id']; ?>">
                <?php echo htmlspecialchars($trip['name']); ?>
                <button onclick="loadPlans(<?php echo $trip['id']; ?>)">Load Plans</button>
            </li>
        <?php endforeach; ?>
    </ul>

    <div id="plan-container"></div>
</body>
</html>
```

#### `load_plans.php` - プランの読み込み

```php
<?php
include 'db.php';

$trip_id = $_GET['trip_id'];

$stmt = $pdo->prepare('SELECT * FROM plans WHERE trip_id = ? ORDER BY date, time');
$stmt->execute([$trip_id]);
$plans = $stmt->fetchAll(PDO::FETCH_ASSOC);

echo json_encode($plans);
?>
```

#### `upload.php` - 写真のアップロード

```php
<?php
include 'db.php';

if ($_FILES['photo']['error'] == UPLOAD_ERR_OK) {
    $file_path = 'uploads/' . basename($_FILES['photo']['name']);
    move_uploaded_file($_FILES['photo']['tmp_name'], $file_path);

    $plan_id = $_POST['plan_id'];
    $stmt = $pdo->prepare('INSERT INTO photos (plan_id, file_path) VALUES (?, ?)');
    $stmt->execute([$plan_id, $file_path]);

    echo 'Upload successful';
} else {
    echo 'Upload failed';
}
?>
```

### 3. **フロントエンド**

#### `styles.css` - スタイルシート

```css
body {
    font-family: Arial, sans-serif;
}

#trip-list {
    list-style-type: none;
}

#trip-list li {
    margin: 10px 0;
}

#plan-container {
    margin-top: 20px;
}
```

#### `script.js` - JavaScript

```javascript
function loadPlans(tripId) {
    $.getJSON('load_plans.php', { trip_id: tripId }, function(plans) {
        let html = '<ul id="sortable">';
        plans.forEach(function(plan) {
            html += '<li class="ui-state-default" data-plan-id="' + plan.id + '">' +
                    plan.title + 
                    '<button onclick="addPhoto(' + plan.id + ')">Add Photo</button>' +
                    '<button onclick="addNote(' + plan.id + ')">Add Note</button>' +
                    '</li>';
        });
        html += '</ul>';

        $('#plan-container').html(html);
        $('#sortable').sortable();
        $('#sortable').disableSelection();
    });
}

function addPhoto(planId) {
    let formData = new FormData();
    formData.append('photo', $('#photo')[0].files[0]);
    formData.append('plan_id', planId);

    $.ajax({
        url: 'upload.php',
        type: 'POST',
        data: formData,
        contentType: false,
        processData: false,
        success: function(response) {
            alert(response);
        }
    });
}

function addNote(planId) {
    let note = prompt('Enter your note:');
    if (note) {
        $.post('add_note.php', { plan_id: planId, content: note }, function(response) {
            alert(response);
        });
    }
}
```

#### `add_note.php` - メモの追加

```php
<?php
include 'db.php';

$plan_id = $_POST['plan_id'];
$content = $_POST['content'];

$stmt = $pdo->prepare('INSERT INTO notes (plan_id, content) VALUES (?, ?)');
$stmt->execute([$plan_id, $content]);

echo 'Note added successfully';
?>
```

### 4. **ファイルのディレクトリ構造**

```
/travel-planner
    /uploads
    db.php
    index.php
    load_plans.php
    upload.php
    add_note.php
    styles.css
    script.js
```

### 注意点
- このコードは基本的な機能を実装したものであり、セキュリティ対策（例: ユーザー認証、入力のサニタイズ、ファイルの検証）が十分ではありません。
- 実際の運用には、セキュリティ対策やエラーハンドリング、ユーザーインターフェースの改善が必要です。

これで、旅行計画を作成・管理するための基本的なウェブアプリケーションが完成です。
