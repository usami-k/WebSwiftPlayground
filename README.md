# Web Swift Playground

iOSDC Japan 2021 のパンフレット記事「[Webブラウザで動くSwift Playgroundを作ろう](https://fortee.jp/iosdc-japan-2021/proposal/f3342cf9-6d11-4693-b219-5b2c2b8b4ffd)」を実際にやってみた。

## はじめに

- 多くのプログラミング言語にはWebで動作するPlaygroundが提供されている。
- Swiftには公式のWeb Playgroundが提供されていない。
- サーバーサイドSwiftとWebプログラミングの技術を使って自分で作れる。

## MVP (Minimum Viable Product)

- 不特定多数が使うWebサービスを公開する、となるととても大変。
- ローカル環境でアクセスできるWebアプリケーションを作る。

## WebのUIを作る

- Webアプリケーションの世界にもUIフレームワークがある。
- 基本設計
    - 画面を左右に2分割する。コードエディタと結果表示。
    - コードを実行するRunボタンを置く。
- フレームワーク
    - TwitterのBootstrapを使用する。

### レイアウト

- `index.html`を作って、BootstrapのStarter Templateをコピーペースト。
    - [Getting Started](https://getbootstrap.com/docs/5.1/getting-started/introduction/)から。
- 記事のコードになるよう`title`と`body`を変更。
- ブラウザで表示確認。

### コードエディタ

- Swiftのシンタックスハイライトに対応したエディタコンポーネントを置く。
    - Ace：とっつきやすい。今回はこれを使う。
    - CodeMirror：バージョンアップで仕組みが変わったので注意。
    - Monaco Editor：高機能。やや難しい。
- エディタコンポーネントのロード。
    - ライブラリはCDN経由で配布されている。
    - [Embedding Ace in Your Site](https://ace.c9.io/#nav=embedding)のcdnjsからコピーペースト。
- エディタコンポーネントを配置。
    - 記事のコードのとおり。
- ブラウザで表示確認。

### 結果表示

- ターミナル風に表示するコンポーネントを置く。
    - Xterm.jsを使う。
- コンポーネントのロード。
    - [xterm CDN by jsDelivr](https://www.jsdelivr.com/package/npm/xterm)を使う。
    - 上記サイトのcssフォルダをクリックすると`xterm.css`が表示されるので、そこからコピーペースト。
    - `xterm.min.js`も同様にコピーペースト。
- コンポーネントを配置。
    - 記事のコードのとおり。
- ブラウザで表示確認。

## バックエンドのAPIを実装する

- バックエンド
    - コードエディタに入力されたコードを受け取り、実行結果を返す。
- 言語
    - サーバーサイドSwift
- フレームワーク
    - Vapor
    - 他はメンテナンスが停止しており、現在は事実上唯一の選択肢。

### プロジェクト作成

- VaporのCLIツールをインストール（Homebrew）。
- Vaporプロジェクト作成、Xcodeで開く。
    - [Hello, world](https://docs.vapor.codes/4.0/hello-world/)のとおり。
    - 依存するSwift Packageのダウンロードが行われるので待つこと。
- `routes.swift`を記事のコードのとおり実装する。
    - POSTで受け取ったコードを実行して標準出力に出すもの。
- Xcodeでビルド、実行。
- cURLで動作確認。
    - `curl -X POST "http://localhost:8080/run" -H 'Content-Type:application/json' --data '{"code":"print(\"Hello World!\")"}'`

### フロントエンドとの結合

- 記事では、GitHubのサンプルコードを参照とのこと。
    - https://github.com/kishikawakatsumi/swift-online-playground-tutorial
- 結合手順
    - バックエンド
        - POSTに対して、JSONでレスポンスを返す。
        - localhostに接続したときに`index.html`の内容を返す。
    - フロントエンド
        - Runボタンで入力コードをPOSTして、レスポンスで受け取った出力内容を表示する。

### JSONレスポンス

- `routes.swift`
    - POSTで返す型を`String`でなく`Content`準拠の型にする。
- cURLで動作確認。
    - JSONが返ってくればOK。

### HTMLを返す

- `Public`ディレクトリを作って`index.html`をそこに入れる。
- `configure.swift`
    - `Public`ディレクトリを使う。
- `routes.swift`
    - GETに対して`index.html`を返す。
- 動作確認
    - Xcodeではなく`swift run`で実行する。
    - ブラウザで`http://localhost:8080`を開く。
    - フロントエンドが表示できればOK。

### フロントエンドの対応

- `index.html`
    - Runボタンの処理をする`script`タグを追加する。
    - エディタの内容をPOSTする。
    - レスポンスから出力結果を取り出して結果表示エリアに出す。
- 動作確認
    - Xcodeではなく`swift run`で実行する。
    - ブラウザで`http://localhost:8080`を開く。
    - Swiftコードを書いてRunボタンを押し、結果が表示されればOK。

## サンドボックスで実行する

- 任意のコードが実行できるので公開しては危険。
- 実行環境を使い捨てのコンテナに閉じ込める。
