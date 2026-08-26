# ビルドガイド（デザインシステム＋共通部品の統合スキャフォールド）

このセッションで作った複数のLP（採用支援系・旅行系など）で共通して使った土台をまとめたもの。
新規LPはここのトークンと部品をコピーして、配色・フォント・コピーだけ差し替えると速く高品質に組める。
**単一の自己完結HTML**が前提（CSS/JSはインライン、外部はWebフォントのみ、画像・動画は同梱）。

## 1. デザインを先に決める（トークン設計）

実装前に必ずパレットとフォントを確定させ、`:root` のカスタムプロパティに落とす。以降の色は
すべてこの変数から導出する（途中で色を作り直さない）。量産AI顔（白地＋青見出し＋オレンジ丸ボタン）は避け、
題材の世界観から独自の配色・書体を選ぶ。

```css
:root {
  /* 色は「地・文字・主役・差し色」を最小構成に。必要なら淡色/濃色を足す */
  --ground:#FBF6EC;         /* 背景（紙白・淡色） */
  --ground-2:#F3ECDC;       /* 交互バンド用 */
  --ink:#21344B;            /* 本文（ほぼ黒の有彩色） */
  --muted:#5E7085;          /* 補助テキスト */
  --accent:#3E9FE0;         /* 主役色（ブランド） */
  --accent-deep:#2A7FC0;    /* 濃い版（hover/文字） */
  --accent-2:#FF6E5B;       /* 差し色（CTA/強調）。主役と補色〜近似で1色 */
  /* CTAをグラデにするなら */
  --cta-from:#2C6BEC; --cta-to:#8A36D9;

  --maxw:1120px; --radius:16px;
  --shadow-sm:0 1px 2px rgba(30,50,75,.05), 0 12px 30px rgba(30,50,75,.08);
  --shadow-md:0 18px 50px rgba(30,50,75,.16);

  /* フォントは2〜3役。見出し＝個性のあるJPゴシック、本文＝Noto Sans JP、数値＝Outfit */
  --display:"Zen Kaku Gothic New", system-ui, sans-serif;  /* 親しみ系なら "Zen Maru Gothic" */
  --body:"Noto Sans JP", system-ui, sans-serif;
  --num:"Outfit", system-ui, sans-serif;
}
```

フォント読み込み（例）：
```html
<link href="https://fonts.googleapis.com/css2?family=Zen+Kaku+Gothic+New:wght@500;700;900&family=Noto+Sans+JP:wght@400;500;700&family=Outfit:wght@600;700;800&display=swap" rel="stylesheet" />
```

トーン別のフォント指針：
- 信頼・堅実（BtoB/医療/士業）→ 見出し `Zen Kaku Gothic New`
- 親しみ・ポップ・旅/観光 → 見出し `Zen Maru Gothic`（角丸）
- 数値・英字・実績 → `Outfit`（800で極太に）

## 2. リセット＆土台

```css
* { box-sizing:border-box; }
html { scroll-behavior:smooth; }
body { margin:0; font-family:var(--body); color:var(--ink); background:var(--ground); line-height:1.85; -webkit-font-smoothing:antialiased; }
h1,h2,h3,h4 { font-family:var(--display); margin:0; line-height:1.4; font-weight:700; }
p{margin:0;} a{color:inherit;text-decoration:none;} img{max-width:100%;display:block;} ul{margin:0;padding:0;list-style:none;}
.wrap { width:100%; max-width:var(--maxw); margin:0 auto; padding-inline:22px; }
:focus-visible { outline:3px solid var(--accent-2); outline-offset:2px; border-radius:4px; }
```

## 3. セクションの色リズム

隣接セクションで同じ背景色を続けない。地→淡色バンド→地→濃色帯…と交互にしてリズムを作る。
見出しは共通の `.s-head`（eyebrow＋H2＋リード）で統一する。

```css
.section { padding-block:clamp(58px,8vw,104px); }
.section--alt { background:var(--ground-2); }          /* 交互バンド */
.section--tint { background:linear-gradient(180deg,#EAF5FD,#F4FAFE); }
.s-head { max-width:760px; margin-bottom:clamp(30px,4.5vw,50px); }
.s-head h2 { font-size:clamp(1.7rem,4.4vw,2.7rem); font-weight:900; margin-top:12px; }
.s-head h2 .u { background:linear-gradient(transparent 62%, var(--accent-2) 62%); } /* マーカー風強調 */
.s-head p { color:var(--muted); margin-top:14px; }
.eyebrow { display:inline-flex; align-items:center; gap:8px; font-family:var(--num); font-weight:700; letter-spacing:.14em; font-size:.8rem; text-transform:uppercase; color:var(--accent-deep); }
```

## 4. ボタン（グラデ／単色／白／ゴースト＋丸アロー）

```css
.btn { display:inline-flex; align-items:center; gap:10px; font-family:var(--display); font-weight:700; font-size:1.02rem; padding:15px 26px; border-radius:999px; cursor:pointer; border:2px solid transparent; transition:transform .18s, box-shadow .18s, background .18s; }
.btn .arr { font-family:var(--num); font-weight:800; }
.btn--lg { padding:18px 34px; font-size:1.1rem; }
.btn--grad { background:linear-gradient(95deg,var(--cta-from),var(--cta-to)); color:#fff; box-shadow:0 10px 26px rgba(95,60,210,.4); }
.btn--accent { background:var(--accent-2); color:#fff; box-shadow:0 10px 24px rgba(240,85,63,.34); }
.btn--white { background:#fff; color:var(--accent-deep); box-shadow:var(--shadow-sm); }
.btn--ghost { background:#fff; color:var(--accent-deep); border-color:var(--accent); }
.btn:hover { transform:translateY(-2px); }
/* 丸アロー（グラデ/色ボタン内） */
.btn .arrow { width:28px; height:28px; border-radius:50%; display:grid; place-items:center; background:rgba(255,255,255,.28); font-family:var(--num); font-weight:800; }
```

## 5. よく使う部品

- **スティッキーヘッダー / タブナビ**：`position:sticky; top:0; backdrop-filter:blur()` の半透明バー。左にロゴ、中央にアンカーリンク（旅行系は 01〜04 のタブ）、右にCTA。`@media(max-width:860px)` でリンクを隠しCTAだけ残す。
- **カード**：`.card`（画像 `aspect-ratio:4/3` ＋ body）。hoverで `translateY(-4px)` と画像 `scale(1.06)`。
- **統計/数値**：`.num { font-family:var(--num); font-weight:800; }` を大きく。ヒーローや実績で `data-count` カウントアップ（§8）。
- **料金表**：`.price-card`（濃色ヘッダー＋点線区切りの行）。金額は数値フォント。
- **利用フロー**：本当に順序があるときだけ番号付き（丸数字＋縦線）。
- **FAQ**：ネイティブ `<details>/<summary>`。`summary::-webkit-details-marker{display:none}` にして＋/×アイコンを自作、`[open]` で回転。
- **導入事例**：引用＋数値＋人物（写真が無ければイニシャルの色丸アバター）。
- **最終CTA**：濃色/グラデ背景＋大見出し＋ぼかしblob装飾。
- **モバイル固定CTA**：`position:fixed; bottom:0` のバー。`@media(max-width:860px)` で表示、`body{padding-bottom}` で被り防止。

```css
.mcta { position:fixed; left:0; right:0; bottom:0; z-index:60; background:rgba(255,255,255,.96); backdrop-filter:blur(8px); border-top:1px solid var(--line); padding:11px 16px; display:none; }
.mcta .btn { width:100%; }
@media (max-width:860px){ .mcta{display:block;} body{padding-bottom:78px;} }
```

## 6. ヒーロー

構図の選択と実装は `hero-patterns.md` を参照：
- A 切り抜き人物×背景色（フルブリード＋下端フェード、%オーバーラップ）
- B 極太の大型数値（カウントアップ）
- C 写真カード横並び（信頼型）
- D 背景動画／動画差し込み（`autoplay muted loop playsinline`＋poster＋veil）
- 液体が揺れるWebGLヒーロー（炭酸/飲料系）は別スキル **carbonated-kv** に完全プロンプトあり。併用可。

## 7. 装飾

`decorative-motifs.md` から題材のトーンに合うものを選ぶ（バルーンテキスト／ウォータードロップ／
ドゥードゥル／市松模様／ホログラム／コンフェッティ／モーションブラー）。**同時に強く出すのは最大2種**。

## 8. モーション（控えめに、必ず抑制分岐）

スクロールリビール＋数値カウントアップが定番。動きは1〜2箇所に集中させる。
`prefers-reduced-motion: reduce` で必ず止める。

```css
.reveal { opacity:0; transform:translateY(22px); transition:opacity .6s, transform .6s; }
.reveal.in { opacity:1; transform:none; }
@media (prefers-reduced-motion:reduce){ html{scroll-behavior:auto;} .reveal{opacity:1;transform:none;transition:none;} }
```
```js
// reveal
(function(){var els=document.querySelectorAll('.reveal');
 if(!('IntersectionObserver'in window)||matchMedia('(prefers-reduced-motion: reduce)').matches){els.forEach(e=>e.classList.add('in'));return;}
 var io=new IntersectionObserver(es=>es.forEach(x=>{if(x.isIntersecting){x.target.classList.add('in');io.unobserve(x.target);}}),{threshold:.12});
 els.forEach(e=>io.observe(e));})();
// count-up: data-count を持つ要素をビューポート進入時に 0→目標
(function(){var r=matchMedia('(prefers-reduced-motion: reduce)').matches;
 var io=new IntersectionObserver(es=>es.forEach(x=>{if(x.isIntersecting){run(x.target);io.unobserve(x.target);}}),{threshold:.5});
 document.querySelectorAll('[data-count]').forEach(n=>io.observe(n));
 function run(el){var t=parseFloat(el.getAttribute('data-count'));if(r||!t){el.textContent=t;return;}
  var s=null;requestAnimationFrame(function tick(ts){if(!s)s=ts;var p=Math.min((ts-s)/1500,1);el.textContent=Math.round(t*(1-Math.pow(1-p,3)));if(p<1)requestAnimationFrame(tick);});}})();
```

## 9. 標準セクション順（増減可）

1. スティッキーヘッダー（＋タブナビ）
2. ヒーロー（最重要）
3. 社会的証明（利用企業・実績）
4. サービス紹介（画像コラージュ可）
5. 特長（3〜4カード）
6. 信頼/安心の帯（濃色でリズム）※動画バンドにしてもよい
7. 料金
8. 利用フロー
9. 導入事例
10. FAQ
11. 最終CTA
12. フッター＋モバイル固定CTA

## 10. 仕上げチェック

- 画像は同梱・`alt`＋`width/height`。動画は poster＋背景グラデのフォールバック。
- ヒーローの重なりは px でなく幅% 指定（モバイルで破綻しない）。
- 横スクロールが出ていないか（フルブリード要素の親に `overflow:hidden`）。
- **必ずレンダリング検証**（`image-pipeline.md` の手順でデスクトップ＋モバイル）。
- 数値・社名・事例がダミーなら、フッター等にサンプルである旨を明記。
