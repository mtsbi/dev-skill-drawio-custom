---
name: drawio-architecture-diagram
description: Creates professional cloud architecture diagrams (AWS, GCP, Azure, OCI, M365, Power Platform) in Draw.io (mxGraph) format using official SVG icons.
author: Gemini CLI
version: 1.0.0
---

# Draw.io Architecture Diagram Skill

本スキルは、AWS、GCP、Azure等の公式アイコンを使用して、Draw.io（mxGraph）形式のプロフェッショナルなアーキテクチャ図を作成・編集するための専門スキルです。

## 1. 概要
ユーザーの要求に基づき、クラウドサービス（AWS, GCP, Azure, OCI, M365, Power Platform）の公式アイコンを適切に配置し、論理的な構造（VPC, Subnet, Grouping）を持った構成図を生成します。

**【重要：禁止事項】**
- **AIが直接XMLテキストを出力することは厳禁です。**
- **`shape=mxgraph.gcp2...` のようなDraw.io組み込みの古いシェイプ名を使用することは厳禁です。**

代わりに、必ず以下の「Pythonスクリプトを生成・実行してDraw.io XMLを作成する」アプローチを採用してください。

## 2. 資産の利用とスクリプトへの組み込み
スキル内にある以下の資産を利用してアイコンを特定します。
- `assets/icons_manifest.json`: 全アイコンの製品名、プロバイダー、パス、キーワードを定義したインデックスファイル。

### Pythonスクリプトの必須要件
AIが生成するスクリプトは、以下のロジックを**必ず**含んで実行する必要があります。
1. `assets/icons_manifest.json` を読み込む。
2. 構成に必要なリソースのキーワードでリストを検索する（空白やアンダースコアの違いを吸収するロジックを含めること。例: `keyword.replace(" ", "_")`）。
3. 対象のSVGファイルを読み込み、**URLエンコード（`urllib.parse.quote(svg_content)`）** して文字列化する。（Base64はパースエラーの原因になるため使用しないこと）
4. `xml.etree.ElementTree` などを用いて mxGraph の XML構造を組み立てる。
5. 作成したXMLを `.drawio` ファイルとして保存する。

## 3. Draw.io XML（mxGraph）の構成とスタイル
スクリプト内で定義するスタイルは以下の構造を遵守してください。

### スタイル定義の基本
- **アイコン（Vertex）**: 
  - `style`: `shape=image;image=data:image/svg+xml,[URL_ENCODED_SVG_DATA];verticalLabelPosition=bottom;verticalAlign=top;aspect=fixed;`
  - 注意: `base64,` ではなくカンマ `,` に続けてURLエンコードされた文字列をそのまま配置します。
  - **絶対に `shape=mxgraph.gcp2...` などの標準図形を使わず、`shape=image;image=data:image/svg+xml,...` でローカルのSVG資産を埋め込んでください。**
  - ラベルはアイコンの下に配置する。
- **コンテナ（Grouping/VPC/Subnet）**:
  - `style`: `shape=rect;fillColor=none;strokeColor=#CCCCCC;dashed=1;rounded=1;verticalAlign=top;align=left;spacingLeft=10;fontStyle=1;`
  - 境界線は点線、角丸、背景なしを基本とする。
- **コネクタ（Edge）**:
  - `style`: `edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;`
  - 直角（Orthogonal）な線を使用し、接続点を自動調整する。

## 4. 設計原則（参考図に基づく）
- **論理的グループ化**: VPC、サブネット、リージョン、あるいは「メインシステム」「CI/CD」といった機能単位でコンテナ（Rect）を作成し、その中にリソースを配置する。
- **一貫性のあるレイアウト**: アイコンのサイズを統一し（例：60x60）、等間隔に配置する。
- **ラベル**: 日本語のラベルを適切に付与し、フォントサイズを読みやすく調整する。

## 5. 実行プロセス
1. **要求分析**: ユーザーが必要としているアーキテクチャの構成要素を抽出する。
2. **スクリプト作成**: 構成要素の座標、アイコン検索、URLエンコード化、XML構築を行うPythonスクリプトを作成し、`run_shell_command` ツールで実行する。
3. **検証**: スクリプトがエラーなく `.drawio` ファイルを生成したことを確認する。

## 6. プロンプト例
- 「AWSで、VPC内に2つのパブリックサブネットと2つのプライベートサブネットを持ち、ALBからEC2に負荷分散し、RDSに接続する構成図を描いて」
- 「GCPのGKEを中心としたCI/CDパイプラインの図を作成して。GitHub, Cloud Build, Artifact Registryを含めて」
- 「AzureのHub-and-Spoke構成図を描いて。ExpressRouteとVPN Gatewayを含めること」
