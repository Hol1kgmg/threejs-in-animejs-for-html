# anime.js v4.5 × three.js アダプター 対応範囲まとめ

このリポジトリで検証した、anime.js v4.5の公式three.jsアダプター (`animejs/adapters/three`) が
「何をカバーし、何をカバーしないか」の整理。「物理演算=自前の`requestAnimationFrame`ループ／
演出=anime.js」という役割分担を前提に、演出側でどこまでアダプターに任せられるかの判断材料とする。

## 前提バージョン

- `animejs`: 4.5.0以上
- `three.js`: 0.150.0以上（アダプターのpeer依存の下限）

## 導入方法

```js
import { animate } from 'animejs';
import 'animejs/adapters/three'; // 副作用インポート。これを読み込むと animate()/utils.set() が
                                  // three.jsオブジェクトを直接ターゲットにできるようになる
```

importmap経由でCDNから読み込む場合（本リポジトリの`examples/03-animejs-playground.html`で採用）:

```html
<script type="importmap">
{
  "imports": {
    "animejs": "https://unpkg.com/animejs@4.5.0/dist/modules/index.js",
    "animejs/adapters/three": "https://unpkg.com/animejs@4.5.0/dist/modules/adapters/three/index.js"
  }
}
</script>
```

## ユーザー操作・機能ベースの対応表

「どんなアプリを作るか」を設計する段階で、想定するユーザー操作・機能がanime.jsのthree.jsアダプター
だけで実現できるかを判断するための早見表。特定のアプリ（サイコロ運試し、コマ対戦など）に依存しない
汎用的な分類にしてあるので、新しいアプリ案を検討する際もまずここを参照する。各カテゴリの技術的な
裏付けは後述の「技術詳細」を参照。

| カテゴリ | 具体例（ユーザー操作起点） | 対応可否 | 理由 |
|---|---|---|---|
| A. イベント駆動の開始→終了が確定した演出 | ボタンクリックで回転開始／再クリックでスナップ停止、フェードイン登場 | ✅対応 | 開始値・終了値をトゥイーンで補間できるため |
| B. ドラッグして溜め、離して発射する操作 | 引っ張って発射、スライダーで力を調整 | △部分対応 | `Draggable`はDOM要素専用。透明なUIをドラッグさせ、値をthree.js側にコールバックで橋渡しする必要あり |
| C. 常時変化する物理演算 | 重力・転がり・衝突反発・摩擦減衰・フィールド外判定 | ❌非対応 | 毎フレーム動的に目標値が変わる／離散イベントで反応するため、自前`requestAnimationFrame`ループが必須 |
| D. クリック/ホバーへの反応演出 | 選択中オブジェクトのハイライト、色・不透明度変化 | ✅対応 | Raycasterで対象を特定後、`animate(material, {...})`で確定値へ補間するだけで済むため |
| E. カメラ・視点演出 | 起動時のカメラフライイン、注目時の寄り | ✅対応 | `camera`を直接ターゲットに`x/y/z`をアニメーション可能なため |
| F. 退場・削除演出 | 条件成立時のフェードアウト後に`scene.remove()` | ✅対応 | 削除前に`opacity`を0へ補間するだけで、削除判定自体（条件分岐）はロジック側の管轄 |

判断の目安: **「開始値と終了値を事前に計算・確定できるか」**が分かれ目。確定できるならA/D/E/Fのように
アダプターに任せられる。毎フレーム条件やその場の力学に応じて目標値が変わり続けるならC（自前ループ）
になる。Bはその中間で、UI操作の受け皿だけDraggable（DOM）に任せ、受け取った値をどう使うかは
A/CどちらのDOM/three.js側処理に渡すかで決まる。

## 技術詳細

以下は上記対応表の技術的な裏付け。プロパティ単位の実装詳細を確認したいときに参照する。

### 対応ターゲット

| 種類 | 例 |
|---|---|
| `Object3D`及びそのサブクラス | `Mesh`, `Light`, `Camera`, `Sprite`, `Points` |
| `Material` | `MeshStandardMaterial`など |
| `Texture` | `TextureLoader`で読み込んだテクスチャ |
| `Fog` / `FogExp2` | シーンのフォグ |
| `UniformNode`（TSL） | カスタムシェーダーのuniform |
| `Color` / `Vector2〜4` | 生のインスタンスも直接可 |

### プロパティの自動マッピング

meshなどのオブジェクトを直接`animate()`に渡すだけで、ネストしたプロパティへ自動的に振り分けられる。

```js
animate(mesh, {
  x: 100,          // mesh.position.x
  y: 50,           // mesh.position.y
  z: -30,          // mesh.position.z
  rotateX: 45,     // mesh.rotation.x（度数法で指定→内部でラジアン変換）
  rotateY: 90,
  rotateZ: 180,
  scale: 2,        // 均一スケール（x/y/z全てに反映）
  scaleX: 1.5,     // 軸ごとの個別指定も可能
  opacity: 0.5,    // mesh.material.opacity
  color: '#ff8800',// mesh.material.color（CSSカラー文字列、var(--x)も可）
  duration: 800,
  ease: 'outElastic(1, .8)',
});
```

- **角度の自動変換**: `rotate*`, `angle`など「角度」と認識される名前のプロパティは度数法で指定でき、
  内部でラジアンに変換される。
- **色**: HEX/RGB/HSL/CSSカスタムプロパティ文字列をそのまま渡せる（`material.color`, `scene.background`,
  `light.color`など）。
- **ベクトル系プロパティ**: `Vector2〜4`はX/Y/Z/Wサフィックスで分解して指定できる
  （例: `texture.offset` → `offsetX`, `offsetY`）。
- **シェーダーuniform**: `ShaderMaterial`のuniformを`uTime`, `uTint`のような名前で直接アニメーションでき、
  mesh経由の短縮記法も可能。
- **インスタンスメッシュ**: `getInstances(mesh)`で個々のインスタンスを取得し、`stagger()`と組み合わせて
  グリッド状に時間差アニメーションが可能。

### 本リポジトリでの実例

`examples/03-animejs-playground.html`のコマ登場演出:

```js
animate(top, {
  scale: 1, // top.scale.x/y/z に一括反映
  duration: 800,
  ease: 'outElastic(1, .8)',
});
```

### できないこと・向いていないこと（対応表カテゴリC）

anime.jsのエンジンは本質的に「開始値→終了値をduration/easingで補間するトゥイーンエンジン」であり、
以下のような**毎フレーム動的に目標値が変わる/離散的なイベントで反応する**処理には向かない。

| 処理 | 理由 |
|---|---|
| 重力による転がり（すり鉢の傾斜に沿った加速） | 現在位置における局所勾配から接線加速度を毎フレーム数値積分する必要があり、固定のtarget値を事前に決められない |
| 衝突判定・反発 | 「中心間距離 < 半径の和」を毎フレーム判定し、条件成立時に速度を瞬時に交換する離散イベント。トゥイーンで表現する対象ではない |
| 自転の摩擦減衰 | 角速度を保持しながら毎フレーム減衰させる連続的な状態更新で、A→Bの単純な補間ではない |
| フィールド外判定・自由落下・削除 | 位置に応じた条件分岐と、それ以降は物理演算に切り替わるため、固定targetのアニメーションと相性が悪い |

`utils.set()`を毎フレーム呼んで無理やり物理演算の結果を反映させることは技術的には可能だが、
`mesh.position`へ直接代入するのと実質同じであり、anime.jsのタイミング制御・イージングの恩恵はなく、
呼び出しオーバーヘッドが増えるだけなので採用しない。

### ユーザー操作を画面に反映させる場合（対応表カテゴリB）

「ユーザーがページ上で操作し、それを画面に反映する」条件下でも、Three.jsアダプター単体ではなく
anime.js v4.5の`Draggable`モジュールと組み合わせることで実現できる。

#### `Draggable`はDOM要素専用（three.jsアダプターとは独立した機能）

```js
import { createDraggable } from 'animejs';
const draggable = createDraggable(target, parameters);
```

- ドキュメント上、ターゲットは「CSSセレクタまたはDOM要素」に限定されており、**Three.jsのmeshを
  直接ドラッグ対象にすることはできない**（Three.jsアダプターが対応するのは`animate()`/`utils.set()`のみで、
  `Draggable`はそれとは別モジュール）。
- そのため、画面上に透明なDOM要素やUIハンドルを重ねて`createDraggable()`でドラッグ可能にし、
  ドラッグの値（角度・距離・離した瞬間の速度）を自分でコールバックから読み取り、three.js側
  （物理ループの初期値やアダプター経由の`animate()`）に橋渡しするコードが必要になる。

#### 主なオプション（ユーザー操作→画面反映に使えるもの）

| オプション | 用途 |
|---|---|
| `x` / `y` | ドラッグ可能な軸を制限 |
| `container` / `containerPadding` / `containerFriction` | ドラッグ範囲の境界と、境界内での摩擦 |
| `releaseMass` / `releaseStiffness` / `releaseDamping` | 離した後のバネ物理（慣性）の質感を調整 |
| `velocityMultiplier` / `minVelocity` / `maxVelocity` | 離した瞬間の速度をどう変換するか |
| `releaseEase` | 離した後の減衰カーブ |

#### このプロジェクトで想定される使い道

- **コマの「引き絞って発射」操作**: 透明なDOM要素をドラッグさせ、離した瞬間の速度・角度を
  `releaseMass`等のコールバックから取得し、コマの初期角速度・発射方向にマッピングする。
  ドラッグ〜発射までは操作演出（anime.js/Draggable管轄）、発射後の転がり・衝突は自前物理ループに
  ハンドオフする、という役割分担になる。
- **スタートボタン**: クリック/タップイベントをトリガーに、
  `animate(camera, {...})`や`animate(top, {...})`でカメラの寄りやコマの登場演出を発火してから
  物理ループを開始する。こちらはDraggable不要で、通常のDOMイベント+アダプターの組み合わせで足りる。
- **オブジェクトの選択ハイライト**: Raycasterでクリック/ホバー中のコマを特定し、選択中は
  `animate(material, { color: '#fff', opacity: ... })`のようなフィードバックをアダプター経由で返す。

#### 注意点

- 発射前（ドラッグ中）と発射直後の初速決定まではUI操作の範疇としてanime.js側で扱えるが、
  発射後の転がり・衝突・落下は前述の「できないこと」の通り自前ループの管轄のまま。
  Draggableの慣性はあくまで「離した後のDOM要素自体の動き」であり、three.js側の物理演算を
  代替するものではない点に注意。

## この プロジェクトでの適用方針

「物理演算=自前ループ／演出=anime.js」の役割分担を維持し、アダプターは以下のような
**「開始値と終了値が事前に決まっている演出」**（対応表カテゴリA/D/E/F）にのみ用いる。

| 用途 | カテゴリ | 対応状況 |
|---|---|---|
| コマ登場時のスケールイン | A | 実装済み（`top`を直接ターゲット） |
| カメラのフライイン（起動演出） | E | 未実装。`camera`を直接ターゲットに`x/y/z`をアニメーション可能 |
| スタート合図の点滅・カウントダウン演出 | A | 未実装。UIオブジェクトやmaterialの`opacity`/`color`で対応可能 |
| コマ削除時のフェードアウト | F | 未実装。削除条件成立時に`material.opacity`を0にアニメーションしてから`scene.remove()` |
| 物理演算本体（回転・重力・衝突・削除判定） | C | 対象外。自前の`requestAnimationFrame`ループのまま |
