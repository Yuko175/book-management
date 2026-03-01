# Book Management API

書籍と著者の管理を行うREST APIアプリケーション。Kotlin、Spring Boot、jOOQ、PostgreSQLを使用したAPI。

## 概要

Book Management APIは、以下の機能を提供します：

- **著者管理**: 著者の作成・更新機能
- **書籍管理**: 書籍の作成・更新機能および著者IDによる検索
- **データベース管理**: Flywayによるスキーマ自動マイグレーション
- **型安全なクエリ**: jOOQを使用したコンパイルタイム型安全性
- **バリデーション**: Spring Validationによる入力値検証
- **エラーハンドリング**: グローバルな例外ハンドラ

## 技術スタック

### 言語・フレームワーク

- **言語**: Kotlin 2.2.21
- **フレームワーク**: Spring Boot 4.0.3
- **Java バージョン**: JDK 21

### データベース

- **DBMS**: PostgreSQL（最新版）
- **マイグレーション**: Flyway
- **ORM/Query Builder**: jOOQ 3.19.30

### テスト・カバレッジ

- **テストフレームワーク**: JUnit 5
- **モックライブラリ**: Mockito
- **カバレッジ測定**: JaCoCo 0.8.12

### その他ライブラリ

- Spring Boot Starter Web
- Spring Boot Starter Validation
- Spring Boot Starter jOOQ
- Jackson Module for Kotlin

## プロジェクト構造

```
book-management/
├── src/
│   ├── main/
│   │   ├── kotlin/io/github/gonzily1269/book_management/
│   │   │   ├── BookManagementApplication.kt
│   │   │   ├── controller/          # REST APIエンドポイント
│   │   │   │   ├── BookController.kt
│   │   │   │   └── AuthorController.kt
│   │   │   ├── service/             # ビジネスロジック
│   │   │   │   ├── BookService.kt
│   │   │   │   └── AuthorService.kt
│   │   │   ├── repository/          # データアクセス層
│   │   │   ├── dto/                 # データ転送オブジェクト
│   │   │   └── exception/           # カスタム例外
│   │   └── resources/
│   │       ├── application.properties
│   │       └── db/migration/        # Flywayマイグレーションスクリプト
│   └── test/kotlin/io/github/gonzily1269/book_management/
│       ├── controller/              # コントローラテスト
│       ├── service/                 # サービステスト
│       ├── repository/              # リポジトリテスト
│       └── exception/               # 例外ハンドラテスト
├── build.gradle                     # Gradle設定
├── compose.yaml                     # Docker Composeファイル
└── README.md                        # このファイル
```

## セットアップ

### 前提条件

- JDK 21以上
- Docker Desktop

### インストール

1. リポジトリをクローン

```bash
git clone https://github.com/gonzily1269/book-management.git
cd book-management
```

2. Dockerを使用してPostgreSQLを起動

```bash
docker-compose up -d
```

3. JOOQコード生成を実行

cleanタスクを実行

```bash
./gradlew clean
```

JOOQコード生成タスクを実行

```bash
./gradlew generateJooq
```

buildタスクを実行

```bash
./gradlew build
```

4. アプリケーションをビルド・実行

```bash
./gradlew bootRun
```

アプリケーションはデフォルトで `http://localhost:8080` で起動します。

## API エンドポイント

### 著者 API

#### 著者を作成

```http
POST /api/authors
Content-Type: application/json

{
  "name": "太郎 山田",
  "birthDate": "1980-01-15"
}
```

**レスポンス** (201 Created):

```json
{
  "id": 1,
  "name": "太郎 山田",
  "birthDate": "1980-01-15"
}
```

#### 著者を更新

```http
PUT /api/authors/{id}
Content-Type: application/json

{
  "name": "太郎 山田",
  "birthDate": "1980-01-15"
}
```

**レスポンス** (200 OK または 404 Not Found):

```json
{
  "id": 1,
  "name": "太郎 山田",
  "birthDate": "1980-01-15"
}
```

### 書籍 API

#### 書籍を作成

```http
POST /api/books
Content-Type: application/json

{
  "title": "Kotlin入門",
  "price": 3500,
  "publicationStatus": "出版済み",
  "authorIds": [1, 2]
}
```

**レスポンス** (201 Created):

```json
{
  "id": 1,
  "title": "Kotlin入門",
  "price": 3500,
  "publicationStatus": "出版済み",
  "authorIds": [
    1,
    2
  ]
}
```

#### 書籍を更新

```http
PUT /api/books/{id}
Content-Type: application/json

{
  "title": "Kotlin完全ガイド",
  "price": 4500,
  "publicationStatus": "出版済み",
  "authorIds": [1, 2, 3]
}
```

**レスポンス** (200 OK または 404 Not Found):

```json
{
  "id": 1,
  "title": "Kotlin完全ガイド",
  "price": 4500,
  "publicationStatus": "出版済み",
  "authorIds": [
    1,
    2,
    3
  ]
}
```

#### 著者IDで書籍を検索

```http
GET /api/books/by-author/{authorId}
```

**レスポンス** (200 OK):

```json
[
  {
    "id": 1,
    "title": "Kotlin入門",
    "price": 3500,
    "publicationStatus": "出版済み",
    "authorIds": [
      1,
      2
    ]
  }
]
```

## ビジネスロジック制約

### 書籍に関するルール

- **タイトル**: 必須項目。
- **価格**: 必須項目。0以上である必要があります
- **出版状況**: 必須項目。出版済みの書籍を未出版状態に変更することはできません
- **著者**: 必須項目。書籍には最低1人以上の著者が必須です(複数の著者を持つことが可能です)

### 著者に関するルール

- **著者名**: 必須項目。
- **生年月日**: 必須項目。現在日以前である必要があります

## テスト

### テスト実行

```bash
./gradlew test
```

### テストカバレッジレポート生成

```bash
./gradlew jacocoTestReport
```

カバレッジレポートは以下の場所に生成されます:

```
build/reports/jacoco/test/html/index.html
```

## データベーススキーマ

マイグレーションスクリプトは以下に配置されています:

```
src/main/resources/db/migration/V1__create_tables.sql
```

アプリケーション起動時に、Flywayが自動的にマイグレーションを実行し、テーブルを作成します。