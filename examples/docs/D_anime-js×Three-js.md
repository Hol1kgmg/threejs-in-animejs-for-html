# anime.js × three.js

**anime.js は v4.5 から three.js に対応しました。**

これによって、3D のものを動かすときも
**HTML の部品を動かすときとまったく同じ書き方**が使えます。

ここでは「anime.js を使うと、どれくらい楽に書けるようになるか」を、
**まったく同じ動きをする 2 つのファイル**で見比べます。

---

# 見比べる — `01` と `02`

| ファイル | 中身 |
| --- | --- |
| `01-without-animejs.html` | anime.js を**使わない**版 |
| `02-with-animejs.html` | anime.js を**使う**版 |

両方ブラウザで開いてください。
青い立方体が**左右に往復しながら回転し、薄くなったり濃くなったりします。**

**画面の動きは完全に同じです。違うのはコードだけです。**

## `01` anime.js を使わない

```js
const duration = 1500;

function easeInOutSine(t) {
  return -(Math.cos(Math.PI * t) - 1) / 2;
}

let startTime = null;
function animateLoop(timestamp) {
  if (startTime === null) startTime = timestamp;
  const elapsed = timestamp - startTime;

  const cyclePos = elapsed % (duration * 2);
  const t = cyclePos < duration
    ? cyclePos / duration
    : 1 - (cyclePos - duration) / duration;
  const p = easeInOutSine(t);

  mesh.position.x = -2 + 4 * p;
  mesh.rotation.y = p * Math.PI * 2;
  mesh.material.opacity = 0.7 + 0.3 * p;

  requestAnimationFrame(animateLoop);
}
requestAnimationFrame(animateLoop);
```

**約 20 行。** 読めなくて大丈夫です。
「**動かすには、これだけの計算を自分で書くことになる**」とだけ見てください。

## `02` anime.js を使う

```js
animate(mesh, {
  x: [-2, 2],
  rotateY: 360,
  opacity: [0.7, 1],
  duration: 1500,
  loop: true,
  alternate: true,
  ease: 'inOutSine',
});
```

**9 行。** しかも書いてあるのは、日本語にするとこれだけです。

> 立方体を、**位置 -2 から 2 へ**、**Y 軸まわりに 360 度**、**透明度 0.7 から 1 へ**、
> **1.5 秒かけて**、**ずっと繰り返し**、**行き帰り交互に**、**なめらかに加減速して**動かす。

`01` にあった計算は 1 つも書いていません。
**「どうやって動かすか」ではなく「どうなってほしいか」だけを書けばよくなります。**

---

# いちばん大事なこと：書き方が変わらない

`B_anime-jsとは.md` で見た、HTML の四角を動かすコードと並べてみます。

```js
// HTML の部品を動かす
animate('.box', {
  translateX: 250,
  duration: 1500,
  loop: true,
  alternate: true,
});
```

```js
// three.js の立体を動かす
animate(mesh, {
  x: [-2, 2],
  duration: 1500,
  loop: true,
  alternate: true,
});
```

**動かす対象が変わっただけで、書き方は同じです。**

そのおかげで、

- **2D で覚えたことが、そのまま 3D で使えます**
- **2D で書いたコードを、ほとんど書き換えずに 3D へ持っていけます**

これがこのハンズオンの締めくくりにつながります。
完成形の `98`（2D 版）と `99`（3D 版）を見比べると、
**魔法陣を動かしている部分がほとんど同じまま**であることが確認できます。

---

ここから先は補足です。ハンズオンを進めるうえで必ずしも覚える必要はありません。

---

# 補足 1：`01` で自分でやっていたこと

`01` の約 20 行が、それぞれ何をしていたのかの内訳です。

| 書いていた行 | やっていたこと |
| --- | --- |
| `requestAnimationFrame(animateLoop)` | 1/60 秒ごとに呼び出してもらう手配 |
| `elapsed = timestamp - startTime` | 開始から何ミリ秒経ったかを毎回数える |
| `cyclePos` と `t` の計算 | 今が「行き」か「帰り」かを判定して、進み具合を 0〜1 に直す |
| `easeInOutSine(t)` | なめらかな加減速を、三角関数の式で自作する |
| `mesh.position.x = ...` など 3 行 | 進み具合から「今あるべき位置・角度・透明度」を毎回計算して代入する |

細かい落とし穴もあります。

- 回転は**ラジアン**で書く必要があり、`Math.PI * 2` が 1 回転という知識が要る
- 速さを変えたいだけでも、計算式との整合を確認しないといけない
- 加減速のしかたを変えたければ、**数式そのものを書き換える**

# 補足 2：何が要らなくなったのか

| 要らなくなったもの | どうなったか |
| --- | --- |
| `requestAnimationFrame` の手配 | anime.js が内部で行う |
| 経過時間の計測 | 同上 |
| 「行き／帰り」の判定 | `alternate: true` の 1 行に |
| 繰り返しの処理 | `loop: true` の 1 行に |
| イージングの数式 | `ease: 'inOutSine'` という**名前を書くだけ**に |
| 位置・角度・透明度の毎フレーム計算 | ゴールの値を書くだけに |
| ラジアンへの変換 | `360` と**度数でそのまま**書けるように |

# 補足 3：準備は 2 行だけ

three.js 対応を使うために足すのは、次の 2 行です。

```js
import { animate } from 'animejs';
import 'animejs/adapters/three';
```

2 行目が **three.js adapter** です。
これを読み込むと、anime.js が three.js のオブジェクトを理解できるようになります。

読み込み先の指定（`importmap`）にも 2 行足しますが、書き写すだけです。

```html
<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.184.0/build/three.module.js",
    "animejs": "https://unpkg.com/animejs@4.5.0/dist/modules/index.js",
    "animejs/adapters/three": "https://unpkg.com/animejs@4.5.0/dist/modules/adapters/three/index.js"
  }
}
</script>
```

**それ以外に覚えることはありません。** インストール作業も不要です。

# 補足 4：まとめ

| | anime.js なし | anime.js あり |
| --- | --- | --- |
| 行数 | 約 20 行 | 9 行 |
| 書く内容 | どうやって動かすか（手順） | どうなってほしいか（結果） |
| 加減速の変更 | 数式を書き換える | 名前を差し替える |
| 速さの変更 | 計算式との整合を確認 | `duration` の数字を変える |
| 回転の単位 | ラジアン（`Math.PI * 2`） | 度（`360`） |
| 2D との書き方の違い | — | なし |

**anime.js は「動きの作り方」を書かなくて済むようにしてくれる道具です。**

> 2D 版と 3D 版で、実際に調整する数値がどう違うかは
> `F_ハンズオン+αの説明.md` で扱います。
