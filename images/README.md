# 料理写真について

`dishes/<id>.jpg` は、`recipes.js` の各レシピ(`id`)に対応する料理写真。フリー素材サイト(Unsplash / Pexels / Pixabay)からダウンロードし、リポジトリにローカル保存している(外部サービスへのライブ依存なし)。

## 保存規約

- ファイル名: `<レシピid>.jpg`(idは`recipes.js`のスラッグと完全一致)
- フォーマット: JPEG固定
- サイズ: 長辺600px程度に縮小(表示上の最大サイズは104px。将来の再利用も見込んだ余裕)
- 未取得のレシピ・自作レシピ(`custom_*`)には写真ファイルを置かない(`recipes.js`の`PHOTO_IDS`にも載せない)。`index.html`側は`hasPhoto`が偽の場合そもそも`<img>`タグを出さず絵文字タイルのみ表示するため、404は発生しない。

## ライセンス早見表

| サイト | ライセンス名 | 注意点 |
|---|---|---|
| Unsplash | Unsplash License | 無加工転売NG・写真家の推奨/提携を装うNG・競合ストックサービスの構築NG。クレジット表記は法的には不要 |
| Pexels | Pexels License | 無加工転売NG・被写体の推奨を装うNG。クレジット表記は法的には不要 |
| Pixabay | Pixabay Content License | 「Editorial use only」「Pixabay+」の有料画像は対象外(選定時に除外済み)。クレジット表記は法的には不要 |

いずれも商用・非商用問わず無償利用可。`sources.json`には出典URL・撮影者名(判明分)を記録している(法的義務ではなく、後から見返せるようにするための記録)。

## `sources.json` のスキーマ

`recipes.js`の全116 idがキーとして存在する。

```json
{
  "<id>": {
    "status": "ok" | "missing",
    "site": "Pexels" | "Unsplash" | "Pixabay",     // status:"ok"のみ
    "photographer": "撮影者名",                      // status:"ok"のみ。Pixabayは特定できず"Pixabay contributor"のものあり
    "sourceUrl": "写真ページのURL",                  // status:"ok"のみ
    "imageUrl": "ダウンロード元の直接画像URL",         // status:"ok"のみ
    "license": "ライセンス名",                        // status:"ok"のみ
    "downloadedAt": "YYYY-MM-DD",                    // status:"ok"のみ
    "note": "却下理由、または写っているものの一文説明"
  }
}
```

`status:"ok"`は「合っているか自信を持てる写真が見つかった」場合のみ。近い写真での妥協はしていない(誤った写真より絵文字表示の方が良い、という方針)。

## 未取得(missing)一覧と理由

2026年8月時点で116品中79品を取得、37品は未取得(絵文字タイル表示のまま)。主な傾向:

- **和食の地味な副菜・乾物**(ひじきの煮物・きんぴらごぼう・切り干し大根・揚げ出し豆腐・ちくわの磯辺揚げ・すまし汁・けんちん汁・なめこの味噌汁・かきたま汁 等): 欧米中心のストック写真サイトには在庫が乏しく、該当する写真がほぼ見つからなかった。
- **見分けが難しい料理**(油淋鶏・青椒肉絲・回鍋肉・とろたま肉野菜炒め・豚キムチ春雨・そぼろ丼・アジフライ 等): 似た中華・韓国料理の写真はあるが、細部(タレの種類・具材)が一致せず却下。
- **専門料理**(酸辣湯・タコライス・冷やし中華・焼きおにぎり・卵かけご飯・ハムエッグ 等): 複数候補を確認したが、いずれも別料理か基準(横長・完成品・透かしなし)を満たさなかった。

個別の却下理由は`sources.json`の該当idの`note`を参照。

## 再取得・差し替え手順

1. `recipes.js`の`IMG`旧マップは削除済みだが、`sources.json`の`missing`エントリの`note`に検索の手がかりが残っている。
2. 新しい候補が見つかったら、`https://images.pexels.com/photos/<id>/pexels-photo-<id>.jpeg?auto=compress&cs=tinysrgb&w=600`(Pexels)/`https://images.unsplash.com/photo-<id>?fm=jpg&q=80&w=600&auto=format&fit=crop`(Unsplash)/`https://cdn.pixabay.com/photo/.../..._640.jpg`(Pixabay)の形式でダウンロードし、`dishes/<レシピid>.jpg`として保存。
3. `sources.json`の該当エントリを`status:"ok"`に更新(site/photographer/sourceUrl/imageUrl/license/downloadedAt/noteを記入)。
4. `recipes.js`の`PHOTO_IDS`配列にidを追加。
5. 下記の検証コマンドを実行。

## 検証コマンド

構造・ファイル整合性のチェックスクリプトはリポジトリにコミットせず、必要な都度 scratchpad に作成して実行する(`RECIPES`全idと`sources.json`のキー集合の突き合わせ、`status:"ok"`と`PHOTO_IDS`の一致、実ファイルの存在、JPEGマジックバイト、サイズ範囲の確認)。
