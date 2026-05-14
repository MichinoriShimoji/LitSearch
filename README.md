# LitScope

ブラウザだけで動く、**日本語・英語の方言／言語学文献の総合検索システム**。

## 公開URL（GitHub Pages）

🔗 **https://michinorishimoji.github.io/LitSearch/**

## できること

- **松岡葵氏作成「地域別方言文献リスト」（11,000件超）** を基礎データベースとして組み込み
- **CiNii Research / OpenAlex / Crossref** のリアルタイム検索を並列に追加
- 重複は自動で1件に統合（タイトル＋年＋著者ベース、サブタイトル差・スクリプト差にも対応）
- ヒット論文の参考文献を PDF（PDF.js）または Crossref メタから自動取得し、関連論文まで一気に拡張
- 結果カードは編集／削除可。**ZIP** で CSV ／ BibTeX ／ Zotero用 BibTeX ／ PDFリンク一覧を取り出せる
- Zotero 用 BibTeX を出力（File → Import またはドラッグ＆ドロップで取り込み）
- すべてブラウザ内で完結。サーバー送信なし

## ファイル構成

| ファイル | 内容 |
|---|---|
| `docs/index.html` | ツール本体（self-contained, ES module）|
| `docs/data/hougen_search_results.csv` | 松岡葵氏作成 地域別方言文献リスト |
| `irabu_dialect_literature.csv` / `.bib` | 伊良部島方言の文献リスト |
| `zotero_import_irabu_tagged.bib` | Zotero 取り込み用（tag付き） |
| `irabu_derivative_candidate_verification.csv` | 未確認候補 |
| `shiiba_*.csv` / `.bib` | 椎葉方言（初期段階） |

PDFs は著作権上、リポジトリには含めません（`.gitignore`）。

## 著作・データ提供

- ツール **LitScope** 著作：© 2026 **下地理則**（九州大学）
- 基礎データ「地域別方言文献リスト」：松岡葵 氏 作成

## ライセンス

ツールコード（`docs/index.html`）は研究・教育目的での利用可。文献リストデータは公開済み書誌メタデータ由来。
