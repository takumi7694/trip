# 旅程プランナー（単一 index.html）

スマホ最優先の旅行計画アプリです。サーバー不要（静的ファイルのみ）で、GitHub Pages に置いて家族と URL 共有して使えます。デザインは暖かみのあるペーパートーン＋自動ダークモード。予定はティント（薄い色面）＋カラーレールで表示します。

- `index.html` … アプリ本体（HTML / CSS / JS 全部入り）
- `sw.js` … オフライン用サービスワーカー（任意。無くても動きます）

初期データは **NY（12/16–19）→ グアダラハラ（12/20–22）→ メキシコシティ（12/23–24）の全9日**が入っています。12/16 は JFK 10:30 着、12/24 は MEX 21:45 発の前提で組んであります。住所・営業時間は Web で調査した内容（2026年8月時点）を反映済みです。

---

## 1. できること

| 機能 | 操作 |
|---|---|
| 日付切り替え | 上部の日付タブをタップ／画面を左右スワイプ |
| 予定を追加 | 空き時間帯をタップ、または右下の「＋」 |
| 予定の詳細を見る | 予定ブロックをタップ（住所・地図・おすすめ度★・営業時間・メモ・URL） |
| 時間を移動 | 予定ブロックを**長押し（約0.4秒）→ そのままドラッグ**（30分単位でスナップ） |
| 所要時間を変更 | ブロック**下端のグリップを上下にドラッグ** |
| 編集・削除 | 詳細画面の「編集」「削除」 |
| 削除の取り消し | 削除直後のトーストの「元に戻す」／メニュー → ゴミ箱 → 復元 |
| リンクを開く | ブロック右上の「↗」／詳細画面の URL |
| **カレンダーへ1日まとめて追加** | 上部の「📅 この日をカレンダーへ」／メニューの「全日程をカレンダーへ」 |
| LINE 共有用テキスト | 上部の「📋 コピー」／メニュー → 「全日程をコピー」 |
| カテゴリ色分け | 上部の凡例チップ。タップでその色だけ非表示（絞り込み）。メニューで色・名前の変更／追加／削除 |
| リスト表示 | 凡例の右にある「タイムライン / リスト」切替 |
| 都市ごとのタイムゾーン | メニュー →「都市・タイムゾーン」。日付タブに略称（NY / GDL / MEX）が出ます |
| 検索 | ヘッダーの虫めがね。全日程から店名・住所・メモを横断検索し、タップでその日へジャンプ |
| ライト / ダーク | ヘッダーの月・太陽アイコンで切替。メニューの「表示」で 自動 / ライト / ダーク を選択（自動＝端末設定に追従） |

タイムラインは 6:00〜翌3:00 を30分グリッドで表示し、**その日の予定に合わせて自動で上下に広がります**（12/20 の 5:10 着や 12/19 の翌1:20 発もはみ出さずに表示。24時以降は `翌1:20` と表記）。

### カレンダー追加の仕組み

Google カレンダーは「複数の予定を1つのリンクでまとめて追加する」URL を用意していないため、**1日分（または全日程）を .ics ファイルにまとめて出す方式**にしています。

- **iPhone / Android**：ボタンを押すと共有シートが開きます。「カレンダー」を選べば、その日の予定が全部まとめて追加できます
- **PC**：.ics ファイルが保存されます。Googleカレンダー → 設定 →「インポート」で取り込みます
- 時刻は**都市ごとのタイムゾーン**で変換されます（NY = America/New_York、GDL・CDMX = America/Mexico_City）。時差1時間のNYとメキシコが混在していても、日本のカレンダーで見て正しい時刻に入ります

---

## 2. GitHub Pages で公開する

1. GitHub で新しいリポジトリを作る（例：`trip`）。**Public** にします。
2. `index.html`（と `sw.js`）をアップロード
   - 画面の **Add file → Upload files** にドラッグ＆ドロップ → **Commit changes**
3. リポジトリの **Settings → Pages** を開く
4. **Build and deployment → Source** を **Deploy from a branch**、**Branch** を `main` / `(root)` にして **Save**
5. 1〜2分待つと `https://<ユーザー名>.github.io/trip/` で公開されます
6. その URL を家族に共有。スマホで開き、**共有ボタン → ホーム画面に追加**すると全画面アプリとして起動します

> 更新したいときは `index.html` を差し替えて Commit するだけです（反映に1分ほどかかります）。

---

## 3. Firebase でリアルタイム同期する（家族で共同編集）

未設定のままでも「その端末だけの保存（localStorage）」で普通に使えます。家族の端末と同期したいときだけ設定してください。

### 3-1. プロジェクトと Realtime Database を作る

1. <https://console.firebase.google.com/> → **プロジェクトを追加**（名前は何でもOK。Googleアナリティクスは不要）
2. 左メニュー **構築 → Realtime Database** → **データベースを作成**
3. ロケーションは `asia-southeast1`（シンガポール）などを選択
4. セキュリティルールは **テストモードで開始** を選択（あとで 3-3 で調整）

### 3-2. 設定値をアプリに貼る

1. 左上の歯車 → **プロジェクトの設定** → 下部の **マイアプリ** → **</>（ウェブ）** を選んでアプリを登録
2. 表示される `firebaseConfig` の中身を `index.html` の冒頭にコピーします

```js
const FIREBASE_CONFIG = {
  apiKey:            "AIzaSy...",
  authDomain:        "my-trip.firebaseapp.com",
  databaseURL:       "https://my-trip-default-rtdb.asia-southeast1.firebasedatabase.app", // ← 必須
  projectId:         "my-trip",
  storageBucket:     "my-trip.appspot.com",
  messagingSenderId: "1234567890",
  appId:             "1:1234567890:web:abcdef"
};
```

- **`databaseURL` が入っていないと同期は始まりません**（Realtime Database の画面上部に表示されている URL です）
- 保存して GitHub に上げ直すと、右上の表示が「ローカル保存」→「同期中」に変わります

### 3-3. セキュリティルール（重要）

テストモードのままだと30日で読み書きできなくなり、また URL を知っている人なら誰でも書き込めます。**Realtime Database → ルール** を次のようにしておくのがおすすめです。

```json
{
  "rules": {
    "trips": {
      "$room": {
        ".read":  "$room === 'kazoku-8f3k2p'",
        ".write": "$room === 'kazoku-8f3k2p'"
      }
    }
  }
}
```

`kazoku-8f3k2p` の部分は**推測されにくい合言葉**に変えてください。そのうえでアプリ側の部屋を合わせます。

- `index.html` の `ROOM_ID` の初期値（`"family"`）を同じ文字列にする、または
- URL の末尾に `#room=kazoku-8f3k2p` を付けて共有する（アプリのメニュー → 共有 →「共有URLをコピー」で作れます）

同じ部屋 ID を開いている端末どうしが、リアルタイムで同期します。

---

## 4. 12/16 の組み直し（JFK 10:30 着）

| 時刻 | 内容 |
|---|---|
| 10:30–12:00 | JFK 到着 → 入国審査・荷物受取（国際線は着陸〜到着ロビーまで **60〜90分**が目安。混雑期は2時間超） |
| 12:00–13:15 | JFK → Williamsburg（配車 30〜60分・$60〜85。節約なら AirTrain $8.75 → A線 → Broadway Junction → L線 Bedford Av で計 $11.75・約80分） |
| 13:15–13:45 | Pod Brooklyn に荷物預け（チェックインは15:00から。当日中の荷物預かりはフロントで可） |
| 14:00–14:45 | Apollo Bagels（**水曜は15:00閉店**。入国が押したら12/19朝に振替） |
| 14:45–16:00 | Williamsburg 散策 |
| 16:00–17:00 | Devoción（18:00まで） |
| 17:00–17:40 | Pod Brooklyn チェックイン |
| 17:40–18:20 | Barclays Center へ移動 |
| 18:20–19:20 | Nets Team Store・アリーナ視察（**開場18:00**＝試合開始90分前） |
| 19:30–22:00 | Mavericks @ Nets |
| 22:30–23:30 | 軽く1杯 → 早めに戻る |

12/19 の帰路も 01:20 発から逆算し直しました（**21:00–21:15 に Williamsburg 発**で 22:20 ターミナル着＝出発3時間前。EWR なら 21:15–21:30 発）。空いた 18:45–20:45 に「最後の夕食 / 一杯」を入れています。

---

## 5. 調査した住所・営業時間（2026年8月時点）

すべてアプリ内の各予定にも入っています。**12月の年末年始・祝日変更は未公表**なので、旅行1週間前に再確認してください。

| 店・施設 | 住所 | 営業時間 |
|---|---|---|
| Apollo Bagels (Williamsburg) | 133 N 7th St, Brooklyn, NY 11249 | 月-木 7:00-15:00 / 金-日 7:00-17:00（売切次第終了） |
| Devoción (Williamsburg) | 148 Grand St, Brooklyn, NY 11249 | 月-金 7:00-18:00 / 土日 7:30-18:00 |
| Edith's Sandwich Counter | 495 Lorimer St, Brooklyn, NY 11211 | 月-木 8:00-15:00 / 金-日 8:00-17:00 |
| Radio Bakery (Greenpoint) | 135 India St, Brooklyn, NY 11222 | 毎日 7:30-15:30（売切次第終了） |
| Sunday In Brooklyn | 348 Wythe Ave, Brooklyn, NY 11249 | 朝食 8:00-9:30 / ブランチ 9:30-16:30 / ディナー 17:00-22:00 |
| Pod Brooklyn | 247 Metropolitan Ave, Brooklyn, NY 11211 | IN 15:00 / OUT 12:00・当日中の荷物預かり可 |
| Ace Hotel Brooklyn | 252 Schermerhorn St, Brooklyn, NY 11217 | ロビーバー 日-木 12:00-24:00 / 金土 12:00-02:00 |
| The Ned NoMad | 1170 Broadway, New York, NY 10001 | Cecconi's 水-土 7:00-23:00 / 日-火 7:00-22:00 |
| PUBLIC Hotel | 215 Chrystie St, New York, NY 10002 | The Roof 日-水 17:00-02:00 / 木-土 17:00-04:00 |
| The Hoxton Williamsburg | 97 Wythe Ave, Brooklyn, NY 11249 | K'Far 日-水 8:00-23:30 / 木-土 8:00-00:30 |
| Wythe Hotel | 80 Wythe Ave, Brooklyn, NY 11249 | Le Crocodile 毎日7:00〜 / Bar Blondeau 月-金 17:00-24:00 |
| Barclays Center | 620 Atlantic Ave, Brooklyn, NY 11217 | 19:30開始の試合は**18:00開場**（90分前） |
| Nets Team Store | 185 Flatbush Ave（アリーナ内） | 毎日 12:00-17:00＋全試合中 |
| NBA Store 5th Ave | 545 5th Ave, New York, NY 10017 | 月-木 10:00-20:00 / 金土 10:00-21:00 / 日 10:00-20:00 |
| Hoops Klub | Greenpoint 237 Russell St ／ 本店 28 Cumberland St | 要予約。日中枠 10:00-14:00 ほか。1日パス$30／シュートアラウンド$40/時 |
| Brooklyn Bridge Park Pier 2 | 334 Furman St, Brooklyn, NY 11201 | 10-4月 8:00-21:00（屋根付きで冬も可・無料） |
| Public Records | 233 Butler St, Brooklyn, NY 11217 | 月火休 / 金 18:00-翌4:00 / 土 10:00-翌4:00 |
| Elsewhere | 599 Johnson Ave, Brooklyn, NY 11237 | 金土 翌4:00まで（イベント日のみ） |
| BASEMENT | 52-19 Flushing Ave, Maspeth, NY 11378 | 金土のみ・22:30前後開場 |

### 調査でわかった変更点（元のエクセルから修正）

- **Edith's**：312 Leonard St の Eatery & Grocery は閉店。現在は **495 Lorimer St の Sandwich Counter**
- **The Hoxton**：ロビーの飲食は Klein's ではなく **K'Far** に入れ替わり
- **The Ned NoMad**：非会員が入れるのは1階 Cecconi's と Little Ned のみ。**Ned's Club・最上階は会員限定**（宿泊者でも不可）
- **BASEMENT**：Paragon の地下ではなく **Knockdown Center（Maspeth, Queens）の地下**
- **NBA 日程**：2026-27 の公式日程で 12/16 Mavericks・12/18 Magic とも **19:30 開始**を確認済み

### 主な出典

- 各店の公式サイト（apollobagels.com / devocion.com / edithsbk.com / radiobakery.nyc / sundayinbrooklyn.com / acehotel.com / thened.com / publichotels.com / thehoxton.com / wythehotel.com / thepodhotel.com / publicrecords.nyc / elsewhere.club / basementny.net / hoopsklub.com）
- Barclays Center 公式 A-Z ガイド／イベントページ、Brooklyn Bridge Park 公式
- MTA 公式（AirTrain $8.75・地下鉄 $3.00・subway+AirTrain $11.75）、NYC TLC 運賃ページ、Uber ルートページ
- CBP（Global Entry の日本対応、MPC は米加国民のみ）

---

## 6. グアダラハラ / メキシコシティ（12/20–12/24）

### フライト前提

| 区間 | 日付 | 時刻 |
|---|---|---|
| HND → JFK | 12/16 | 11:35 → 10:30（同日着） |
| JFK → MEX → GDL | 12/20 | 1:20発 → MEX 5:10着 → 8:25発 → GDL 10:00着 |
| GDL → MEX | 12/23 | 11:05 → 12:25 |
| MEX → NRT | 12/24 | 21:45 → 翌6:30 |

12/20 の MEX 乗継は3時間15分。**国際線→国内線なのでメキシコ入国審査＋預け荷物の受取／再預けが必要**で、余裕は多くありません。

### 曜日で決まった組み替え

2026年の曜日は 12/20=日、12/21=月、12/22=火、12/23=水、12/24=木。調査の結果、当初案のままでは入れない店があったので入れ替えました。

- **月曜（12/21）は Museo Cabañas・MUSA・Insurgente・Bar Américas・Mecenas・Pare de Sufrir が全部休み** → この日をテキーラ日にしたのは正解。夜は月曜も開いている **Oliveria**（Mecenas は月曜休み）
- **日曜（12/20）は El Terrible Juan と Casa ITESO が休み** → どちらも別日に配置済み。Insurgente は日曜21:00閉店なので 18:00–19:30 に
- **火曜（12/22）は Pare de Sufrir が休み** → 夜は火曜営業の **Mecenas**（19:00–、OpenTableで19:00–20:30の予約枠あり）
- **Bar Américas は水〜日営業**なので 12/20(日) は行ける。ラインナップは公式カレンダー / Resident Advisor で直前確認
- **CRAFT Americana は Col. Americana ではなく Chapalita（Av. Tepeyac 497）** → 近い **La Americana Taproom**（Av. Vallarta 1020 2階）に差し替え
- **Jose Cuervo Express（テキーラ列車）は原則土曜のみ運行**。今回の日程には土曜が無いため乗れません。Tequila へはバス（Tequila Plus・片道130 MXN・30分毎）、車で約1時間、または送迎付きツアー（TQM Tours 179USD／Koyote Tours 2,400 MXN）

### 12/24（クリスマスイブ）の注意

- **Museo Tamayo は INBAL 系美術館の例年運用で 12/24 は 10:00–14:00 の短縮営業**（12/25 は休館）。2026年分の告知は未公表
- **Museo Jumex も過去年は 12/24 が 10:00–14:00**。旅程は「14:00 までに退館」前提で組んであります
- **LagoAlgo の 12/24 営業は完全に不明**。特別営業・貸切・早仕舞いがありうるので12月初旬に電話（55 5515 9585）か OpenTable で要確認
- メキシコでは 12/24 夜の家族の夕食（Cena de Nochebuena）が最大の行事で、**個人経営の飲食店・ショップは 12/24 午後〜夜と 12/25 終日を休むのが一般的**
- 空港へは **17:15–17:45 に Roma Norte 発**。通常20〜30分ですが、12/24 夕方は混むので60〜75分を確保（国際線3時間前＝18:45 に T1 着）。19:00 を過ぎると逆に街から車が消えて極端に空くのが例年のパターン。配車ドライバーが減るので Uber Reserve 等で事前手配推奨

### 主な住所・営業時間（2026年8月時点）

| 店・施設 | 住所 | 営業時間 |
|---|---|---|
| Café palReal | C. Lope de Vega 113, Arcos Vallarta, 44130 GDL | 月-土 8:00-23:00 / 日 8:00-18:00（別ソースで食い違いあり） |
| Museo Cabañas | Calle Cabañas 8, Plaza Tapatía, 44360 GDL | 火-日 10:00-17:00（月休）・一般160 MXN |
| Insurgente Tap Room | Calle Argentina 16, Col. Americana, 44160 GDL | 火-土 16:30-24:00 / 日 13:00-21:00（月休） |
| Bar Américas | Av. Chapultepec Sur 507, 44160 GDL | 水-日 22:00-翌7:00（月火休） |
| El Terrible Juan Café | C. Colonias 440, Col. Americana, 44160 GDL | 月-土 8:00-22:00（日休） |
| Casa ITESO Clavigero | José Guadalupe Zuno 2083, Col. Americana, GDL | 月-金 9:00-19:00 / 土 10:00-14:00（日休）・無料 |
| MUSA | Av. Juárez 975, Col. Americana, 44160 GDL | 火-土 10:00-18:00 / 日 10:00-15:00（月休）・無料 |
| Oxen Concept Store | Av. Miguel Hidalgo 1323, Col. Americana, 44160 GDL | 月-金 11:00-20:00 / 土 11:00-19:00 / 日 11:00-17:00 |
| Casa Habita | Lerdo de Tejada 2308, Col. Lafayette, 44140 GDL | テラス 月火 11:00-20:00 / 水-日 10:00-23:00 |
| Mecenas | Av. de la Paz 2133, Col. Americana, GDL | 火-木・日 19:00-24:45 / 金土 19:00-01:45（月休） |
| Oliveria Cocktail Bar | Calle Libertad 1852, Col. Americana, 44160 GDL | 月-木 18:00-01:00 / 金土 18:00-02:30 |
| La Fortaleza（蒸留所） | F. J. Sauza Mora 567, Centro, 46400 Tequila | 施設 8:00-17:00・**完全予約制**・約50USD/2時間 |
| El Tequileño（蒸留所） | Chiapas 51, Centro, 46400 Tequila | 公式サイト（Peek）で要予約・時間不明 |
| Panadería Rosetta | Colima 179, Roma Norte, 06700 CDMX | 月-土 7:00-22:00 / 日 7:30-21:30・予約不可/行列 |
| Buna（Café Rico） | Orizaba 42, Roma Norte, 06700 CDMX | 毎日 8:00-21:00 |
| kurimanzutto | Gob. Rafael Rebollar 94, San Miguel Chapultepec, 11850 | 火-木 11:00-18:00 / 金土 11:00-16:00（日月休）・無料 |
| Fundamentally | Colima 184 2階, Roma Norte, 06700 CDMX | 月-土 11:00-19:30 / 日 11:00-18:00 |
| La Roma Brewing | Av. Yucatán 84 local L, Roma Norte, 06700 CDMX | 水 12:00-23:00 / 木 12:00-24:00 / 金土 12:00-翌2:00 |
| Departamento | Av. Álvaro Obregón 154 2階, Roma Norte, 06700 CDMX | 水-土 19:00-翌3:00・**テーブル要予約** |
| LagoAlgo | Lago Mayor, Bosque de Chapultepec II, CDMX | レストラン 毎日 8:30-19:00 / ギャラリー 水-日 10:00-18:00 |
| Museo Tamayo | Paseo de la Reforma 51, 11580 CDMX | 火-日 10:00-18:00・一般95ペソ（日曜無料） |
| Museo Jumex | M. de Cervantes Saavedra 303, Granada, 11520 CDMX | 火-金 10:00-17:00 / 土 10:00-19:00・無料 |

**空港アクセス**：GDL空港→Col. Americana は 20〜35分、認可タクシー260〜340 MXN／Uber 120〜180 MXN（GDLは空港でのUber乗車OK）。MEX空港→Roma Norte は通常20〜30分、認可タクシー370〜440 MXN／Uber・DiDi 250〜320 MXN（2026年3月からAICMが配車アプリのターミナル内乗車を規制中、指定ゾーンまで歩く場合あり）。

**営業時間が確認できなかった店**：Metódico Café、Xaneque GDL、La Americana Taproom（火曜以外）、Roma Vintage の曜日別。いずれも Instagram / Google マップで当日確認してください。

### 主な出典（メキシコ分）

- 各施設の公式サイト（museocabanas.jalisco.gob.mx / musaudg.mx / cultura.iteso.mx / casahabita.com / mecenasbar.com / baramericas.com.mx / cervezainsurgente.com / tequilafortaleza.com / tequileno.com / lago-algo.mx / museotamayo.org / fundacionjumex.org / kurimanzutto.com / elenareygadas.com / buna.mx / wefundamentally.com）
- INBAL の12/24短縮営業告知、Museo Jumex「HORARIOS FIN DE AÑO」告知
- AICM 公式（3時間前推奨）、mundocuervo.com（Jose Cuervo Express の運行日・料金）

---

## 7. 補足

- **PWA**：マニフェストとアイコンはアプリ内で自動生成しています。`sw.js` を一緒に置くと、機内モードでもキャッシュから起動できます（`file://` では動きません。GitHub Pages などの https 環境が必要です）。
- **データの持ち出し**：メニュー → データ → 「JSON書き出し / 読み込み」でバックアップできます。
- **サンプルに戻す**：メニュー最下部の「サンプルに戻す」で初期の全9日程に戻ります。
- **都市を増やす**：メニュー →「都市・タイムゾーン」→「＋ 都市を追加」。開始日・都市名・略称・タイムゾーンを設定すると、その日以降がその都市として扱われます。
