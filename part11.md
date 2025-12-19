## ビジネスメタをオントロジーで書く - Part 11

### プレフィクスとネームスペース
前回では初見のRDFグラフデータがナニモノなのかを (LLMにしろ人間にしろ) 知るには、まずトリプル数を数え、それからクラスやプロパティのサンプルを取って、そこからプレフィクスとネームスペースを導出する、という手順を踏む、というお話をご紹介したが、この線でいえば抽象度が一番高い (数が少ない) のは最後の「プレフィクスとネームスペース」ということになろう。

この「プレフィクスとネームスペース」とはどんなものかというとこちらである。turtle形式なら大抵ファイルの一番最初に書いてある。

```
PREFIX :       <http://example.org/northwind#>
PREFIX ac:     <http://example.org/attribute-concepts#>
PREFIX ac-cat: <http://example.org/attribute-concepts/category/>
PREFIX arc:    <http://example.org/architypes#>
PREFIX ent:    <http://example.org/entity-concepts#>
PREFIX meta:   <http://example.org/meta#>
PREFIX nw:     <http://example.org/northwind#>
PREFIX owl:    <http://www.w3.org/2002/07/owl#>
PREFIX rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:   <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:    <http://www.w3.org/2001/XMLSchema#>
```

平たく言うと「プレフィクス」は「郵便番号」、「ネームスペース」は「住所表記」である。上では例えば`ac:`が郵便番号、`<>`で囲まれたURLみたいなの (RDFの世界ではURIと呼ぶ) が住所表記である。

そして「何の」プレフィクスか、といえば、RDFトリプルを成す概念的な要素、すなわち主語 (subject)、述語 (predicate)、目的語 (object) の、ということになる (目的語については全てではないが、その辺りは追々)。

「なぜ」プレフィクスが要るのか、といえば、「要素を特定するため」である。「タナカさん」だけでは郵便は届かない、というか特定できない。どの住所にお住いのタナカさんですか、となるのは当然である。

概念要素を「特定」できれば、ほかの概念要素との「区別」もつく。タナカさんが二人いても、ネリマのタナカさんとナカノのタナカさんとは別人である。タナカさんといった「名前」 が存在する、ネリマとかナカノといった「空間」なので、日本語で「名前空間」、横文字にすると「ネームスペース」、といった塩梅である。

以上はアナロジーだが、ビジネスで使う場合はこれは「語彙の領域」になる。製造業とサービス業で使われることば (語彙) はそれぞれ違う。場合によっては同じひとつの会社の中でも、本社と工場で同じことばを別の意味で使っていたりもする。一方例えば人事経理ITといったバックオフィス系では、使われる語彙は産業に関わらず共通点も多いだろう。また同じ産業であれば、会社が違っていても同じ意味で使われてる、いわゆる「業界用語」も当然存在する。

まあこれはDDDなどでいうところのドメインと似たようなもの、とも捉えられる。あるドメイン境界内に存在するドメイン語彙に付ける接頭語、である。但しこちらにはセマンティックな含意などはなく、あくまで単なるラベル、或いはシンボルである。

で、名前空間がなぜURLみたいなのか、というと「その接頭辞のついた語彙は全世界で一意である」ことの担保、である。`www.example.org`と書いてあるのはあくまで例であって、例えばこれを自社のURLにすれば、それで「この語彙の意味は当社固有である」という宣言になる。

下の4つは見てお気づきの通り、[W3C](https://ja.wikipedia.org/wiki/World_Wide_Web_Consortium) が作り、パブリックに公開している名前空間である。名前空間には上述のように自前で作るものと、W3Cのような第三者が標準として提供するものがある。そもそもRDF自体がW3Cが作った標準で、RDFを書くときは自ずと最低でも下の4つは参照することになる。

ここでこ思い出さなければならないのは、このシリーズの最初に書いた「W3Cの文書はLLMはすでに学習済」ということである。自前の語彙には説明が必要だが、W3Cの語彙の意味は逆に我々がGenAIに教わる立場である。

プレフィクスとネームスペースについてはなんとなく解った。しかし、RDFグラフが初見で、どんなプレフィックスやネームスペースが使われているか分からない場合、RDFトリプルを浚って導出する、というのはどう見ても非効率で、やはり合点がいかない。何10億トリプルとかでもそれやるんですか？と誰もが疑問に思うだろう。ということでもう一度よく調べてみると、「オントロジー・ヘッダー」なるそのための手法がすでにちゃんと存在する。しかもベストプラクティスだという (では最初になぜ「トリプルを全部浚って導出」とか答えたのかはナゾである)。

### オントロジー・ヘッダー
プレフィクスとネームスペースそのものはRDFトリプルではないので、SPARQLでは拾えない、というのが問題なので、であればそれらもRDFトリプルで書けばクエリで抜けるよね、という単純な話である。

まず、SHACLの語彙`sh:PrefixDeclaration`を使い、プレフィクス`ac:`と名前空間`http://example.org/attribute-concepts#`の1ペアを`INSERT`する。FusekiのWebUIはデータを読むためのものなので、`INSERT`文は`update`用のSPARQLエンドポイントに`curl`コマンドで`POST`する。

- curlでINSERT

```Powershell
curl "-X" "POST" `
  "-H" "Content-Type: application/sparql-update" `
  "--data" @"
PREFIX sh: <http://www.w3.org/ns/shacl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

INSERT DATA {
    <urn:prefix-declarations> a sh:PrefixDeclaration ;
        sh:declare [
            sh:prefix "ac" ;
            sh:namespace "http://example.org/attribute-concepts#"^^xsd:anyURI ;
        ] .
}
"@ `
  "http://localhost:3030/northwind/update"

```

エラーが出なければ、今度はWebUIで`SELECT`を実行し結果を確認する

- FusekiのWebUIでSELECT

```SPARQL
PREFIX sh: <http://www.w3.org/ns/shacl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?prefix ?namespace WHERE {
    ?decl sh:declare ?d .
    ?d sh:prefix ?prefix ;
       sh:namespace ?namespace .
}
```
- WebUIのレスポンス (CSV)

| prefix | namespace|
|---|---|
| ac | http://example.org/attribute-concepts#|

素晴らしい。`sh:PrefixDeclaration`が効いたのを確認できたので、ここにさらにラベルとコメントをつけ足す。ラベルやコメントの内容はとりあえずGenAIに適当に書いてもらうが、本来は人間の役割ではある。

- `label`と`comment`を追加

```powershell
curl "-X" "POST" `
  "-H" "Content-Type: application/sparql-update" `
  "--data" @"
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX ac: <http://example.org/attribute-concepts#>

INSERT DATA {
  ac:
    rdf:type owl:Ontology ;
    rdfs:label "属性概念スキーマ (Attribute Concepts)"@ja ;
    rdfs:comment "価格、数量、日付などの、エンティティの属性に関する汎用的な概念を定義するボキャブラリーです。GraphRAGにおける主要な属性語彙として利用されます。"@ja .
}
"@ `
  "http://localhost:3030/northwind/update"

```
結果をブラウザでも見てみよう。

- WebUIでSELECT

```SPARQL
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX ac: <http://example.org/attribute-concepts#>

SELECT ?predicate ?object
WHERE {
    ac: ?predicate ?object .
}
```
- WebUI SELECTのレスポンス (CSV)

|predicate | object |
|---|---|
| http://www.w3.org/1999/02/22-rdf-syntax-ns#type | http://www.w3.org/2002/07/owl#Ontology|
| http://www.w3.org/2000/01/rdf-schema#label | 属性概念スキーマ (Attribute Concepts) |
| http://www.w3.org/2000/01/rdf-schema#comment | 価格、数量、日付などの、エンティティの属性に関する汎用的な概念を定義するボキャブラリーです。GraphRAGにおける主要な属性語彙として利用されます。|

プレフィクス`ac:`を共通の「主語」とする3つの「述語」「目的語」が示されている。最初の行は「これはオントロジーです」、次が「こんなラベルがついてます」、最後が「こんな注釈がついてます」といったものだ。この辺は次回改めて取り上げる。

残りの分も同様にFusekiに`INSERT`してしまい、結果を確認する (クエリ文は最初の`ac:`の確認で使ったものと同様だが、プレフィクスを使って少しシンプルになっている)

- SHACLによるプレフィクスと名前空間のペアのクエリ

```SPARQL
PREFIX sh: <http://www.w3.org/ns/shacl#>
SELECT ?prefix ?namespace
WHERE {
  <urn:prefix-declarations> sh:declare ?declaration .
  ?declaration sh:prefix ?prefix ;
  sh:namespace ?namespace .
  }
```
- 上のクエリ結果 (csv)。`owl`や`rdf`はない (LLMにとっては既知なので要らない)

| prefix | namespace |
|---|---|
| ac | http://example.org/attribute-concepts# |
| ac-cat | http://example.org/attribute-concepts/category/ |
| arc | http://example.org/architypes# |
| ent | http://example.org/entity-concepts# |
| meta | http://example.org/meta# |
| nw | http://example.org/northwind# |

- 名前空間のラベルとコメントのクエリ

```SPARQL
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
SELECT ?ontology ?label ?comment
WHERE {
  ?ontology rdf:type owl:Ontology ;
  rdfs:label ?label ;
  rdfs:comment ?comment .
  }
ORDER BY ?ontology
```
- 上のクエリ結果 (csv)

| ontology | label | comment |
|---|---|---|
| http://example.org/architypes | アーキタイプスキーマ (Architypes) | エンティティやリレーションシップの基本的な構造やパターン(アーキタイプ)を定義するためのボキャブラリーです。|
| http://example.org/attribute-concepts# | 属性概念スキーマ (Attribute Concepts)| 価格、数量、日付などの、エンティティの属性に関する汎用的な概念を定義するボキャブラリーです。GraphRAGにおける主要な属性語彙として利用されます。|
| http://example.org/attribute-concepts/category/ | 属性概念カテゴリのスキーマ (Attribute Concept Categories) | 属性概念( c:)を分類するためのカテゴリ(例: テキスト、識別子)を定義するボキャブラリーです。|
| http://example.org/entity-concepts# | エンティティ概念スキーマ (Entity Concepts) | Northwindデータセット内の主要なエンティティ(例: 顧客、製品、注文)に関する概念を定義するボキャブラリーです。|
| http://example.org/meta# | メタスキーマ (Meta)  | オントロジー自体を記述するためのメタデータや、他のスキーマを横断するような概念を定義するボキャブラリーです。|
| http://example.org/northwind# | Northwind固有スキーマ (Northwind) | Northwindデータセットに固有のインスタンス(例: 特定の顧客や製品)を定義するために使用されるボキャブラリーです。|

このOntology Header込みの状態で`CONSTRUCT`句で[turtle形式のファイル](https://github.com/Yoshiyuki-iasa/northwind/blob/main/northwind_ontology_pt11.rdf)をとっておく。これもProtégéで問題なく開ける。"Active Ontology"タブの画面左側をよく見ると"Ontology Header"と、ここにもちゃんと書いてあるではないか。そしてそこには先ほど`INSERT`した名前空間がズラリと並んでいる。つまりProtégéで読むのはもちろん、Ontology Headerを書くことも出来る、というわけだ。標準化のチカラ、まさに恐るべし、である。

ここで使用したSPARQLクエリやSHACLの文法は初心者には意味不明で、その内容の理解には追々取り組むとして、ともあれこれで 「完全自立メタデータ」 の完成である。SPARQLエンドポイントと、オントロジー・ヘッダーが付いていることだけをLLMに伝えれば、あとは勝手に中身を読み解いてくれる (筈) で、LLMそのものの再訓練に掛けるカネや時間を大幅にカットできる。

W3Cのおかげで可搬性も相互運用性も心配無用で、個人的にはまだ良く解ってないがMCPとかAIエージェントとかを含むアーキテクチャを考えるうえでも、柔軟さなどの優位性が得られるのではないか。またこれは必然的に相当高いセキュリティ要件を伴うアーティファクトとなるが、プラットフォームに縛られず必要なデータ保護の手立てを自由に打つことが出来るだろう。

ただ、今触っているこのデータセットの中身はNorthwindのRDBスキーマを基にしているとはいえ、今のところGeminiに適当に書かせたもので、ラベルやコメントも部分的に付けてあるに過ぎない。LLMはオントロジーの「書き方」は知っているが、企業・組織やビジネスに固有の概念は人間が言語化するしかない (営業機密のようなことをLLMが最初から知っていたら逆に怖い)。つまり 「本当の話はまだ」 なのである。

今のトリプルカウントは767になった。次回はSPARQLクエリでその中身を解き明かしていこう。

([Part12につづく](part12.md))
