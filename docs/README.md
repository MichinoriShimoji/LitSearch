# Literature Pipeline (static)

ブラウザだけで動く文献収集ツール。GitHub Pages からそのまま使えます。
**サーバー不要・データはローカルに留まる・成果物はZIPで一括ダウンロード**。

## 基本フロー（3ステージ）

### Stage 1 — キーワード検索
1. ページを開く
2. 検索条件を入力 → **検索**（Crossref + OpenAlex を並列）
3. 候補をその場で編集して「本体へ追加」 or 「候補を全件本体へ追加」

### Stage 2 — 孫引き一括抽出（任意）
4. 本体CSVに DOI付きレコードが溜まったら、サイドバーの **「本体の全レコードから参考文献を一括抽出」** を押す
5. 各親論文の Crossref reference 配列を取得し、参考文献を1件ずつ自動再検証
6. DOI lookup または title 検索で確実な一致が取れたものは「確認済」として本体に追加
7. 失敗したものは **未検証フラグ付き**で本体に追加（カードが黄色く強調表示）

### Stage 3 — 確認・ダウンロード
8. 黄色いカードを編集して「確認済みにする」 or 「削除」
9. **ZIPでまとめてダウンロード** で完成版を取得
10. （任意）**Zoteroへ直接送信** で Zotero Desktop に取り込み

## 孫引き再検証（Stage 2）

サイドバーの「**本体の全レコードから参考文献を一括抽出**」ボタンで実行します。各カードに個別ボタンはありません。

### 抽出ソースの優先順位

1. **PDF優先**：`pdf_url` がある場合、PDF.js でブラウザから直接フェッチ → テキスト抽出 → References セクションを正規表現で同定 → 個別の文献に分解
2. **Crossrefフォールバック**：PDF取得失敗 or `pdf_url` 未設定の場合、Crossref `reference` メタを使用

### 各参考文献の検証

抽出された参考文献は1件ずつ自動検証：

- DOI付き → Crossref で直接 lookup → 成功なら**確認済**として本体に追加
- DOIなし → unstructured text から author/year/title を抽出 → Crossref+OpenAlex で検索
- 「タイトル類似度＋著者＋年」で confident match を判定 → 成功なら確認済として追加
- どちらも失敗 → **未検証フラグ付き**で本体に追加（カードが黄色く強調表示）

進行状況はステータスに「Stage 2 [pdf] — 親 3/12 (10.xxxx/...) — 参考文献 17/47 検証中... (累計: 確認済 32 / 未検証 11)」とリアルタイム表示。

### CORS の現実

- arXiv / PMC など Open Access リポジトリは PDF を直接取得できることが多い
- 出版社直リンク（Elsevier、Wiley等）は CORS でブロックされることが多い → Crossrefにフォールバック
- PDF が読めなかったか、Crossref に reference メタがない場合、その親はバッジに「孫引失敗」と表示

### カードのバッジ

- 📄 **PDF孫引済** — PDFから参考文献を抽出
- **Crossref孫引済** — Crossref メタデータから抽出
- **孫引失敗** — どちらも取れなかった（ホバーで理由表示）

### 未検証カードの扱い

未検証カード（黄色いボーダー＋⚠アイコン）には：
- **失敗理由**：DOI lookup 失敗 / タイトル検索で確実な一致なし / パース不能 など
- **原文（unstructured text）**：参考文献リストにあった元の文字列
- **親論文DOI**：どの論文の参考文献由来か

ユーザは ZIPダウンロード前に：
- フィールドを編集して **「確認済みにする」** で黄色フラグを外す
- **「削除」** で消去する
- 編集を保留したまま **ZIP** → 確認済みは本体CSV、未検証は `unverified.csv` として別ファイルで同梱（マニフェストにも明記）

### ZIPに入るファイル一覧

| ファイル | 内容 |
|---|---|
| `<project>_literature.csv` | 確認済レコードのみ |
| `<project>_literature.bib` | 確認済の BibTeX |
| `<project>_zotero_import_tagged.bib` | 確認済の BibTeX（keywords付き） |
| `<project>_pdf_urls.md` | 確認済の PDF URL リスト |
| `<project>_unverified.csv` | 未検証項目（あれば、`unverified` 列付き） |
| `README.md` | 生成日時・件数・ファイル説明 |

## ZIPに入るもの

| ファイル | 内容 |
|---|---|
| `<project>_literature.csv` | 本体レコードCSV |
| `<project>_literature.bib` | BibTeX（タグなし） |
| `<project>_zotero_import_tagged.bib` | BibTeX（Zotero用 keywords付き） |
| `<project>_derivative_candidate_verification.csv` | 未確認候補CSV |
| `<project>_pdf_urls.md` | PDF URLリスト（Markdown） |
| `README.md` | 生成日時とファイル一覧 |

## Zoteroへの送信

「Zoteroへ直接送信」ボタンを押すと、Zotero Desktop の `http://127.0.0.1:23119` に BibTeX を POST します。
- **Zotero Desktop を起動**しておく必要があります
- Chrome / Firefox / Edge は HTTPS ページから localhost への通信を許可します（自動）
- Safari は厳しめなので動かないことがあります → その場合は ZIP から `.bib` をダウンロードして手動 import

手動 import の場合：Zotero Desktop で **File → Import...** から `.bib` を選択。

## データの保持

作業データは**ブラウザの localStorage** に保存されます（プロジェクトID単位で別キー）。
- 異なるブラウザ・端末へは引き継がれません
- 念のため定期的に「ZIPでまとめてダウンロード」して、手元に保存しておくのが安全
- 「個別ダウンロード／既存CSVを取り込む」を開けば、既存CSVのアップロードや localStorage のクリアも可能

## プロジェクトの切り替え

「プロジェクトID」を変えると localStorage の別キーで作業が始まります。
- `irabu` → `irabu_dialect_literature.csv` / `irabu_dialect_literature.bib` などの命名
- それ以外（例：`tsugaru`） → `tsugaru_literature.csv` / `tsugaru_literature.bib` などの命名

## 制約・注意

- 検索は **Crossref + OpenAlex** のみ。CiNii、NDL、J-STAGE、Scholar、ResearchGate、Academia.edu、researchmap は左の「確認用リンク」で別タブを開いて手動確認
- PDF 自動ダウンロード・添付はなし（URL を控えて手動）
- 状態は localStorage 限定。複数端末で同期したい場合は ZIP を持ち運ぶ

## GitHub Pages の有効化（オーナー向け）

リポジトリの **Settings → Pages**：
- Source: **Deploy from a branch**
- Branch: `main` / Folder: `/docs`

数分で `https://<username>.github.io/<repo>/` に公開されます。

## 開発メモ

- 単一HTMLファイル（`index.html`）に CSS/JS を全部入れ
- 外部依存は **JSZip 3.10.1**（CDN）だけ
- 検索API: Crossref / OpenAlex（いずれもCORS対応）
