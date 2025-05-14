# CODLアプリケーションの構造

## ディレクトリ構造

CODLアプリケーションのプロジェクト構造は、Tauri + SvelteKitの標準的な構成に従っています。主要なディレクトリとその役割は以下の通りです：

```
codl/
├── docs/                    # プロジェクトドキュメント
├── src/                     # フロントエンドのソースコード (SvelteKit)
│   ├── app.html             # HTMLテンプレート
│   └── routes/              # SvelteKitのルーティング
│       ├── +layout.ts       # レイアウト関連のTypeScript
│       └── +page.svelte     # メインページのSvelteコンポーネント
│
├── src-tauri/               # Rustバックエンドのソースコード
│   ├── src/                 # Rustのソースファイル
│   │   ├── main.rs          # アプリケーションエントリポイント
│   │   └── lib.rs           # ライブラリコード
│   ├── Cargo.toml           # Rustの依存関係設定
│   ├── Cargo.lock           # 依存関係のロックファイル
│   ├── build.rs             # ビルドスクリプト
│   ├── tauri.conf.json      # Tauriの設定ファイル
│   ├── capabilities/        # Tauriのケイパビリティ設定
│   │   └── default.json     # デフォルトのケイパビリティ
│   ├── gen/                 # 生成コード
│   ├── icons/               # アプリケーションアイコン
│   └── target/              # Rustのビルド出力
│
├── static/                  # 静的アセット
│   ├── favicon.png          # ファビコン
│   ├── svelte.svg           # Svelteロゴ
│   ├── tauri.svg            # Tauriロゴ
│   └── vite.svg             # Viteロゴ
│
├── .gitignore               # Gitの除外リスト
├── package.json             # npm依存関係とスクリプト
├── package-lock.json        # npm依存関係のロックファイル
├── svelte.config.js         # Svelteの設定
├── tsconfig.json            # TypeScriptの設定
└── vite.config.js           # Viteの設定
```

## 主要ファイルの説明

### フロントエンド (SvelteKit)

#### src/app.html

HTML基本テンプレートファイルです。SvelteKitはこのファイルをベースにしてページをレンダリングします。

#### src/routes/+page.svelte

アプリケーションのメインページを定義するSvelteコンポーネントです。現在は、シンプルな挨拶機能を提供するデモUIを含んでいます。

```svelte
<script lang="ts">
  import { invoke } from "@tauri-apps/api/core";

  let name = $state("");
  let greetMsg = $state("");

  async function greet(event: Event) {
    event.preventDefault();
    // Tauriコマンドの呼び出し
    greetMsg = await invoke("greet", { name });
  }
</script>

<main class="container">
  <h1>Welcome to Tauri + Svelte</h1>
  
  <!-- ロゴ表示部分 -->
  <div class="row">
    <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo vite" alt="Vite Logo" />
    </a>
    <a href="https://tauri.app" target="_blank">
      <img src="/tauri.svg" class="logo tauri" alt="Tauri Logo" />
    </a>
    <a href="https://kit.svelte.dev" target="_blank">
      <img src="/svelte.svg" class="logo svelte-kit" alt="SvelteKit Logo" />
    </a>
  </div>
  
  <!-- 挨拶フォーム -->
  <form class="row" onsubmit={greet}>
    <input id="greet-input" placeholder="Enter a name..." bind:value={name} />
    <button type="submit">Greet</button>
  </form>
  <p>{greetMsg}</p>
</main>

<style>
  /* スタイル定義 (省略) */
</style>
```

#### src/routes/+layout.ts

SvelteKitのレイアウト設定を行うTypeScriptファイルです。ページ間の共通レイアウトや設定を管理します。

### バックエンド (Tauri + Rust)

#### src-tauri/src/main.rs

Rustアプリケーションのエントリポイントです。このファイルは非常にシンプルで、`lib.rs`で定義された機能を呼び出すだけです。

```rust
// Windows環境でのリリース時にコンソールウィンドウを表示しない設定
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

fn main() {
    // lib.rsで定義されたrun関数を呼び出す
    codl_lib::run()
}
```

#### src-tauri/src/lib.rs

バックエンドのメイン機能を提供するライブラリファイルです。Tauriアプリケーションの初期化、コマンド定義、プラグイン設定などが含まれています。

```rust
// greetコマンドの実装
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}! You've been greeted from Rust!", name)
}

// アプリケーションの実行関数
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        // URLを外部ブラウザで開くプラグインを追加
        .plugin(tauri_plugin_opener::init())
        // コマンドハンドラを登録
        .invoke_handler(tauri::generate_handler![greet])
        // Tauriアプリケーションを実行
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

#### src-tauri/Cargo.toml

Rustの依存関係と設定を管理するファイルです。パッケージメタデータ、クレート設定、依存クレートなどを定義します。

```toml
[package]
name = "codl"
version = "0.1.0"
description = "A Tauri App"
authors = ["you"]
edition = "2021"

[lib]
name = "codl_lib"
crate-type = ["staticlib", "cdylib", "rlib"]

[build-dependencies]
tauri-build = { version = "2", features = [] }

[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-opener = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

#### src-tauri/tauri.conf.json

Tauriアプリケーションの設定ファイルです。アプリケーション名、ウィンドウサイズ、ビルド設定、セキュリティポリシーなどを定義します。

```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "codl",
  "version": "0.1.0",
  "identifier": "com.codl.app",
  "build": {
    "beforeDevCommand": "npm run dev",
    "devUrl": "http://localhost:1420",
    "beforeBuildCommand": "npm run build",
    "frontendDist": "../build"
  },
  "app": {
    "windows": [
      {
        "title": "codl",
        "width": 800,
        "height": 600
      }
    ],
    "security": {
      "csp": null
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

#### src-tauri/capabilities/default.json

Tauriのケイパビリティ設定ファイルです。アプリケーションが許可する操作（ファイルアクセス、ネットワーク通信など）を定義します。

### フロントエンドの設定ファイル

#### package.json

npmパッケージとスクリプトの設定ファイルです。依存関係とビルド/開発スクリプトが定義されています。

```json
{
  "name": "codl",
  "version": "0.1.0",
  "description": "",
  "type": "module",
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview",
    "check": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json",
    "check:watch": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json --watch",
    "tauri": "tauri"
  },
  "license": "MIT",
  "dependencies": {
    "@tauri-apps/api": "^2",
    "@tauri-apps/plugin-opener": "^2"
  },
  "devDependencies": {
    "@sveltejs/adapter-static": "^3.0.6",
    "@sveltejs/kit": "^2.9.0",
    "@sveltejs/vite-plugin-svelte": "^5.0.0",
    "svelte": "^5.0.0",
    "svelte-check": "^4.0.0",
    "typescript": "~5.6.2",
    "vite": "^6.0.3",
    "@tauri-apps/cli": "^2"
  }
}
```

#### svelte.config.js

Svelteの設定ファイルです。アダプター、プラグイン、ビルドオプションなどを定義します。

#### tsconfig.json

TypeScriptのコンパイラ設定ファイルです。型チェック、コンパイルオプション、ソースマップなどの設定を行います。

#### vite.config.js

Viteのビルド設定ファイルです。プラグイン、ビルドオプション、開発サーバー設定などを定義します。

## Rustコードファイルの詳細解説

### main.rsの役割

`main.rs`はRustアプリケーションのエントリポイントです。Tauriの場合、このファイルはとてもシンプルで、主に以下の役割を持ちます：

1. Windowsでのコンソールウィンドウ表示制御
2. ライブラリ関数の呼び出し

特に `#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]` は、リリースビルド時にWindowsでコンソールウィンドウが表示されないようにするための設定です。

### lib.rsの役割

`lib.rs`はTauriアプリケーションの中心的な部分で、以下の重要な役割があります：

1. **コマンド定義**：
   フロントエンドから呼び出せるRust関数を定義します。`#[tauri::command]`アトリビュートで装飾された関数は、JavaScriptから呼び出し可能になります。

2. **アプリケーション初期化**：
   `run()`関数では、Tauriアプリケーションのビルダーパターンを使用して:
   - プラグインの初期化
   - コマンドハンドラの登録
   - アプリケーション実行コンテキストの生成
   を行います。

3. **エラー処理**：
   `.expect()`メソッドを使用して、アプリケーション実行中に発生するエラーをハンドリングします。

## ファイル間の関連性

### Rust側のファイル関連

1. **main.rs → lib.rs**：
   メインバイナリのエントリポイントからライブラリ関数を呼び出しています。

2. **lib.rs ↔ Cargo.toml**：
   `Cargo.toml`に定義された依存クレート（tauri、tauri-plugin-opener など）を`lib.rs`で使用しています。

3. **Rust ↔ tauri.conf.json**：
   Rustコードは`tauri::generate_context!()`を通じて`tauri.conf.json`の設定を読み込みます。

### フロントエンドとバックエンドの連携

1. **+page.svelte → lib.rs**：
   Svelteコンポーネントから`invoke()`関数を使用して、`lib.rs`に定義された`greet`コマンドを呼び出しています。

2. **tauri.conf.json → フロントエンド**：
   ウィンドウサイズや開発サーバーURLなど、フロントエンドの挙動に影響する設定が含まれています。

## ファイル拡張のパターン

CODLアプリケーションを拡張するためには、以下のパターンに従うことが推奨されます：

1. **新しいRustコマンドの追加**：
   `lib.rs`に新しいコマンド関数を追加し、`invoke_handler`に登録します。

   ```rust
   // 新しいコマンドの追加例
   #[tauri::command]
   fn calculate_sum(a: i32, b: i32) -> i32 {
       a + b
   }
   
   // 関数をinvoke_handlerに登録
   .invoke_handler(tauri::generate_handler![greet, calculate_sum])
   ```

2. **フロントエンドからの呼び出し**：
   Svelteコンポーネントから新しく追加したコマンドを呼び出します。

   ```typescript
   import { invoke } from "@tauri-apps/api/core";
   
   // 新しいコマンドの呼び出し
   const sum = await invoke("calculate_sum", { a: 5, b: 3 });
   ```

3. **新しいプラグインの追加**：
   `Cargo.toml`に依存関係を追加し、`lib.rs`でプラグインを初期化します。

   ```rust
   // プラグインの初期化
   .plugin(tauri_plugin_new_plugin::init())
