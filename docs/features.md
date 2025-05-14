# CODLアプリケーションの機能

## 現在実装されている機能

現在のCODLアプリケーションはデモバージョンであり、基本的な機能のみが実装されています。以下に主要な機能を説明します。

### 1. 挨拶機能

アプリケーションの主要な機能として、ユーザーが入力した名前に対して挨拶文を返すシンプルなデモ機能が実装されています。この機能は、フロントエンド(Svelte)からバックエンド(Rust)への通信の基本的な実装例を示しています。

#### フロントエンド実装

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

<form class="row" onsubmit={greet}>
  <input id="greet-input" placeholder="Enter a name..." bind:value={name} />
  <button type="submit">Greet</button>
</form>
<p>{greetMsg}</p>
```

#### バックエンド実装

```rust
// greetコマンドの実装
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}! You've been greeted from Rust!", name)
}

// コマンドハンドラの登録
.invoke_handler(tauri::generate_handler![greet])
```

### 2. ウェブリンク機能

アプリケーションは、Tauri、Vite、SvelteKitの公式ウェブサイトへのリンクを提供しています。これらのリンクはフロントエンドで実装されており、`tauri-plugin-opener`プラグインによってシステムのデフォルトブラウザでリンクを開くことができます。

#### プラグインの初期化

```rust
// URLを外部ブラウザで開くプラグインを初期化
.plugin(tauri_plugin_opener::init())
```

#### フロントエンド実装

```svelte
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
```

### 3. クロスプラットフォーム対応

Tauriを使用することで、Windows、macOS、Linuxの主要なデスクトップ環境で動作するアプリケーションとなっています。これは特別な実装なしに、Tauriフレームワークの機能として提供されています。

## APIリファレンス

### バックエンドAPI (Rust)

現在のバージョンでは、以下のRust APIが実装されています。

#### greet

ユーザーの名前を受け取り、挨拶メッセージを返す関数です。

**シグネチャ:**
```rust
#[tauri::command]
fn greet(name: &str) -> String
```

**パラメータ:**
- `name` (string): 挨拶する相手の名前

**戻り値:**
- 挨拶メッセージを含む文字列

**例:**
```rust
// 入力: "Alice"
// 出力: "Hello, Alice! You've been greeted from Rust!"
```

### フロントエンドAPI

フロントエンドからバックエンドのRust関数を呼び出すには、Tauriの`invoke`関数を使用します。

#### invoke

**シグネチャ:**
```typescript
invoke<T>(command: string, args?: Record<string, unknown>): Promise<T>
```

**パラメータ:**
- `command` (string): 呼び出すRustコマンドの名前
- `args` (object, optional): コマンドに渡す引数のオブジェクト

**戻り値:**
- `Promise<T>`: コマンドの戻り値を含むPromise

**例:**
```typescript
// greetコマンドの呼び出し
const message = await invoke("greet", { name: "Alice" });
// message: "Hello, Alice! You've been greeted from Rust!"
```

## 今後の拡張ポイント

現在のCODLアプリケーションは基本的なデモ機能のみを提供していますが、以下のような機能拡張が可能です：

### 1. データ永続化機能

ローカルストレージやデータベースを使用したデータ永続化機能を追加できます。

```rust
// SQLiteデータベースを使用する例
#[tauri::command]
fn save_data(data: String) -> Result<(), String> {
    // データベースにデータを保存するロジック
    Ok(())
}
```

### 2. ファイルシステム操作

Tauriのファイルシステム操作機能を使用して、ファイルの読み書きを行うことができます。

```rust
use std::fs;

#[tauri::command]
fn read_file(path: String) -> Result<String, String> {
    fs::read_to_string(path).map_err(|e| e.to_string())
}

#[tauri::command]
fn write_file(path: String, content: String) -> Result<(), String> {
    fs::write(path, content).map_err(|e| e.to_string())
}
```

### 3. システム通知

システム通知を表示する機能を追加できます。

```rust
#[tauri::command]
fn show_notification(title: String, body: String) -> Result<(), String> {
    // システム通知を表示するロジック
    Ok(())
}
```

### 4. 多言語対応

国際化（i18n）サポートを追加して、多言語対応を実装できます。

```typescript
// フロントエンドでのi18nの例
import { createI18n } from 'vue-i18n'; // or similar library for Svelte

const i18n = createI18n({
  locale: 'ja',
  messages: {
    en: {
      greeting: 'Hello'
    },
    ja: {
      greeting: 'こんにちは'
    }
  }
});
```

## まとめ

CODLアプリケーションは現在、基本的なデモ機能のみを提供していますが、Tauriの豊富なAPIとSvelteKitの柔軟なフロントエンドフレームワークを活用することで、多様な機能拡張が可能です。今後の開発では、上記の拡張ポイントを参考に、アプリケーションの機能を充実させていくことが期待されます。
