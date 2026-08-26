# ヒーロー構図パターン集

LPの第一印象を決めるヒーローの型と、実装スニペット。参考画像に寄せる指示があるときは、
まず該当パターンを選び、数値を合わせ込む。

## パターンA：切り抜き人物 × 背景色（エネルギッシュな訴求型）

炭酸・採用・サービス系などで強い。鮮やかな単色/グラデ背景に、背景を除去した人物を
直接置き、極太の数値と巨大な「No.1」やバッジを重ねる。参考例：カイテク等の縦型ヒーロー。

構図（縦1列・中央寄せ）：
```
[小さなタグ（例：介護・看護のスポットワーク）]
[特大数字 120] [右に説明2行 ＋ 日付注記]      ← 数字と文字を横並び
[切り抜き人物（画面幅いっぱい／フルブリード）]
   ↑に重ねて [白バッジ×2] [巨大 No.1 👑] [注記]
[CTAブロック：吹き出しラベル ＋ グラデボタン]（縦積み）
```

### 重なりの作り方（%オーバーラップが肝）

人物写真は縦横比が固定なので、負マージンを **px でなく幅% ** にすると、画面幅が変わっても
重なり比率が保たれモバイルで破綻しない。

```css
.hero-stage { position: relative; max-width: 760px; margin: 0 auto; display: flex; flex-direction: column; align-items: center; }
.hero-stage h1 { position: relative; z-index: 3; }               /* 見出しは前面 */

/* 人物を画面幅いっぱいに（フルブリード）＋見出しの下に少し潜り込ませる */
.hero-photo-wrap { position: relative; z-index: 1; width: 100vw; margin-left: 50%; transform: translateX(-50%); max-width: 880px; margin-top: -3%; }
.hero-photo-wrap img {
  width: 100%; height: auto; display: block;
  filter: drop-shadow(0 12px 18px rgba(150,70,0,.22));
  /* 下端を背景色へフェードさせ、切り抜きの硬い縁を消す */
  -webkit-mask-image: linear-gradient(to bottom, #000 70%, rgba(0,0,0,.4) 88%, transparent 100%);
          mask-image: linear-gradient(to bottom, #000 70%, rgba(0,0,0,.4) 88%, transparent 100%);
}
/* No.1 は人物の下半身に重ねる（％なので比率が保たれる） */
.no1 { position: relative; z-index: 3; margin-top: -22%; display: flex; align-items: flex-end; justify-content: center; gap: 12px; }
```

ポイント：
- 切り抜き画像の下端は必ず mask-image でフェードさせると「ステッカー感」が消える。
- 見出しの巨大数字は数値フォント（Outfit等）800、説明テキストはJP gothic 900 を横並び。
- `word-break: keep-all;` を見出しに付け、日本語が変な位置で折れるのを防ぐ（幅が足りなければ改行位置を `<br>` で明示）。

## パターンB：極太の大型数値（ファクト訴求）

「120万人」「No.1」「0円」など、数字を主役にする。数値は等幅感のある幾何サンセリフ
（Outfit / Archivo など）で 800、単位・説明は小さめに添える。IntersectionObserver で
カウントアップさせると目を引く（ただし1画面1箇所まで）。

```js
// data-count を持つ要素をビューポート進入時に 0→目標へ加算
var io = new IntersectionObserver(function(es){
  es.forEach(function(en){ if(en.isIntersecting){ run(en.target); io.unobserve(en.target); } });
}, { threshold: 0.5 });
function run(el){
  var target = parseFloat(el.getAttribute('data-count'));
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches || !target){ el.textContent = target; return; }
  var start=null, dur=1500;
  requestAnimationFrame(function tick(t){
    if(!start) start=t; var p=Math.min((t-start)/dur,1); var e=1-Math.pow(1-p,3);
    el.textContent = Math.round(target*e); if(p<1) requestAnimationFrame(tick);
  });
}
```

## パターンD：背景動画ヒーロー／動画の差し込み（没入・高品質感）

全画面の背景動画は「デザイン性が高い」と感じられやすい定番の手法（例：soai.jp のような
サイト）。空・海・街・人の動きが入るだけで情報量と没入感が跳ね上がる。ヒーローだけでなく、
ページ中間に**動画バンド**を差し込むと緩急が生まれる。

実装の要点：
- `<video autoplay muted loop playsinline>` が必須（`muted` と `playsinline` が無いとモバイルで自動再生されない）。
- **必ず `poster` 画像と背景グラデを敷く**：動画が重い/ブロックされても、ポスター→グラデの順でフォールバックし、テキストが常に読める。
- 動画の上に**ベール（半透明の暗色グラデ）**を重ねて文字の可読性を確保する。
- 動画は `object-fit:cover` で全面に。テキスト・ナビは通常DOMで上に重ねる（z-index管理）。
- 素材は軽量なものを（目安 5〜8MB / 720p 程度、無音）。同梱するとリポジトリが重くなる点に注意。
- フリー動画の入手：**Mixkit**（`https://assets.mixkit.co/videos/<id>/<id>-720.mp4` を UA＋Referer付きで取得、帰属不要）や **Pexels動画**（`https://www.pexels.com/download/video/<id>/`）。ホットリンク保護で403になることがあるので、取得して同梱する。
- `prefers-reduced-motion: reduce` では動画を止める配慮を（`autoplay` を JS で外す、またはポスター静止に）。

```html
<section class="hero">
  <video autoplay muted loop playsinline preload="auto" poster="assets/img/hero-poster.jpg">
    <source src="assets/img/hero.mp4" type="video/mp4" />
  </video>
  <div class="veil"></div>
  <div class="wrap"><!-- 見出し・CTA（z-index:2） --></div>
</section>
```
```css
.hero { position:relative; overflow:hidden; min-height:92vh; display:flex; align-items:center; color:#fff;
        background:linear-gradient(160deg,#7FC8F1,#2A7FC0); }   /* 動画不可時のフォールバック */
.hero video { position:absolute; inset:0; width:100%; height:100%; object-fit:cover; z-index:0; }
.hero .veil { position:absolute; inset:0; z-index:1; background:linear-gradient(180deg,rgba(20,50,80,.3),rgba(20,50,80,.5)); }
.hero .wrap { position:relative; z-index:2; }
```

**中間の動画バンド**（ページの途中に差し込む）：同じ `<video>+veil` 構造を `padding-block` 大きめの
セクションに入れ、中央にひとことメッセージを重ねるだけ。写真セクションの連続に“動く休符”を作れる。

検証時の注意：ヘッドレスでは `networkidle` が動画ストリームで発火しないことがある。`waitUntil:'load'`＋
明示的 `waitForTimeout` を使う。自動再生されずポスターが写ることがあるが、それはフォールバックが
効いている証拠なので問題ない。

## パターンC：写真カード横並び（落ち着いた信頼型）

医療・士業・BtoBの堅めなサービス向け。左にコピー＋CTA、右に角丸の写真カード（＋実績の
小カードを重ねる）。エネルギッシュさより信頼感。背景は温かみのある紙白などの淡色。

## 共通：CTAボタン（グラデ＋丸アロー＋吹き出しラベル）

参考の飲料/採用系LPに多い、青→紫グラデの主CTA。ラベルを吹き出しで上に添えると誘導が強まる。

```css
.btn { display:inline-flex; align-items:center; gap:12px; font-weight:700; padding:16px 26px 16px 32px; border-radius:999px; }
.btn--primary { background:linear-gradient(95deg,#2C6BEC,#8A36D9); color:#fff; box-shadow:0 10px 26px rgba(95,60,210,.4); }
.btn .arrow { width:28px; height:28px; border-radius:50%; display:grid; place-items:center; background:rgba(255,255,255,.28); }
.btn--white { background:#fff; color:#7A3BE0; } .btn--white .arrow { background:#7A3BE0; color:#fff; }
/* 吹き出しラベル */
.cta-label { position:relative; background:rgba(255,255,255,.95); color:#c0400a; font-weight:700; font-size:.85rem; padding:5px 16px; border-radius:999px; }
.cta-label::after { content:""; position:absolute; left:50%; transform:translateX(-50%); bottom:-5px; border:6px solid transparent; border-top-color:rgba(255,255,255,.95); }
```

## 共通：No.1 / バッジ

```html
<div class="no1">
  <div class="no1-badges">
    <span class="nb">介護・看護<b>有資格者 利用率</b></span>
    <span class="nb">専門アプリ<b>掲載求人数</b></span>
  </div>
  <div class="no1-main"><svg class="crown" ...>👑パス</svg>No.<span>1</span></div>
  <div class="no1-note">※自社調べ</div>
</div>
```
背景が色地なら、バッジは白の実体ピル（`background:#fff; color:<deep accent>`）にすると視認性が高い。
半透明白＋白枠でも可。No.1 は白＋text-shadow で巨大に。
