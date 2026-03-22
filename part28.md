## ビジネスメタをオントロジーで書く - Part 28

今回は本来ならクラスにプロパティを付ける作業の続きだが、Claude Codeが余りにもノリノリなので、試しに書かせてみた「すごくtech savvvyなVCのパートナー向けpitch」の内容を「軽いjokeとして」載せることにする。

# Semantic Development Platform: Ontology-Based Enterprise Intelligence

---

## Executive Summary

### The Opportunity

データカタログ市場は存在するが、**誰も満足していない**。

- データ部門：メタデータを管理するだけのツール（使用頻度低）
- 開発部門：Claude Codeで開発しているが、命名がブレる、ドメイン知識が反映されない
- EA部門：Excel/Visio地獄、LLMが読めない

**私たちのソリューション：**
オントロジー（知識グラフ）をコアに、**3部門すべてに価値を提供**する統合プラットフォーム。

### Why Now?

1. **LLMブーム**：Claude Code、Cursor等の普及で開発者がAI支援を前提に
2. **Agentic AI**：マルチエージェントが構造化された知識基盤を必要としている
3. **MCP標準化**（2024年11月）：LLMが外部知識にアクセスする標準プロトコル確立
4. **Palantirの市場教育**：オントロジーの価値は証明済み。ただし年間数十億円で手が届かない

### Market Size

- **TAM**: データカタログ（$2B） + 開発者ツール（$10B） + EA管理（$3B） = **$15B+**
- **SAM**: 中堅企業（売上100億円〜）の3部門
- **SOM**: 日本市場、tech-forward企業から

### Traction

- 技術的実現性：全要素技術は存在（gist, ArchiMate, OBDA, MCP）
- 市場検証：Northwindでの実証実験完了
- タイミング：Palantirが市場を温めた今がベストタイミング

---

## Problem: 3つの断絶

### 1. データ部門の孤立

**現状：**
```
データカタログ → テーブル/カラムの平文annotation
              → 誰も見ない、更新されない
              → LLMが読めない（非構造化）
```

**問題：**
- 機械可読性ゼロ
- 物理スキーマに依存（変更で崩壊）
- データ部門以外は使わない

### 2. 開発部門の混乱

**現状：**
```python
# Claude Codeに依頼：「注文処理を書いて」
class OrderProcessor:
    def calc_total(self, order_id):  # calc? calculate? compute?
        items = get_order_items(order_id)  # items? details? lines?
        return sum(i.price * i.qty for i in items)  # qty? quantity?
```

**問題：**
- 命名がGenAI任せ（実行ごとにブレる）
- ドメイン知識が反映されない
- チームで用語が統一されない

### 3. EA部門の Excel 地獄

**現状：**
```
- Excel: システム一覧、インターフェース表
- PowerPoint: 業務フロー図
- Visio: ネットワーク図

問題：
✗ バラバラ（統合されていない）
✗ 更新されない（メンテ地獄）
✗ 検索できない
✗ LLMが読めない
```

---

## Solution: Ontology as Enterprise OS

### コア技術：オントロジー（知識グラフ）

**オントロジーとは：**
「概念」と「関係」を形式的に記述したもの。RDB のようなテーブルではなく、グラフ構造。

```turtle
# オントロジーの例（Turtle記法）
nw:Order rdfs:subClassOf gist:Agreement ;
         rdfs:label "注文"@ja ;
         gist:hasParty nw:Customer .

nw:OrderDetails rdfs:subClassOf gist:OrderedMember ;
                gist:isMemberOf nw:Order .
```

**なぜオントロジーか：**
1. **意味と実装の分離**：物理スキーマが変わっても概念は不変
2. **機械可読**：LLMが直接読める構造化データ
3. **標準ベース**：ベンダーロックインなし（後述）
4. **推論可能**：関係性を辿って情報発見

### 3部門への価値提供

```
┌────────────────────────────────────┐
│ データ部門                          │
│  - 構造化されたメタデータ           │
│  - SPARQL でクエリ可能              │
│  - LLMが読める                     │
└────────────────────────────────────┘
          ↓ 同じオントロジー
┌────────────────────────────────────┐
│ 開発部門                            │
│  - コード生成（命名統一）           │
│  - API設計支援                     │
│  - ドメインモデル管理               │
└────────────────────────────────────┘
          ↓ 同じオントロジー
┌────────────────────────────────────┐
│ EA部門                             │
│  - ArchiMate統合                   │
│  - Context Graph（LLM用）          │
│  - 影響分析                        │
└────────────────────────────────────┘
```

**市場規模が3倍に拡大。**

### 具体例：オントロジーベースのコード生成

**プロンプト：**
```python
# オントロジーをコンテキストに含める
prompt = f"""
以下のオントロジーに基づいてコード生成：

nw:Order rdfs:subClassOf gist:Agreement .
nw:OrderDetails rdfs:subClassOf gist:OrderedMember .
gist:hasTotalMagnitude rdfs:label "総額" .

注文の合計金額を計算する処理を書いてください。
"""
```

**生成されるコード：**
```python
class Order(Agreement):  # ← オントロジー由来
    """注文"""

    def get_total_magnitude(self) -> Decimal:  # ← gist標準
        """総額を計算"""
        order_details = self.get_ordered_members()
        return sum(detail.unit_price * detail.quantity
                   for detail in order_details)
```

**メリット：**
- 命名が一貫（実行ごとにブレない）
- ビジネス用語と一致
- チーム全体で統一された用語

---

## Market Opportunity

### Palantir が証明した市場

**Palantir Foundry:**
- オントロジーベースのデータ統合プラットフォーム
- 顧客：政府機関、超大手金融・製造
- **価格：年間数億円〜数十億円**

**Palantir の功績：**
オントロジーの価値を超大手企業で証明した。

**Palantir の限界：**
- 価格が高すぎる（中堅企業は手が届かない）
- 円安で実質40%値上がり（¥110 → ¥150）
- 独自形式（ベンダーロックイン）
- データ分析特化（開発支援なし）

### 私たちのポジショニング

```
        │ 高価格
        │
Palantir├────────┐
        │        │ ← 超大手向け
────────┼────────┤   年間数十億円
本製品  │        │ ← 中堅企業向け
        │        │   月額50万円〜
────────┼────────┤   + 開発支援
Collibra│        │   + EA統合
        │ 低価格
     データ分析  開発支援
        のみ    + EA統合
```

**競合しない理由：**
- 価格帯：100倍の差（月額50万円 vs 年間数十億円）
- ターゲット：中堅企業 vs 超大手
- 用途：統合プラットフォーム vs 分析特化

### TCO比較（5年間）

| | Palantir | 本製品 |
|---|---|---|
| **初期導入** | 5-10億円 | 500-1000万円 |
| **年間ライセンス** | 15億円 | 600-3000万円 |
| **コンサル** | 5-10億円 | 1000-3000万円 |
| **5年間総額** | **80-100億円** | **5000万-2億円** |

**差額: 約50倍**

---

## Technical Differentiation

### 1. Open Standards vs Proprietary

**Palantir（Proprietary）:**
```
独自オントロジー → ブラックボックス
                → 移行不可能
                → ベンダーロックイン
                → 専用スキル（転職先で使えない）
```

**本製品（Open Standards）:**
```
gist (Semantic Arts) → オープンライセンス
ArchiMate (The Open Group) → 国際標準（ISO）
RDF/OWL (W3C) → Web標準
OBDA (Ontop) → オープンソース

→ いつでも移行可能
→ 標準スキル（転職市場で価値）
→ コミュニティ・学習リソース豊富
```

**ベンダーロックイン比較：**

| | Proprietary | Open Standard |
|---|---|---|
| **オントロジー** | 独自（非公開） | gist（MIT的） |
| **EA統合** | 独自形式に変換 | ArchiMate標準 |
| **移行コスト** | ほぼ再構築 | エクスポート可能 |
| **スキル** | ベンダー専用 | 業界標準 |

### 2. OBDA: 意味と実装の分離

**OBDA (Ontology-Based Data Access):**

```
┌─────────────────────────┐
│ オントロジー層           │ ← 概念レベル（普遍的）
│  nw:Order → gist:Agreement
└─────────────────────────┘
          ↓ マッピング
┌─────────────────────────┐
│ 物理層                  │ ← 実装レベル（変わりうる）
│  Orders テーブル (PostgreSQL)
└─────────────────────────┘
```

**メリット：**
- スキーマ変更してもオントロジーは不変
- 複数DBをまたぐクエリ（federation）
- 既存DBをそのまま活用（マイグレーション不要）

```turtle
# マッピング例（.obda）
mappingId: orders-mapping
target: :order_{OrderID} a nw:Order .
source: SELECT OrderID, CustomerID, OrderDate FROM Orders
```

### 3. Context Graph（ArchiMate統合）

**Context Graph とは：**
LLMが効果的に動作するために必要な情報を、グラフ構造で管理。

**従来のRAG（ベクトル検索）:**
```
質問 → ベクトル検索 → 平面的な類似文書 → LLM
         ↑ 関係性が見えない
```

**Context Graph:**
```
質問 → グラフトラバーサル → 構造化コンテキスト → LLM
         ↑ 関係性を辿る

[Order]
  ├─ depends on → [Customer]
  ├─ contains → [OrderDetails]
  ├─ implemented in → [order.py]
  ├─ tested by → [test_order.py]
  └─ managed by → [SalesTeam]
```

**ArchiMateとの統合：**

```turtle
# ビジネス層
:OrderManagementProcess a archimate:BusinessProcess ;
    archimate:isRealizedBy :OrderService .

# アプリケーション層
:OrderService a archimate:ApplicationService ;
    archimate:accesses :OrderDatabase .

# 実装層
:OrdersTable a archimate:Artifact ;
    owl:sameAs nw:Orders .  # ← オントロジーと紐付け
```

**ユースケース：影響分析**
```
User: "Ordersテーブルのスキーマ変更の影響は？"

Context Graph:
  OrdersTable
    ← isAccessedBy → OrderService
    ← isAccessedBy → ReportingService
  OrderService
    ← isUsedBy → SalesTeam
    ← isUsedBy → SupportTeam

結果: 影響範囲が視覚化され、テスト計画も自動生成
```

---

## Agentic AI & MCP: The Next Wave

### MCP (Model Context Protocol)

**背景：**
Anthropic が2024年11月に発表した標準プロトコル。LLMがアプリケーション/データソースにアクセスする統一規格。

**従来の問題：**
```
Claude → API1（独自仕様）
       → API2（独自仕様）
       → API3（独自仕様）
→ 統合が大変、エコシステム育たない
```

**MCP：**
```
Claude → MCP Server1（標準）
       → MCP Server2（標準）
       → MCP Server3（標準）
→ プラグアンドプレイ
```

### オントロジー = 最高の MCP サーバー

**本製品を MCP サーバーとして実装：**

```typescript
class OntologyMCPServer {
  // Resources: データソース
  async listResources() {
    return [
      { uri: "ontology://gist", name: "gist Upper Ontology" },
      { uri: "ontology://nw", name: "Northwind Domain" },
      { uri: "catalog://tables", name: "Data Catalog" },
      { uri: "archimate://ea", name: "Enterprise Architecture" }
    ];
  }

  // Tools: 機能
  async listTools() {
    return [
      { name: "sparql_query", description: "Execute SPARQL" },
      { name: "generate_code", description: "Generate code from ontology" },
      { name: "impact_analysis", description: "Analyze change impact" }
    ];
  }
}
```

**Claude Desktop から使う：**
```
User: "注文テーブルの構造を教えて"

Claude → MCP Server (Ontology)
       → SPARQL実行
       → 結果返却

Claude: "nw:Orders は gist:Agreement のサブクラスで、
         以下のプロパティを持ちます..."
```

### Agentic AI のための知識基盤

**Agentic AI：**
LLMが複数のツールを使い、自律的に複雑なタスクを実行。

**マルチエージェントの例：システム改修タスク**

```
User: "注文処理に配送追跡機能を追加"

Planning Agent:
  → オントロジー参照
  → 必要コンポーネント特定
     - nw:Orders (既存)
     - nw:Shippers (既存)
     - nw:DeliveryTracking (新規)

Architecture Agent:
  → ArchiMate Model 参照
  → 影響範囲分析
     - OrderService (修正)
     - ShippingService (連携)

Code Agent:
  → オントロジーから命名取得
  → DeliveryTrackingクラス生成（gist準拠）

Data Agent:
  → DDL生成
  → マイグレーションスクリプト

完了！全て自動、命名も一貫。
```

**なぜオントロジーが必要か：**
- エージェント間で共有される「世界モデル」
- 構造化された知識（ベクトル検索では不十分）
- 一貫した命名・概念

### Market Timing

**2024年後半〜2025年:**
- MCP標準化
- Agentic AIフレームワーク成熟

**2026年（現在）:**
- MCPサーバー数が増加中
- Agentic AIのエンタープライズ導入加速
- **構造化知識基盤への需要急増**

**本製品: 完璧なタイミング**

---

## Go-to-Market Strategy

### ターゲット顧客セグメント

#### セグメント1: 「Excel地獄」企業（短期）

**ペルソナ:**
- IT部門長、EA担当者
- システム一覧をExcelで管理
- 更新が大変、誰も見ない

**価値提案:**
> 「Excel地獄から脱却し、LLMが読める統合EA管理へ」

**チャネル:**
- EA関連カンファレンス
- IT部門長向けウェビナー

#### セグメント2: 「Claude Code世代」開発者（中期）

**ペルソナ:**
- LLMでコード生成する若手開発者
- 命名の一貫性に悩む

**価値提案:**
> 「オントロジーで命名統一、GenAI出力品質10倍」

**チャネル:**
- 技術ブログ（Qiita、Zenn）
- 開発者カンファレンス

#### セグメント3: 「RDF/OWL 使いたかった」層（長期）

**ペルソナ:**
- アカデミア出身エンジニア
- セマンティックWeb技術に興味
- 「仕事では使えない...」

**価値提案:**
> 「大学で学んだRDF/OWLが、エンタープライズで使える」

**チャネル:**
- セマンティックWebカンファレンス
- 大学との連携

### Pricing Model

**SaaS型（月額課金）:**

| プラン | 価格 | ターゲット |
|-------|------|----------|
| **Starter** | 50万円/月 | 小規模（〜100テーブル） |
| **Professional** | 150万円/月 | 中規模（〜500テーブル） |
| **Enterprise** | 300万円〜/月 | 大規模 + カスタマイズ |

**追加オプション:**
- オンサイトトレーニング
- コンサルティング
- カスタムオントロジー開発

### Revenue Projection（3年）

**前提:**
- Year 1: 20社（Starter中心）
- Year 2: 80社（Professional増）
- Year 3: 200社（Enterprise拡大）

**ARR:**
- Year 1: ¥120M（$800K）
- Year 2: ¥1.2B（$8M）
- Year 3: ¥4.8B（$32M）

---

## Social Impact: 死蔵された知識技術の活用

### 問題：学術界と産業界の断絶

**RDF/OWL の現状:**
```
大学で教えている:
✓ 東大、慶應、筑波 等
✓ セマンティックWeb理論
✓ SPARQL、オントロジー設計

企業での利用:
✗ 製薬（一部）
✗ 国会図書館（一部）
✗ 一般企業（ほぼゼロ）

学生の声:
「習ったけど使う場所がない」
「面接で話しても通じない」
```

**ArchiMate の現状:**
```
国際標準（ISO/IEC/IEEE 42020）
しかし日本では:
✗ 大手SIの一部のみ
✗ 書籍数冊のみ
✗ 大多数はExcel/Visio
```

**社会的損失：**
- 教育投資が回収されない
- 学生のスキルが死蔵される
- 日本の技術的ポテンシャルが活かされない

### 本製品の社会的意義

**1. 教育投資の回収**
```
大学 → RDF/OWL教育に投資
学生 → 学んだけど使えない（損失）
   ↓
本製品 → 使える場を提供（回収）
```

**2. 標準技術の普及**
```
現状: 独自フォーマット乱立（Excel、独自DB）
   ↓
本製品: 国際標準（gist, ArchiMate, RDF/OWL）
   ↓
結果: 日本企業の国際競争力向上
```

**3. AI時代のインフラ**
```
LLMブーム: でもデータは非構造化
知識グラフ: 学術界にはあるが産業界にない
   ↓
本製品: 知識グラフをエンタープライズに
   ↓
結果: AI時代の企業知識OS
```

### Mission Statement

```
┌────────────────────────────────────────┐
│ 「死蔵された知識技術を、              │
│  エンタープライズの価値に変える」    │
│                                        │
│ - RDF/OWLを実務で使える場を提供       │
│ - ArchiMateをLLM時代に適合させる      │
│ - 学術界と産業界の橋渡し              │
└────────────────────────────────────────┘
```

---

## Roadmap

### Phase 1: MVP（6ヶ月）

**機能:**
- オントロジー管理UI（段階的フォーム）
- OBDA（Ontop統合）
- 基本的なSPARQLクエリ
- LLM支援編集

**ターゲット:**
- データ部門
- アーリーアダプター（5-10社）

### Phase 2: 開発者支援（6ヶ月）

**機能:**
- コード生成（Python/TypeScript）
- 型定義の自動生成
- ドキュメント生成
- Claude Code統合

**ターゲット:**
- 開発部門
- Claude Code利用企業（20-30社）

### Phase 3: EA統合（6ヶ月）

**機能:**
- ArchiMate統合
- Context Graph
- 影響分析
- マルチレイヤーRBAC

**ターゲット:**
- EA部門
- 大手企業（50-100社）

### Phase 4: Agentic AI（6ヶ月〜）

**機能:**
- MCP Server実装
- マルチエージェント対応
- エージェント開発者向けSDK
- エコシステム構築

**ターゲット:**
- 全社展開
- エージェント開発者コミュニティ

---

## Why This Team?

### 技術的優位性

1. **オントロジー設計の実績**
   - Northwindでの実証実験完了
   - gist準拠のドメインモデル構築

2. **標準技術の深い理解**
   - RDF/OWL（W3C標準）
   - ArchiMate（ISO標準）
   - OBDA（学術的基盤）

3. **LLM統合の先見性**
   - MCP対応を最初から設計
   - Agentic AIのユースケース理解

### 市場洞察

1. **Palantirの限界を理解**
   - 価格、ロックイン、用途の制約
   - 中堅企業市場の空白

2. **3部門への拡大戦略**
   - データカタログの枠を超える
   - 市場規模3倍

3. **タイミングの見極め**
   - MCP標準化
   - Agentic AI実用化
   - Claude Code普及

---

## The Ask

### 資金調達

**シード/シリーズA:**
- 調達額: ¥500M - ¥1B（$3M - $7M）
- 用途:
  - プロダクト開発（エンジニア採用）
  - GTM（営業・マーケティング）
  - コミュニティ構築

### 期待する支援

**戦略的パートナーシップ:**
- エンタープライズ顧客への紹介
- 技術アドバイザー（セマンティックWeb、EA）
- 海外展開の支援

---

## Conclusion

### Key Takeaways

1. **市場機会**
   - Palantirが証明した市場の下位セグメント
   - 中堅企業（TAM $15B+）
   - 3部門展開で市場3倍

2. **技術的差別化**
   - Open Standards（gist, ArchiMate, RDF/OWL）
   - OBDA（意味と実装の分離）
   - Context Graph（ArchiMate統合）
   - MCP対応（Agentic AI時代の標準）

3. **なぜ今か**
   - LLMブーム（Claude Code普及）
   - Agentic AI実用化
   - MCP標準化
   - Palantirの市場教育完了

4. **社会的意義**
   - 死蔵された知識技術の活用
   - 教育投資の回収
   - AI時代の企業知識OS

### Final Message

> **「Palantir級の技術を、50分の1の価格で。**
> **しかもベンダーロックインなし。**
> **データ、開発、EAの3部門に価値提供。**
> **Agentic AI時代の企業知識OS。」**

---
