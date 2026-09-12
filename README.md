# threejs-in-animejs-for-html

[anime.js](https://animejs.com/) を使ったアニメーション表現を学ぶためのデモプロジェクトです。
`examples/codes/` 配下に、anime.js を使わない場合との比較や基本的な使い方、
応用的な表現のデモをHTMLファイル単位で分けて用意しています。

![anime.js のブラウザ動作デモ](./src/animejs-at-browser.gif)


| ファイル | 内容 |
| --- | --- |
| [00-introduction.html](./examples/codes/00-introduction.html) | anime.js を使った最小限のコード例。四角形が 1 つ動くだけのサンプル |
| [01-without-animejs.html](./examples/codes/01-without-animejs.html) | three.js のアニメーションを素の JavaScript（requestAnimationFrame・手動イージング）だけで実装した場合 |
| [02-with-animejs.html](./examples/codes/02-with-animejs.html) | 01 と同じアニメーションを anime.js（v4.5）の three.js adapter（`animate(mesh, {...})`）で簡潔に実装した場合 |
| [03-animejs-playground.html](./examples/codes/03-animejs-playground.html) | 検索フォームだけが置かれた自由記述用のファイル。JavaScript は空 |
| [04-magic-circle-handson.html](./examples/codes/04-magic-circle-handson.html) | 魔法陣の SVG と CSS だけが入っていて、動かす JS がコメントアウトされているハンズオン用ファイル。STEP 1 → 4 を順にコメント解除していく |
| [98-magic-circle-search.html](./examples/codes/98-magic-circle-search.html) | 04 の完成形（2D 版）。3 層の魔法陣が順に描かれ、描き終わった層から回転する検索フォーム |
| [99-magic-circle-3d.html](./examples/codes/99-magic-circle-3d.html) | 98 を 3D 空間に置いた版。CSS3DRenderer で 3 層を奥行き方向に並べ、空間ごと傾けている |

ハンズオン形式の説明資料は [examples/docs/](./examples/docs/) にあります。

このプロジェクトは、anime.js v4.5 で three.js に対応したこと（three.js adapter）を紹介する目的で作られています。

## 動作確認方法

ブラウザ上で`file://` のパスから開く

or

```bash
python3 -m http.server 8000
```

起動後、`http://localhost:8000/examples/codes/00-introduction.html` のようにアクセス

## 開発者向けセットアップ

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

```bash
mise trust && mise run setup
```

gitleaks / lefthook のインストールと Git フックの設定が一括で行われます。



## おまけ
three.jsをanime.jsのadapterで使ったバージョン

![anime.js × three.js のブラウザ動作デモ](./src/animejs-for-threejs-at-browser.gif)

