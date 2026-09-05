# threejs-in-animejs-for-html

[anime.js](https://animejs.com/) を使ったアニメーション表現を学ぶためのデモプロジェクトです。
`examples/` 配下に、anime.js を使わない場合との比較や基本的な使い方、
応用的な表現のデモをHTMLファイル単位で分けて用意しています。

| ファイル | 内容 |
| --- | --- |
| [examples/00-introduction.html](./examples/00-introduction.html) | anime.js を使った最小限のコード例 |
| [examples/01-without-animejs.html](./examples/01-without-animejs.html) | 同じ three.js アニメーションを素の JavaScript（requestAnimationFrame・手動イージング）だけで実装した場合 |
| [examples/02-with-animejs.html](./examples/02-with-animejs.html) | 01と同じアニメーションを anime.js（v4.5）の three.js adapter（`animate(mesh, {...})`）で簡潔に実装した場合 |
| [examples/03-animejs-playground.html](./examples/03-animejs-playground.html) | anime.js を使った応用的な表現（作成予定） |

このプロジェクトは、anime.js v4.5 で three.js に対応したこと（three.js adapter）を紹介する目的で作られています。

## 動作確認方法

`examples/` 配下のファイルは ES Modules と importmap を使用しているため、
`file://` で直接開くと動作しません（ブラウザのセキュリティ制約により import が失敗します）。
必ずローカルサーバー経由で開いてください。

```bash
python3 -m http.server 8000
```

起動後、`http://localhost:8000/examples/00-introduction.html` のようにアクセスしてください。

## 前提条件

[mise](https://mise.jdx.dev/) がインストールされ、シェルに統合されていること。

macOS (Homebrew):

```bash
brew install mise
```

シェル統合 (zsh):

```bash
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc
source ~/.zshrc
```

## セットアップ

```bash
mise trust && mise run setup
```

gitleaks / lefthook のインストールと Git フックの設定が一括で行われます。

