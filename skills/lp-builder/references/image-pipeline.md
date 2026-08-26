# 画像パイプライン（取得・背景除去・検証）

## 1. フリー素材を取得する

Pexels / Unsplash の画像CDNから直接ダウンロードして同梱する（外部ホットリンクに依存しない）。
Pexels License / Unsplash License はいずれも商用利用可・帰属表示任意。

```bash
mkdir -p assets/img && cd assets/img
# 例：候補を複数落として目視で選ぶ（?w= で幅指定）
curl -s -o cand1.jpg "https://images.pexels.com/photos/<ID>/pexels-photo-<ID>.jpeg?auto=compress&cs=tinysrgb&w=1200"
```

- ダウンロード後は必ず Read で画像を開いて内容を確認する（IDだけで内容は分からない）。
- 題材に合う数枚に絞り、`hero.jpg` `about-*.jpg` `trust.jpg` 等に意味のある名前でリネーム。
- 出典を `assets/img/CREDITS.md` に表で残す（ファイル／用途／出典URL）。

## 2. 人物の背景を除去する（切り抜きヒーロー用）

`rembg`（u2net）で背景を透過PNGにする。初回はモデル(~176MB)をDLする。

```bash
pip install rembg onnxruntime pillow
python3 -c "
from rembg import remove; from PIL import Image
inp = Image.open('assets/img/hero.jpg')
Image.open('assets/img/hero.jpg')  # noop
out = remove(inp)
out.save('assets/img/hero-cut.png')
print('done', out.size)
"
```

確認のコツ：切り抜き結果を背景色に合成してプレビューし、縁のはみ出しや不自然な残りを見る。

```bash
python3 -c "
from PIL import Image
fg = Image.open('assets/img/hero-cut.png').convert('RGBA')
bg = Image.new('RGBA', fg.size, (245,133,26,255))   # 背景色に合わせる
bg.alpha_composite(fg); bg.convert('RGB').save('/tmp/cut-preview.jpg', quality=88)
"
```

- 切り抜き画像の下端は、CSS の mask-image で背景色にフェードさせると縁が自然になる（hero-patterns.md 参照）。
- 複雑な背景（室内など）でも人物は概ね綺麗に抜けるが、抜けが甘い場合は別カットに差し替える方が早い。

## 3. レンダリング検証（デスクトップ＋モバイル）

Playwright の Chromium で撮る。ブラウザ本体は `PLAYWRIGHT_BROWSERS_PATH=/opt/pw-browsers` に置かれる。
スクラッチパッドはセッション間で消えることがあるので、毎回セットアップから行うと確実：

```bash
cd <scratchpad>
npm init -y >/dev/null 2>&1
npm install playwright >/dev/null 2>&1
npx playwright install chromium 2>&1 | tail -1   # 本体が無い/版ずれのとき必要
```

トラブル対処：
- `Cannot find module 'playwright'` → `npm install playwright` をやり直す（node_modulesが消えている）。
- 起動時に "npx playwright install" バナー → 版に合う本体が無い。`npx playwright install chromium` を実行。
- **動画付きページは `networkidle` が発火しない**（ストリームが続くため）。`waitUntil:'load'`＋明示 `waitForTimeout` を使う。

```js
// shot.js
const { chromium } = require('playwright');
(async () => {
  const b = await chromium.launch();
  // reducedMotion:'reduce' にすると .reveal が即表示になり、
  // スクロール未発火で空白に見える問題を回避できる（本番CSSの reduced-motion 分岐を利用）
  for (const [name, vp] of [['desktop',{width:1280,height:900}],['mobile',{width:390,height:844}]]) {
    const ctx = await b.newContext({ viewport: vp, reducedMotion: 'reduce' });
    const p = await ctx.newPage();
    await p.goto('file:///ABSOLUTE/PATH/index.html', { waitUntil: 'networkidle' });
    await p.waitForTimeout(1200);
    await p.screenshot({ path: name + '.png', fullPage: true });
  }
  await b.close(); console.log('done');
})();
```

```bash
node shot.js   # → desktop.png / mobile.png を Read で目視
```

特定セクションだけ確認したいときは要素スクショ：
```js
const el = await p.$('.hero'); await el.screenshot({ path: 'hero.png' });
```
（position:fixed の要素は要素スクショに写り込むことがある＝実表示のバグではない点に注意）

### 見るポイント
- ヒーローの見出し・写真・数値の重なりが崩れていないか。
- モバイルで負マージンが効きすぎて顔/文字に重なっていないか（重なりは幅%指定に）。
- セクション背景色が交互になってリズムが出ているか。
- 横スクロール（body の横はみ出し）が発生していないか。フルブリード要素は親に overflow:hidden。
