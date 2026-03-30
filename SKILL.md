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
    - **【注意】** `<mxGeometry as="geometry" />` 要素を作成する際、Pythonの予約語 `as` をキーワード引数として直接渡すことはできません。`ET.Element('mxGeometry', {'as': 'geometry', ...})` のように辞書型で属性を指定し、決して `as_` のまま出力しないようにしてください（Draw.ioの読み込みエラーになります）。
5. 作成したXMLを `.drawio` ファイルとして保存する。

## 3. Draw.io XML（mxGraph）の構成とスタイル
スクリプト内で定義するスタイルは以下の構造を遵守してください。

### スタイル定義の基本
- **アイコン（Vertex）**: 
  - `style`: `shape=image;image=data:image/svg+xml,[URL_ENCODED_SVG_DATA];verticalLabelPosition=bottom;verticalAlign=top;aspect=fixed;`
  - 注意: `base64,` ではなくカンマ `,` に続けてURLエンコードされた文字列をそのまま配置します。
  - **絶対に `shape=mxgraph.gcp2...` などの標準図形を使わず、`shape=image;image=data:image/svg+xml,...` でローカルのSVG資産を埋め込んでください。**
  - ラベルはアイコンの下に配置する。
- **コンテナ（VPC/Subnet/Group）**:
  - `style`: `shape=rect;fillColor=none;strokeColor=#CCCCCC;dashed=1;rounded=1;verticalAlign=top;align=left;spacingLeft=35;fontStyle=1;fontSize=12;`
  - **ヘッダー表現**: コンテナの左上に、その役割を示す小さなアイコン（24x24程度）を配置し、ラベルの `spacingLeft` で重ならないように調整します。
  - **プロバイダー別カラー**:
    - AWS: `strokeColor=#242F3E;` (VPC), `strokeColor=#0073BB;` (Public Subnet), `strokeColor=#00A4A6;` (Private Subnet)
    - GCP: `strokeColor=#4285F4;`
    - Azure: `strokeColor=#0089D6;`
- **コネクタ（Edge）**:
  - `style`: `edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;strokeColor=#555555;`
  - 直角（Orthogonal）な線を使用し、接続点を自動調整する。

## 4. 設計原則（参考図に基づく）
- **論理的グループ化と階層構造**:
  - **Cloud/Region**: 最上位の枠。プロバイダーのロゴを左上に配置する。
  - **VPC/Virtual Network**: ネットワークの境界。太めの実線または点線で表現し、VPCアイコンを添える。
  - **Subnet**: パブリック/プライベートの区別を明確にする。鍵アイコン（Lock）や色分け（パブリックは青/緑系、プライベートは青/青緑系）を用いて視覚的に区別する。
- **配置の整合性**:
  - アイコンサイズは標準 60x60、コンテナ内アイコンは 48x48 など、階層に応じて調整する。
  - コンテナ内のリソースは、上下左右に適切なマージン（例：40px以上）を持たせる。
- **ラベル**: 日本語のラベルを適切に付与し、フォントサイズを読みやすく調整する（タイトルは太字）。

## 5. 実行プロセス
1. **要求分析**: ユーザーが必要としているアーキテクチャの構成要素を抽出する。
2. **スクリプト作成**: 構成要素の座標、アイコン検索、URLエンコード化、XML構築を行うPythonスクリプトを作成し、`run_shell_command` ツールで実行する。
3. **検証**: スクリプトがエラーなく `.drawio` ファイルを生成したことを確認する。

## 6. プロンプト例
- 「AWSで、VPC内に2つのパブリックサブネットと2つのプライベートサブネットを持ち、ALBからEC2に負荷分散し、RDSに接続する構成図を描いて」
- 「GCPのGKEを中心としたCI/CDパイプラインの図を作成して。GitHub, Cloud Build, Artifact Registryを含めて」
- 「AzureのHub-and-Spoke構成図を描いて。ExpressRouteとVPN Gatewayを含めること」

