# 関東有名ケーキ店・カフェマップ

関東のケーキ店・カフェ候補を地図で見るための静的HTMLツールです。

このREADMEは、今後このツールを人間または別のAIが編集するときの保守メモを兼ねています。

## リポジトリ運用

- Dev: `/Users/yoichi/Documents/GitHub/HTMLTools/kanto-cake-cafe-map/`
- Production: `/Users/yoichi/Documents/GitHub/productions/kanto-cake-cafe-map/`
- 公開想定URL: `https://monge13.github.io/kanto-cake-cafe-map/`

通常はDev側の `index.html` を編集し、検証後にProduction側へコピーします。

```bash
cp /Users/yoichi/Documents/GitHub/HTMLTools/kanto-cake-cafe-map/index.html /Users/yoichi/Documents/GitHub/productions/kanto-cake-cafe-map/index.html
```

Production側のpushはユーザーが行う運用です。AIがpushできない環境でも、Production側にコミットまで作っておけばユーザーが `git push` できます。

```bash
cd /Users/yoichi/Documents/GitHub/productions/kanto-cake-cafe-map
git push
```

## 設計思想

- 旅行や散歩の途中で「近くに良いケーキ店・カフェがあるか」を見るための地図です。
- ただのランキング転載ではなく、駅からの行きやすさ、散歩との相性、手土産向き、イートイン体験を重視します。
- ケーキ専門店だけではなく、ショコラ、ジェラート、焼き菓子、老舗喫茶、ホテルペストリー、甘味として重要なカフェも拾います。
- 「有名だがケーキランキングに出にくい店」を漏らさないことを重視します。例: `teal`、`Maison romi-unie`、`UN GRAIN`、`L'atelier MOTOZO`。
- 閉店・移転・営業時間変更が起きやすいので、訪問前確認リンクを出す方針です。
- UIはスマホで使いやすいことを優先します。地図が勝手に戻る操作は避けます。

## 現在の構成

すべて `index.html` に入っています。ビルド工程はありません。

- HTML/CSS/JSの単一ファイル
- 地図: Leaflet + OpenStreetMap
- 店舗データ: JavaScript配列
- 訪問済みフラグ: `localStorage`
- 追加データ用の古いlocalStorageキー: `kantoCakeMap.extraShops.v1`
- 訪問済みキー: `kantoCakeMap.visited.v1`

主なデータ配列:

- `baseShops`: 初期からあるケーキ店中心のデータ
- `cafeShops`: カフェ中心のデータ
- `expandedShops`: 後から増やした店舗データ
- `rankingSources`: 更新チェック用の外部検索リンク

## 店舗データの形

基本形は以下です。

```js
{
  name: "店名",
  category: "cake",
  area: "tokyo-south",
  areaLabel: "東京 南側・城南",
  lat: 35.00000,
  lng: 139.00000,
  station: "最寄駅",
  walk: "徒歩5分前後",
  type: ["top", "gift", "salon"],
  signature: "代表菓子、特徴",
  note: "なぜ載せるか。散歩/手土産/名店性など。",
  url: "https://..."
}
```

`category`:

- `cake`: ケーキ、パティスリー、ショコラ、焼き菓子、甘味全般
- `cafe`: カフェ、喫茶、コーヒー主体の店

`area`:

- `tokyo-east`: 東京 東側・都心東
- `tokyo-west`: 東京 西側・多摩
- `tokyo-south`: 東京 南側・城南
- `tokyo-north`: 東京 北側
- `kanagawa`: 神奈川
- `saitama`: 埼玉
- `chiba`: 千葉
- `north-kanto`: 北関東

`type`:

- `top`: 名店・遠征級
- `walk`: 散歩向き
- `gift`: 手土産
- `salon`: イートイン/サロン

## 店舗追加の方針

追加時は以下を確認してください。

- 公式サイト、公式SNS、食べログ、Google検索などで営業中らしいことを確認する
- 住所・最寄駅・徒歩目安をできるだけ確認する
- 緯度経度はGoogle MapやOpenStreetMap相当で概算でもよいが、明らかに違う場所に置かない
- `note` には「なぜこの地図に載せるのか」を短く書く
- 食べログURLしか見つからない場合は一時的に食べログURLでも可。ただし公式があれば公式を優先する
- 店名は重複キー扱いになるので、同一店の表記ゆれに注意する

避けたい追加:

- ただ近所にあるだけの普通のチェーン店
- 閉店済み、移転未確認の店
- 評価根拠が薄い店
- アウトレットや駅ナカ実用店だけで名店感がないもの。ただし地域の甘味地図として意味がある場合は `note` に理由を書く

## 更新チェックUI

画面下部の「データ更新」内にあるボタンは、現在は `更新チェックを開始` の1つだけです。

このボタンの役割:

- `rankingSources` の候補探索リンクを一覧化する
- 既存掲載店ごとの「閉店/営業確認」Google検索リンクを一覧化する
- 一部の候補探索リンクを新規タブで開く

重要:

- 静的HTMLなので、食べログ等から自動スクレイピングして店舗データを書き換える設計ではありません。
- 外部サイトの規約やCORS制限を避けるため、ユーザー/編集者が確認するためのリンク生成に留めています。
- 以前はJSON/CSV貼り付けUIがありましたが、ユーザー要望により表には出していません。

## 「行った」フラグ

`行った` / `未訪問` ボタンは `localStorage` に即時保存します。

地図の再描画は行いません。理由:

- 地図位置やズームが戻ると使いにくい
- ユーザーは訪問済みフラグを連続で押したい
- `行ったのみ` / `未訪問のみ` フィルタを切り替えたタイミングで反映されれば十分

この挙動は `toggleVisited()` と `updateVisitedControls()` で実現しています。ここで安易に `render()` を呼ばないでください。

## アフィリエイト/予約リンクを追加する場合

将来的に以下のリンクを店舗ごとに持たせる可能性があります。

- 公式
- Google Map
- 公式オンラインショップ
- 食べログ店舗ページ
- 食べログ予約
- 一休レストラン予約
- 楽天/Yahoo/Amazon/百貨店EC等のオンライン購入

推奨データ形:

```js
links: {
  official: "https://...",
  map: "https://...",
  onlineShop: "https://...",
  tabelog: "https://...",
  tabelogReserve: "https://...",
  ikyu: "https://...",
  affiliateOnline: "https://..."
}
```

現状は `url` だけです。後方互換のため、`links` を追加しても `url` は残すのが安全です。

リンク表示方針:

- 予約できる店だけ `予約` ボタンを出す
- 通販できる店だけ `通販` ボタンを出す
- アフィリエイトリンクには `rel="sponsored nofollow"` を付ける
- ページ内に「一部リンクはアフィリエイト広告を含みます」と明記する
- 食べログや一休の規約、ASPの提携条件を確認してからリンク化する

特に相性が良い収益化:

- 一休: ホテルラウンジ、アフタヌーンティー、レストラン予約
- 食べログ予約: ネット予約可能なカフェ/レストランだけ
- 公式通販/百貨店EC: 焼き菓子、チョコレート、ギフト向き商品

## 検証手順

編集後は最低限これを実行します。

```bash
node - <<'NODE'
const fs = require('fs');
const file = '/Users/yoichi/Documents/GitHub/HTMLTools/kanto-cake-cafe-map/index.html';
const html = fs.readFileSync(file, 'utf8');
const script = html.match(/<script>([\s\S]*)<\/script>/)[1];
new Function(script);
for (const area of ['tokyo-east','tokyo-west','tokyo-south','tokyo-north','kanagawa','saitama','chiba','north-kanto']) {
  const count = (html.match(new RegExp(`area: "${area}"`, 'g')) || []).length;
  console.log(area, count);
}
console.log('all', (html.match(/name: "/g) || []).length);
NODE
```

差分チェック:

```bash
cd /Users/yoichi/Documents/GitHub/HTMLTools
git diff --check -- kanto-cake-cafe-map/index.html
```

Productionへ同期後:

```bash
cd /Users/yoichi/Documents/GitHub/productions/kanto-cake-cafe-map
git diff --check -- index.html
```

## 現在の掲載件数の目安

2026-06-29時点:

- 全体: 165件
- 東京 東側・都心東: 45件
- 東京 西側・多摩: 34件
- 東京 南側・城南: 37件
- 東京 北側: 27件
- 神奈川: 10件
- 埼玉: 6件
- 千葉: 5件
- 北関東: 1件

今後は神奈川、埼玉、千葉、北関東が薄いので、そこを増やす余地があります。
