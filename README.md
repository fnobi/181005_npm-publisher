# なれる! <span>npm publisher</span>

## npmパッケージとは<span>（一応）</span>

- Node.jsでつかえるライブラリ。`npm instal xxx` で簡単にインストールできる。
- [npmjs.com](https://www.npmjs.com/)でユーザーさえ作れば、**誰でもパッケージを公開できる**。
  - 名前は早いもの勝ち・誰かが取ってしまった名前は別のパッケージではもう使えない（そうしない方法もある。後述。）
  - わりと自由なコミュニティなので[ときどき荒れたりする](https://yosuke-furukawa.hatenablog.com/entry/2016/03/27/152500)

## npmを公開すると<span>なにがいいの？</span>

- 社内外含め、**多くの人に使ってもらえるツールを作れる**（ユーザーがいるものを作る楽しさ）
- 客観的な便利さ・分かりやすさを持ったプログラムを書く訓練になる
- コマンドラインツールを作るときに、 `npm -g install xxx` でインストールできるよ！というのがやっぱり分かりやすい

## npm packageとして<span>つくれるもの</span>

- **CLI**
  - 「ターミナルから○○できたら便利だなあ…」が出発点
  - 案件用に書いた一括処理を切りだしてコマンド化する、とかもあり
  - のび作のものだと[koko](https://www.npmjs.com/package/koko)とか[axis-psd](https://www.npmjs.com/package/axis-psd)とか

- **プラグイン**
  - 特定のフレームワーク・ツールを拡張する目的でつくる
  - 例えばwebpackのloaderとかgulpのpluginとか
  - 単にNode.jsの知識だけではなく、どういうインタフェースを用意しなきゃいけないかなどフレームワーク側の理解が必要
  - のび作のものだと[gulp-mass-production](https://www.npmjs.com/package/gulp-mass-production)とか

- **Node.jsスクリプト用ヘルパー**
  - Node.jsのスクリプト中でrequireして使うことを想定するライブラリ
  - 一番普通っちゃ普通。この頃は結構みんなNodeスクリプトがりがり書いてるので、ここの需要が大きいのかも
  - のび作のものだと[simple-reproduction](https://www.npmjs.com/package/simple-reproduction)とか

## 公開するための事前準備

- **ユーザー作成**
  - [npmjs.com](https://www.npmjs.com/)でユーザーを作成
  - `npm login`

### おすすめツール

- **hub**
  - [hub.github.com](https://hub.github.com/)
  - githubのレポジトリをターミナルから扱う時に便利なコマンドを集めたやつ
  - 個人的には`hub create`だけ異様な頻度で使っている

## scoped modules

- 自分用の名前空間を区切ってパッケージを公開することもできる
  - [publish | npm Documentation](https://docs.npmjs.com/cli/publish)
  - [npmで名前空間を持ったモジュールを公開する方法(scoped modules) | Web Scratch](https://efcl.info/2015/04/30/npm-namespace/)
  - この場合は名前の重複を気にしなくて良い
  - ex) [@vue/cli](https://www.npmjs.com/package/@vue/cli), [@babel/preset-stage-3](https://www.npmjs.com/package/@babel/preset-stage-3)

### つかいかた

- package名を `{npmユーザ名}/{パッケージ名}` という形にする
- publish時に `npm publish` ではなく `npm publish --access=public` を使う
  - デフォルトでは、 `--access=restricted` という権限になっている（自分として認証しないとinstallできない）
  - restrictedは有料ユーザー専用機能

## 最速公開の手順

### ディレクトリの準備

- npmパッケージでファイルとして必要なのは、基本 `package.json` と `index.js`
- **npmとして公開するためのプロジェクトと、動作確認をするためのプロジェクトを分離してそれぞれ持っておく** と便利
  - パッケージのディレクトリを相対パスでrequireする

```
module.exports = function () {
  return \`
＿人人人人人人＿
＞　ぞい！！　＜
￣ＹＹＹＹＹＹ￣\`;
};
```

```
const createZoi = require('../../create-zoi');
console.log(createZoi());
```

### publishまで

- `git init`: レポジトリ新規作成
- `hub create`: github上にremote repo作成
- `npm init`: package.json作成
- `npm publish`: 公開！
- 動作確認用プロジェクトでnpm installできるか試してみる

### コマンドラインツールをつくる

- bin用の別ファイルを作成して、package.jsonでそのファイルのパスを書く
  - 実行可能ファイルとして設置する必要がある関係で、コードをここで直接ゴリゴリ書くのはあまりおすすめしない（shebangが邪魔）
- [https://docs.npmjs.com/files/package.json#bin](https://docs.npmjs.com/files/package.json#bin)

```
#!/usr/bin/env node
require('../cli.js');
```

## バージョンを上げる

```
# 下記を一気にやってくれる
# ・package.jsonのversionを更新(この場合1.0.0 => 1.1.0)
# ・それをコミット
# ・git上に同名のtagをつける
$ npm version minor

# いちおうgithub上の状態も揃えたいのでpushする
$ git push
$ git push --tags

# 再公開
$ npm publish
```

### SemVerについて

- [SemVer（セマンティックバージョニング）：Dev Basics／Keyword - ＠IT](http://www.atmarkit.co.jp/ait/articles/1612/06/news012.html)

- `1.2.3` → 左から **メジャーバージョン / マイナーバージョン / パッチバージョン**

#### どのバージョンを上げるべきか？
- バグ修正
  - → **パッチバージョンを上げる**
- 呼び出し方は変わらない（後方互換性あり）機能改善・追加機能
  - → **マイナーバージョンを上げる**
- 呼び出し方が変わる（後方互換性なし）大幅なアップデート
  - → **メジャーバージョンを上げる**
