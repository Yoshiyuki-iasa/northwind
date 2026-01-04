## ビジネスメタをオントロジーで書く - Part 14

### 最初に何をクエリするか？

RDFで書かれたメタデータの探索を実装、ともなれば、LangChainやLlamaIndexなどの"text2sparql"関連技術で開発することになるのだろうが、ここでは一応基本も押さえておくべく、「SPARQL手打ち」も試しておくことにする。

ドキュメントなどが存在しない未知のRDFデータセットを探索する際は、まずクラスやトリプルのサンプルを取得するといった手順が存在するようだが、このNorthwindデータセットには[Part 11](part11.md)で付与したオントロジー・ヘッダーが含まれている。そのクエリ例はPart11を参照するとして、その結果W3Cなどの標準以外の名前空間が (`owl:ontology`として) 以下のように書かれていることが分かったとしよう。

|Prefix|(実際の)意味|
|---|---|
|`arc:`|"architype"の略。SQLテーブルの類別|
|`ent:`|"entities"の略。SQLテーブル。|
|`ac-cat:`|"attribute concept categoris"の略。SQLテーブルの属性の類別|
|`ac:`|"attribute concepts"の略。SQLテーブルの属性。|
|`meta:`|アノテーション・プロパティとの肝炎付けに用いられる述語他を含む名前空間|
|`nw:`|"northwind"。アノテーションの対象となるRDBスキーマ|

Northwindのオントロジー・メタに実際に書いてあるコメントとは異なるが、元々はこういう意図である。厳密にはこれを書いたGenAIの意図で、やはりこの辺りは人間が意図を込める必要がある、というのは前回指摘した。

とはいえ、せっかく細かく分かれているので、まずはそれぞれどんな語彙があるのかをクエリしてみよう。例えば上のコントロジー・ヘッダーの記述を手掛かりに、まずは`arc:`でアタリを付けるべく語彙を拾いたい際のSPARQLは以下である (SPARQL手打ち、とは言ったものの、「餅は餅屋」で実際に書くのはGenAIである。ちなみにクエリの前には名前空間が必要だがここでは省略する)。

```SPARQL
SELECT DISTINCT ?resource ?label ?comment
WHERE {
  ?resource ?p ?o .
  FILTER (STRSTARTS(STR(?resource), STR(arc:)))
  OPTIONAL {
    ?resource rdfs:label ?label .
  }
  OPTIONAL {
    ?resource rdfs:comment ?comment .
  }
}
```
すると以下のような語彙が存在することがわかる (URIはプレフィクスに置換してある)

|語彙|ラベル|コメント|
|---|---|---|
|`arc:`|"アーキタイプスキーマ (Architypes)"@ja|"エンティティやリレーションシップの基本的な構造やパターン(アーキタイプ)を定義するためのボキャブラリーです。"@ja|
|`arc:architype`|Architype root||
|`arc:Resource`|Resource|主語または目的語となる名詞の集合。顧客、従業員、商品、勘定科目など。|
|`arc:Event`|Event|時間と共に発生する状態変化の記録。受注や出荷。|
|`arc:Classification`|Classification|MECEな概念の集合。国、都道府県、ステータスなど。|
|`arc:Rule`|Rule|振る舞いを規定するための複数の概念のあらかじめ定められた組み合わせ。値引きルール、返品ルールなど。
|`arc:Summary`|Summary|別のデータセットから導出された集計結果。総勘定元帳など。|
|`arc:Association`|Association|複数のエンティティ間の関係についての事実。営業テリトリーと営業員など。|

見てみると例えば行目のは要るのか、とか、言語タグの有り無し、とかいろいろ気になることはある一方、リターンが少数で分かりやすく、細切れもこれはこれでアリかも知れない、とも思えてくる。

### "Connecting the dots"

それはさておき、これらの語彙がRDF上でどのように使われているかといえば、目的語であろう。ナントカのナントカは(例えば)`arc:Resource`(目的語)である、というトリプルが存在するはずだ (ちなみに上のリストは「`arc:Resource`(主語)のラベルは何でコメントは何」を示している)。それをクエリしてみよう。

```SPARQL
SELECT DISTINCT ?subject ?label
WHERE {
  ?subject ?p arc:Resource .
  OPTIONAL {
    ?subject rdfs:label ?label .
  }
}
```

するとこういう結果が得られる。

|subject|label|
|---|---|
|`ent:Employees`|Employees Concept|
|`ent:Customers`|Customers Concept|
|`ent:Shippers`|Shippers Concept|
|`ent:Suppliers`|Suppliers Concept|
|`ent:Products`|Products Concept|

あとはSPARQLの条件式の部分を変えて同じことを繰り返すのみである。例えば`ent:Employees`の主語を検索すると`nw:Employees`に行きつく。ここで`DESCIBE nw:Employees`と実行すると、そのリソースを主語とするトリプルがturtle形式で表示される (実際には全名前空間も表示されるがここでは省略)。

```SPARQL
nw:Employees  rdf:type         owl:Class;
        rdfs:label             "Employees";
        meta:hasEntityConcept  ent:Employees .
```

もちろん普通の手順でクラスから始める手もある。FusekiのブラウザUIならボタン一発でクラス一覧のクエリを呼び出せる。何なら名前空間から始めるより、こちらの方が楽かも知れない。クラス一覧のクエリにはカスタム名前空間は要らないし、名前空間の一覧は`DESCRIBE`で取得できる。そして上の例のように、何やら独自の名前空間と語彙がある、というところまでわかる。

あるいはタイトルやコメントは全てリテラル値として文字列を持つので、カスタム名前空間などが分からなくても、含まれているであろうビジネス語彙で検索してみる、という手もある。

```SPARQL
SELECT ?subject ?predicate ?object
WHERE {
  ?subject ?predicate ?object .
  FILTER (isLiteral(?object))
  FILTER (CONTAINS(STR(?object), "区分"))
}
```

結果はこうである (boldは筆者)

|subject|predicate|object|
|---|---|---|
|`ac:Enum`|`rdfs:comment`|分類テーブルがない (リテラル) **区分**値|
|`ac:Flag`|`rdfs:comment`|**区分**値または列挙子のうち、特に真偽二値のみのもの|
|`ac:Name`|`rdfs:comment`|識別子または**区分**値の名称|

…ということで、全く見も知らずのオントロジーを、紙とペンで"connecting the dots"よろしくちなちま探索するのは結構大変そうである。やはりNeo4j Desktopのようなグラフィカルな表現が出来るに越したことはない。Fusekiのようにお手軽な何らかの手段が見つかると良いのだが。

(Part15に続く)