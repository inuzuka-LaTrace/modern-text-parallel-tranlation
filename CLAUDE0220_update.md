# フランス語圏象徴主義文学対訳アプリ開発 - プロジェクト全体文書

## アプリ概要

### 目的
19〜20世紀フランス語圏の批評・詩・散文テキストの原文と日本語仮訳を並べて学習できるWebアプリケーション

### アプリ名（改訂検討中）
旧称「19-20世紀フランス批評理論・文学テキスト対訳アプリ」から実態に即した名称への変更を検討中。候補：
- 「フランス語圏象徴主義文学対訳」
- 「近代フランス語文学対訳」

### ターゲットユーザー
- フランス文学・批評理論を学ぶ学生・研究者
- フランス語学習者（中級〜上級）
- 比較文学・思想史に興味がある読者

### 主な機能一覧
1. **対訳表示機能** — フランス語原文と日本語仮訳を行単位で並列表示。表示/非表示の切り替え（原文・仮訳・自分の訳）
2. **ユーザー訳文機能** — 各行に自分の訳を記入・保存。localStorageによる永続化
3. **テキスト管理機能** — カテゴリー別フィルタリング、テキスト選択UI、メタデータ表示
4. **検索機能** — タイトル・著者・キーワード・本文（french／仮訳）横断検索。カテゴリーフィルターとは独立動作
5. **カスタマイズ機能** — ダークモード/ライトモード、フォントサイズ調整（小・中・大・特大）、フォント切り替え（Garamond, Noto Serif, Sans）。⚙️ボタンのドロップダウンに集約
6. **学習支援機能** — キーワード表示、関連テキストへのナビゲーション、文脈説明、段落数表示
7. **段落アコーディオン** — 折りたたみ時は行番号＋原文冒頭のみ表示。「すべて展開／折りたたむ」一括操作。自分の訳がある行には紫ドット表示
8. **行結合機能（検討中）** — 行単位／スタンザ単位／自由結合の切り替え（JSONは行単位で保持し、UI側でグルーピング）

---

## コンプライアンス方針（2026-02-20 策定）

### 著作権について
- 収録テキストはすべて没後70年以上経過した作家のもの（仏・日いずれの著作権法でもパブリックドメイン）
- 日本語訳はすべてClaude生成の新規訳文であり、既存翻訳書からの流用はない

### 翻訳の表記方針
- フィールド名は **`provisionalTranslation`（仮訳）** を正式とする（旧`officialTranslation`からの移行完了）
- UIは両フィールド名にフォールバック対応済み（`provisionalTranslation ?? officialTranslation`）
- ラベル表記は「公式訳」→「仮訳」に修正済み
- フッター・ウェルカム画面に免責文を設置：「掲載の日本語訳は学習補助のための試訳であり、確定した翻訳ではありません」

### 原典テキストの取り扱い
- **流通量の多い作家**（ボードレール、マラルメ、ヴァルモール夫人、ヴェルレーヌ等）：Claudeの知識ベースから作成可
- **流通量の少ない作家**（ヴァン・レルベルグ等）：Gallica等からPDFを入手し、スクリーンショットをClaudeに送付して原典から直接JSON化する
- 原典未確認のまま生成したファイルは「要削除または原典照合必須」として管理する

### 象徴主義方面の意義
先行する日本語翻訳がほぼ存在しないため既存訳との混同リスクが低く、かつ日本語アクセス手段の空白を埋める学術的意義が高い。

---

## 現在の技術スタック

### Frontend
- **Framework**: React 18
- **Build Tool**: Vite 5
- **CSS Framework**: Tailwind CSS 3
- **Language**: JavaScript (JSX)

### State管理
- React Hooks (useState, useEffect, useRef)
- localStorage（ユーザー訳文の永続化）

### デプロイ
- **Hosting**: Vercel
- **Repository**: GitHub
- **CI/CD**: Vercel自動デプロイ（git push時）

### フォント
- **欧文Serif**: EB Garamond (Google Fonts)
- **和文Serif**: Noto Serif JP (Google Fonts)
- **欧文Sans**: Inter (Google Fonts)
- **和文Sans**: Noto Sans JP (Google Fonts)

---

## データ構造（最新JSONスキーマ）

**1テキスト1ファイル方式**

```json
{
  "id": "vanlerberghe_entrevision",
  "title": "Entrevision",
  "author": "Charles Van Lerberghe",
  "source": "Entrevisions",
  "year": "1898",
  "firstPublication": "L'Art jeune, 1895",
  "difficulty": "中級",
  "category": "vanlerberghe",
  "modernRelevance": "★★★★☆",
  "context": "...",
  "keywords": ["vision", "mystère", "lumière"],
  "relatedTexts": ["vanlerberghe_solitude"],
  "paragraphs": [
    {
      "id": 1,
      "french": "...",
      "provisionalTranslation": "..."
    }
  ]
}
```

### フィールド説明
- `firstPublication`：詩集収録前の初出誌情報（判明している場合のみ）
- `provisionalTranslation`：Claude生成の学習補助用仮訳（正式フィールド名）
- `paragraphs`：詩の場合は1行＝1段落として格納。スタンザ結合はUI側で処理予定
- `relatedTexts`：関連テキストIDの配列。作家横断の共鳴関係を記述する

### 旧フォーマット（廃止）
ラッパーキーで囲む形式。新規作成はすべて新フォーマットで行う。

---

## ファイル構成

```
french-critique-app/
├── package.json
├── vite.config.js
├── tailwind.config.js
├── postcss.config.js
├── index.html
└── src/
    ├── main.jsx
    ├── index.css
    ├── App.jsx
    └── data/
        ├── baudelaire/
        │   ├── index.js
        │   └── *.json（28ファイル）
        ├── mallarme/
        │   ├── index.js
        │   └── *.json（22ファイル）
        ├── valery/
        │   ├── index.js
        │   └── *.json（2ファイル）
        ├── valmore/
        │   ├── index.js
        │   └── *.json（14ファイル）
        ├── vanlerberghe/
        │   ├── index.js
        │   └── *.json（原典確認済み7ファイル）
        └── verlaine/
            ├── index.js
            └── *.json（6ファイル）
```

### index.js（各作家共通）
```javascript
const modules = import.meta.glob('./*.json', { eager: true });
const texts = {};
for (const path of Object.keys(modules)) {
  const entry = modules[path].default;
  if (entry && entry.id) texts[entry.id] = entry;
}
export default texts;
```

---

## 決まっている仕様・ルール

### キー命名規則
- **テキストID**: `{著者姓小文字}_{テキスト識別子}` 例: `vanlerberghe_entrevision`
- **カテゴリーID**: `{著者姓}_{分野}` または `{著者姓}` 例: `valmore`
- **ファイル名**: テキストIDのサフィックス部分 例: `entrevision.json`

### カテゴリー体系
```javascript
const categories = {
  baudelaire_aesthetics:    'ボードレール美学',
  baudelaire_music:         'ボードレール音楽論',
  baudelaire_modernity:     'ボードレール近代性',
  mallarme_poetics:         'マラルメ詩学',
  mallarme_book:            'マラルメ書物論',
  mallarme_representation:  'マラルメ表象論',
  mallarme_theatre:         'マラルメ演劇・表象論',
  mallarme_music:           'マラルメ音楽論',
  mallarme_culture:         'マラルメ文化論',
  valery:                   'ヴァレリー',
  valmore:                  'ヴァルモール',
  vanlerberghe:             'ヴァン・レルベルグ',
  verlaine_critique:        'ヴェルレーヌ批評',   // 今回追加
};
```

### 著者別カラーバッジ（テキストカード）
| 著者 | 色 |
|---|---|
| Baudelaire | アンバー（amber） |
| Mallarmé | スカイ（sky） |
| Valéry | バイオレット（violet） |
| Desbordes-Valmore | ピンク（pink） |
| Van Lerberghe | エメラルド（emerald） |
| Verlaine | ローズ（rose） |

### UI設計方針
- 原文ラベル: 青系（indigo）/ 仮訳ラベル: 緑系（green）/ 自分の訳ラベル: 紫系（purple）
- 原文テキストにイタリックは適用しない（Garamond・Noto Serif・Sans すべて非イタリック）
- レスポンシブ対応・ダークモード対応

---

## 実装済みUI仕様（2026-02-20 更新）

### 検索バー
- ヘッダー直下に配置
- タイトル・著者・keywords・french・provisionalTranslation（およびofficialTranslation）を横断検索
- 検索入力時はカテゴリーフィルターを「すべて」にリセット
- プレースホルダー：「本文テキストで検索」

### テキストカード（一覧）
- 著者別カラーバッジを表示
- カード本体クリックでテキスト選択
- 右上の「∨」ボタンで本文セクションへスクロール（2回押し方式）
  - 1回目：ボタンが indigo に変色・拡大（`readyToScroll` ステートにIDをセット）
  - 2回目：`bodyRef` を用いて `scrollIntoView` を実行。テキスト未選択の場合は先に切り替えてから 80ms 後にスクロール
  - 別カードの「∨」を押すと前のアクティブ状態はリセット

### 段落アコーディオン
- 各段落はデフォルト折りたたみ（行番号＋原文冒頭のみ表示）
- 「すべて展開」「すべて折りたたむ」ボタンで一括操作
- 自分の訳がある段落には折りたたみ時も紫ドットを表示

### 設定パネル（⚙️ボタン）
- フォントサイズ・フォント切り替え・原文／仮訳／自分の訳の表示切り替えを集約
- パネル外クリックで自動的に閉じる

---

## 現在の収録状況（2026-02-20 更新）

### ボードレール（28ファイル）

**美学・近代性**
| ファイル名 | id | カテゴリー | 備考 |
|---|---|---|---|
| `romantisme_complet.json` | `baudelaire_romantisme_complet` | aesthetics | Salon de 1846 第2章・完全版 |
| `rire.json` | `baudelaire_rire` | aesthetics | |
| `reine.json` | `baudelaire_reine` | aesthetics | |
| `delacroix_1.json` | `baudelaire_delacroix_1` | aesthetics | ドラクロワ論①天才とメランコリー |
| `delacroix_2.json` | `baudelaire_delacroix_2` | aesthetics | ドラクロワ論②色彩と音楽 |
| `delacroix_3.json` | `baudelaire_delacroix_3` | aesthetics | ドラクロワ論③素描と色彩 |
| `delacroix_4.json` | `baudelaire_delacroix_4` | aesthetics | ドラクロワ論④偉大さと結論 |
| `valmore_critique.json` | `baudelaire_valmore_critique` | aesthetics | ヴァルモール夫人論（1861）完全版 |
| `heroisme.json` | `baudelaire_heroisme` | modernity | 完全版に差し替え済み（11段落） |
| `peintre_1.json` | `baudelaire_peintre_1` | modernity | 近代生活の画家①美・モード・幸福 |
| `peintre_2.json` | `baudelaire_peintre_2` | modernity | 近代生活の画家②風俗スケッチ |
| `peintre_3.json` | `baudelaire_peintre_3` | modernity | 近代生活の画家③芸術家・群衆・幼年性 |
| `peintre_4.json` | `baudelaire_peintre_4` | modernity | 近代生活の画家④近代性の定義 |
| `peintre_5.json` | `baudelaire_peintre_5` | modernity | 近代生活の画家⑤構成の方法 |
| `peintre_6.json` | `baudelaire_peintre_6` | modernity | 近代生活の画家⑥戦争画論 |
| `peintre_7.json` | `baudelaire_peintre_7` | modernity | 近代生活の画家⑦祝祭・式典 |
| `peintre_8.json` | `baudelaire_peintre_8` | modernity | 近代生活の画家⑧軍人論 |
| `peintre_9.json` | `baudelaire_peintre_9` | modernity | 近代生活の画家⑨ダンディ論 |
| `peintre_10.json` | `baudelaire_peintre_10` | modernity | 近代生活の画家⑩女性論 |
| `peintre_11.json` | `baudelaire_peintre_11` | modernity | 近代生活の画家⑪化粧礼讃 |
| `peintre_12.json` | `baudelaire_peintre_12` | modernity | 近代生活の画家⑫女たち論 |
| `peintre_13.json` | `baudelaire_peintre_13` | modernity | 近代生活の画家⑬乗り物論 |
| `lettre_wagner.json` | `baudelaire_lettre_wagner` | music | |
| `wagner_1.json` | `baudelaire_wagner_1` | music | |
| `wagner_2.json` | `baudelaire_wagner_2` | music | |
| `wagner_3.json` | `baudelaire_wagner_3` | music | |
| `wagner_4.json` | `baudelaire_wagner_4` | music | |
| `encore.json` | `baudelaire_wagner_encore` | music | |

> ⚠️ 旧`peintre.json`（`baudelaire_peintre`）は`peintre_1.json`で上書き済み。旧`romantisme.json`は`romantisme_complet.json`と共存中（整理候補）。

### マラルメ（22ファイル・149段落）
| カテゴリー | テキスト数 |
|---|---|
| mallarme_poetics | 16編 |
| mallarme_theatre | 4編（Ballets, Les Fonds, Hamlet, Crayonné） |
| mallarme_music | 2編（Wagner rêverie, Plaisir sacré） |

### ヴァレリー（2ファイル・12段落）
現在2編。5編程度追加が目標。

### ヴァルモール夫人（14ファイル）
| ファイル名 | タイトル | 備考 |
|---|---|---|
| `roses.json` | Les Roses de Saadi | 代表作 |
| `adieu.json` | L'Adieu | 初期 |
| `chambre.json` | Ma chambre | 日常・内面 |
| `qu_en_avez.json` | Qu'en avez-vous fait ? | リフレイン詩 |
| `separees.json` | Les Séparées | 別離 |
| `mal_du_pays.json` | Le Mal du pays | 望郷 |
| `reve_intermittent.json` | Rêve intermittent d'une nuit triste | 母・夢 |
| `insomnie.json` | L'Insomnie | 初期 |
| `fleurs.json` | Les Fleurs | 花・祈り |
| `couronne.json` | La Couronne effeuillée | 晩年 |
| `a_monsieur.json` | À Monsieur A. D. | 愛の告白 |
| `priere_fils.json` | Prière pour mon fils | 母性 |
| `avis.json` | Avis | 序詩。ボードレール引用詩句の出典 |
| `tristesse.json` | Tristesse | ボードレール引用詩句の出典 |

### ヴァン・レルベルグ（原典確認済み7ファイル）
| ファイル名 | タイトル | 初出／出典 | 行数 |
|---|---|---|---|
| `entrevision.json` | Entrevision | L'Art jeune, 1895 | 30行 |
| `solitude.json` | Solitude | Parnasse de la jeune Belgique, 1887 | 16行 |
| `invocation.json` | Invocation | La Pléiade, 1886 | 14行 |
| `rosier_mystique.json` | Le Rosier Mystique | Entrevisions, 1898 | 20行 |
| `sous_les_arches.json` | Sous les arches de roses | Entrevisions, 1898 | 16行 |
| `tombee_du_soir.json` | Tombée du soir | Entrevisions, 1898 | 17行 |
| `la_mort.json` | La Mort | Entrevisions, 1898 | 16行 |

**⚠️ 要削除ファイル（原典未確認・旧生成）：**
`entrevisions.json`, `silence.json`, `soir.json`, `arbres.json`, `pan.json`

### ヴェルレーヌ（6ファイル・新規作家）
『呪われた詩人たち』全6章を収録。カテゴリー: `verlaine_critique`

| ファイル名 | id | 章 | 対象作家 | 版 |
|---|---|---|---|---|
| `poetes_maudits_corbiere.json` | `verlaine_poetes_maudits_corbiere` | Ⅰ | コルビエール | 1884年初版 |
| `poetes_maudits_rimbaud.json` | `verlaine_poetes_maudits_rimbaud` | Ⅱ | ランボー | 1884年初版 |
| `poetes_maudits_mallarme.json` | `verlaine_poetes_maudits_mallarme` | Ⅲ | マラルメ | 1884年初版 |
| `poetes_maudits_valmore.json` | `verlaine_poetes_maudits_valmore` | Ⅳ | ヴァルモール | 1888年増補版 |
| `poetes_maudits_villiers.json` | `verlaine_poetes_maudits_villiers` | Ⅴ | ヴィリエ・ド・リラダン | 1888年増補版 |
| `poetes_maudits_lelian.json` | `verlaine_poetes_maudits_lelian` | Ⅵ | パーヴル・レリアン（自己論） | 1888年増補版 |

### 総計
**約79ファイル・500段落以上**

---

## テキスト間の主要な共鳴関係（relatedTexts）

| 起点テキスト | 接続先 | 共鳴の性質 |
|---|---|---|
| `baudelaire_valmore_critique` | `valmore_avis`, `valmore_tristesse`, `verlaine_poetes_maudits_valmore` | ヴァルモール夫人を論じる批評の三角形 |
| `verlaine_poetes_maudits_mallarme` | `mallarme/`（全22編） | 批評←→作品の対照読解 |
| `baudelaire_delacroix_2` | `baudelaire_wagner_1`, `baudelaire_romantisme_complet` | 色彩＝音楽論の連鎖 |
| `baudelaire_peintre_4` | `baudelaire_heroisme`, `baudelaire_peintre_1` | 「近代性」定義の展開 |
| `verlaine_poetes_maudits_villiers` | （リラダン収録時に接続予定） | 呪われた詩人論の拡張 |

---

## 今後の作業候補

### データ拡充（優先度順）

| 作家・テキスト | 方針 | 備考 |
|---|---|---|
| ヴァン・レルベルグ追加詩 | Gallicaスクリーンショット方式 | Entrevisions PDF入手済み |
| ヴァレリー追加（現2→5編目標） | Claudeの知識ベース | |
| ローデンバック | 検討中 | アプリ方向性と合致 |
| グールモン | 検討中 | Litanies de la Rose等 |
| ヴィリエ・ド・リラダン詩 | Gallicaで原典確認後 | Premières poésies（1859）等 |
| ボードレール「De la couleur」 | Claudeの知識ベース | Salon de 1846 |

### 機能面

| 機能 | 状態 |
|---|---|
| 行結合UI（行／スタンザ／自由結合） | 検討中 |
| ブックマーク機能 | 候補 |
| PWA対応 | 候補 |
| TypeScript化 | 長期 |

---

## 開発メモ

### よく使うコマンド
```bash
npm run dev
npm run build
git add . && git commit -m "message" && git push
```

### トラブルシューティング
- **PostCSS設定エラー**: `.cjs` 拡張子を使用
- **JSON構文エラー**: トップレベルが直接オブジェクト（`{` で始まり `}` で終わる）
- **importエラー**: ファイルパスとファイル存在を確認
- **フォントイタリック**: 原文 `<p>` へのイタリック適用は削除済み。新規UI実装時も適用しないこと

### iPhoneからの編集
- github.dev（URLの `github.com` → `github.dev`）推奨

### 原典確認フロー（流通量の少ない詩人）
1. Gallicaで該当詩集を検索・PDF入手
2. 該当ページをスクリーンショット
3. ClaudeにSSを貼り付けてJSON化依頼
4. `firstPublication` フィールドに初出誌情報を記録

### 新しい作家ディレクトリを追加する手順
1. `src/data/{author}/` ディレクトリを作成
2. `index.js`（自動読み込み共通形式）を配置
3. JSONファイルを配置
4. `App.jsx` のデータ集約箇所に `import` を追加
5. `categories` オブジェクトに新カテゴリーを追加
6. 著者別カラーバッジの色定義を追加

---

最終更新: 2026-02-20（このセッションでの大規模更新）
