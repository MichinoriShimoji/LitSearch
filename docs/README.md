# LitScope

日本語・英語の方言／言語学文献の総合検索システム。GitHub Pages からブラウザだけで使えます。

🔗 https://michinorishimoji.github.io/LitSearch/

## 仕組み

| 検索ソース | 役割 |
|---|---|
| 松岡葵『地域別方言文献リスト』（11,000件超） | ローカル基礎データベース（書誌情報の優先源） |
| CiNii Research | 日本語雑誌・書籍章のリアルタイム検索 |
| OpenAlex | 国際的な学術論文メタデータ |
| Crossref | DOI ベースの正規メタデータ |

検索ボタン1回で4ソースを並列照会し、重複を自動マージ。ヒット論文の参考文献も PDF（PDF.js）または Crossref メタから自動取得して、関連論文まで拡張します。

## 基本フロー

1. ページを開く
2. キーワードまたは著者を入力 → **検索** ボタン
3. 自動で走る処理：
   - ローカルDB + CiNii + OpenAlex + Crossref を並列照会
   - 重複（DOI ・タイトル類似度・prefix 一致）を1件に統合
   - ヒット論文ごとに参考文献を取得（PDF優先 → Crossref フォールバック）
   - 取れた参考文献は DOI lookup / title 検索で再検証
4. カードを編集 or 削除して整える
5. **ZIPでまとめてダウンロード** で完成版を取得
6. Zotero に取り込みたい場合は、Zotero 用 BibTeX（`zotero_import_*.bib`）を File → Import またはドラッグ＆ドロップ

## 重要な設計

- **ローカルDBが書誌情報の優先源**：手動整備された誌名・巻号・ページが、Web 経由のメタを上書き
- **CiNii > Crossref/OpenAlex**：日本語タイトル ⇔ 英語タイトルの自動切替（CiNii の dc:language で判定）
- **重複統合**：DOI ・タイトル＋年＋著者・タイトル prefix・タイトル類似度の4軸でマージ
- **データはブラウザ内のみ**：localStorage も使わない。リフレッシュごとに完全リセット
- **ZIP 出力で永続化**：CSV / BibTeX / Zotero用 BibTeX / PDFリンク一覧 / 未検証CSV

## Zoteroへの取り込み

ZIP（または個別ダウンロード）から `.bib` を保存し、Zotero に **File → Import** または BibTeX ファイルを直接ドラッグ＆ドロップで取り込んでください。
- Zotero の CORS ポリシーにより、外部ドメイン（GitHub Pages 等）からの `/connector/import` 直接送信は不可
- `zotero_import_*.bib` は タグ（keywords）付きで出力されるので、取り込み後の整理が楽です

## CORS の現実

- arXiv / PMC など Open Access リポジトリは PDF を直接取得できることが多い
- 出版社直リンク（Elsevier、Wiley など）は CORS でブロックされる → Crossref メタにフォールバック
- 既知のCORS拒否ドメインは事前にスキップして無駄なリトライを避けます

## 著作・データ提供

- ツール **LitScope** © 2026 **下地理則**（九州大学）
- 基礎データ「地域別方言文献リスト」：**松岡葵 氏** 作成

## GitHub Pages の有効化（オーナー向け）

Settings → Pages：
- Source: Deploy from a branch
- Branch: `main` / Folder: `/docs`

## 技術メモ

- 単一HTMLファイル（`index.html`）に CSS/JS を全部入れ
- 外部CDN: JSZip 3.10.1 / PDF.js 3.11.174
- API: Crossref / OpenAlex / CiNii Research（いずれもCORS対応 JSON）
- PDFパース: PDF.js（HEAD pre-flight + ハードタイムアウト）
