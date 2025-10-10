## ビジネスメタをオントロジーで書く - Part 6

### オントロジー・ファイルをNeo4jに読み込む

ポピュラーなRDFストアというと、Neo4jやAmazon Neptune、Apache Jena Fuseki、Blazegraphなどの名前が挙がるようである。ここまですべて無償のサービスやツールで進めてきたし、以前少し触ったことがあるNeo4jをまず試すことにする。

オントロジー・ファイルをグラフデータベースに読み込む前に、`rdfs:label`で表示名称を個々の概念に与えておく。これが無いとNeo4jはGraphビューでノードの名前を適当に付けてしまう。これもGemini CLIにお願いすればあっという間である。オントロジーファイルをNeo4jに読ませる前提であれば、SHACLから「素の」オントロジーを生成する時点で付け加えておくべき指示ではある。

[ラベル付きのオントロジー・ファイル](https://github.com/Yoshiyuki-iasa/northwind/blob/main/northwind_ontology_pt6.ttl)が出来たところで、まずはPCにNeo4j Desktop 2を準備し空のDBインスタンスを立てる。Neo4jでRDFを扱うには、neosemantics (n10s) というプラグインが必要で、それのjarファイルが[neo4j-labsのGitHubリポジトリ](https://github.com/neo4j-labs/neosemantics/releases)にあるので、Neo4j Desktop 2のインスタンス一覧表示にあるPath: ボタンを押した先の`plugins`フォルダに入れておく。

DBを起動したら、とりあえずクエリ画面をつないで、`MATCH (n) OPTIONAL MATCH (n)-[r]-() RETURN n, r;`などと打ってみて、エラーが出たり、接続先DBを間違えていたりしてないことを確認する。問題なければ続けて以下の操作を行う。

1. 一意制約の設定：`CREATE CONSTRAINT n10s_unique_uri FOR (r:Resource) REQUIRE r.uri IS UNIQUE;`

2. n10sの初期化：`CALL n10s.graphconfig.init();`

3. プレフィクスの登録：本来はオントロジー・ファイルの最初に書いてあるのをそのままインポートしてくれれば良いのだが、n10sが変なプレフィクスを勝手に作ったりするので、インポートの前にオントロジー・ファイルと整合したプレフィクスを手動で登録しておく。
```
CALL n10s.nsprefixes.add("owl", "http://www.w3.org/2002/07/owl#");
CALL n10s.nsprefixes.add("rdf", "http://www.w3.org/1999/02/22-rdf-syntax-ns#");
CALL n10s.nsprefixes.add("rdfs", "http://www.w3.org/2000/01/rdf-schema#");
CALL n10s.nsprefixes.add("xsd", "http://www.w3.org/2001/XMLSchema#");
CALL n10s.nsprefixes.add("nw", "http://example.org/northwind#");
CALL n10s.nsprefixes.add("arc", "http://example.org/architypes#");
CALL n10s.nsprefixes.add("ac", "http://example.org/attribute-concepts#");
CALL n10s.nsprefixes.add("ent", "http://example.org/entity-concepts#");
```
`CALL n10s.nsprefixes.list();`で登録済みのプレフィックスを確認できる。

上記の手順が終わったら、`CALL n10s.rdf.import.fetch("file://{PATH_TO_FILE}", "Turtle");`でオントロジー・ファイルのインポートを実行する。上手くいけば下記のような表示が返ってくる。
```
╒═════════════════╤═════════════╤═════════════╤════════════════════════════════════════════════════════════════════════════════════════════════════╤═════════╤═════════════════════╕
│terminationStatus│triplesLoaded│triplesParsed│namespaces                                                                                          │extraInfo│callParams           │
╞═════════════════╪═════════════╪═════════════╪════════════════════════════════════════════════════════════════════════════════════════════════════╪═════════╪═════════════════════╡
│"OK"             │694          │694          │{                                                                                                   │""       │{                    │
│                 │             │             │  rdfs: "http://www.w3.org/2000/01/rdf-schema#",                                                    │         │  singleTx: false    │
│                 │             │             │  arc: "http://example.org/architypes#",                                                            │         │}                    │
│                 │             │             │  ac: "http://example.org/attribute-concepts#",                                                     │         │                     │
│                 │             │             │  owl: "http://www.w3.org/2002/07/owl#",                                                            │         │                     │
│                 │             │             │  xsd: "http://www.w3.org/2001/XMLSchema#",                                                         │         │                     │
│                 │             │             │  rdf: "http://www.w3.org/1999/02/22-rdf-syntax-ns#",                                               │         │                     │
│                 │             │             │  ent: "http://example.org/entity-concepts#",                                                       │         │                     │
│                 │             │             │  nw: "http://example.org/northwind#"                                                               │         │                     │
│                 │             │             │}                                                                                                   │         │                     │
└─────────────────┴─────────────┴─────────────┴────────────────────────────────────────────────────────────────────────────────────────────────────┴─────────┴─────────────────────┘
```
インポートが出来たら、最初にDB確認のために打った`MATCH (n) OPTIONAL MATCH (n)-[r]-() RETURN n, r;` (矢印キーで履歴で戻れる) を再び実行してみれば、例の「丸と線の塊」が表示される。

しかし拡大してみるとノードの名前が意味不明で、これではどの丸が何なのかが分からない。ここでオントロジー・ファイル上で振った`rdfs:label`の値を使って、`n.name`というNeo4jのノードプロパティに値をセットせよ、というコマンドを流す。
```
MATCH (n)
WHERE n.rdfs__label IS NOT NULL
SET n.name = n.rdfs__label;
``` 
### つながらないグラフデータ
これでどれが何かが分かるようになった。独立した大中小の3つの塊が見える。一番大きいのはいわゆる「素の」DBスキーマの部分である。プレフィクスでいえば`nw:`、つまりNorthwindである。2番目に大きいのは`ac:`のついた「属性」の、小さいのは`ent:`のついた「テーブル」の、それぞれアノテーション・プロパティの一群である。

これは 「繋がってナンボ」 のグラフデータとしては宜しくない。と同時に、まあ予想がつく結果でもある。

アノテーション・プロパティをテーブルや属性に割り当てるのに、「割当先はこのアノテーション・プロパティについてブール値で真です」、と書いたのだが、Neo4jはそれを 「線」 とは認めてくれないのだ。

やはりここは線で繋げたい。Neo4jで線を引けば良い、という話もあろうが、やはりオントロジー・ファイルから見直すべきだろう。さてどんな手が打てるだろうか。

(Part7に続く)
