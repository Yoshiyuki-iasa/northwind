## ビジネスメタをオントロジーで書く - Part 8

### 区分値 (Classifier) のメタの書き方

ビジネスメタで欠かせないが、前回まででまだ書けていないもののひとつが 「区分値の説明」 である。

区分値の意味を説明する際は、おのずとインスタンスの値とその意味とを書き並べることになる。テーブル定義の備考欄に 「"1"は何々、"2"は何々…」 と書かれているアレである。しかしここでそんなカッコ悪いことをやる訳にはいかない。オントロジーではどう記述すれば良いか。もちろん`rdfs:comment`を使って同様のことは出来るが、それではつまらない。

一方、オントロジーにおける一般的なプラクティスだと、class (テーブル) にindividual (実データ) をぶら下げて、それらにコメントを書く、ということになるが、メタデータのオントロジーという建前でいえば、データそのものであるindividualがオントロジーに混じるのは避けたい。

結論を言えば、区分値のアノテーション・プロパティにサブクラスをぶら下げる、ということになる。例えば、Northwindが食品ECだとして、飲料とか調味料とかの商品分類を持っているとする。この情報を`Categories`テーブルが保持していて、そこには`CoategoryID`, `CategoryName`といった属性がある。

サブクラスはどの属性にぶら下げるか、というと`CategoryID`である。ぶら下げるサブクラスは`1`とか`2`とかの値そのものではなく、`Beverages`や`Condiments`という概念である。そしてそれらは`1`とか`2`とかの 「カテゴリ値」 を持つ、という書き方になる。

コードで書くとこういうことである。

```turtle
# カテゴリ値のための名前空間 (プレフィクス) を追加
@prefix ac-cat: <http://example.org/attribute-concepts/category/> .

# 「インスタンスはIDの値を持つ」アノテーション・プロパティの追加
meta:hasValueId rdf:type owl:AnnotationProperty ;
    rdfs:label "has value id" ;

# 「カテゴリID」属性には「飲料」カテゴリがあり、その値は`1`
ac-cat:Beverages rdf:type owl:AnnotationProperty;
    rdfs:subPropertyOf ac:CategoryID;
    rdfs:label "Beverages";
    meta:hasValueId 1 .

# (以下同様にインスタンスを追記
```
こうなると、`CategoryName`の方も何か書きたくなるのが人情である。ただこれはDDLではないので、コメントを足す程度に留める。このコメントを解釈して`CREATE TABLE`をAIが書いてくれる日がいつか来るかも知れない。
```turtle
# 「カテゴリ」テーブルの「カテゴリ名」属性
ac:CategoryName rdf:type owl:AnnotationProperty;
    rdfs:subPropertyOf ac:Name;
    rdfs:label "CategoryName Concept";

# 英語のコメント
    rdfs:comment "The value of the corresponding data property (e.g., nw:CategoryName) is intended to match the rdfs:label of one of the conceptual sub-properties of ac:CategoryID. This links the instance data to the conceptual model for categories."@en ;

# 日本語のコメント
    rdfs:comment "対応するデータプロパティ（例: nw:CategoryName）が取る値は、ac:CategoryID の概念的なサブプロパティの rdfs:label と一致することが意図される。これにより、インスタンスデータとカテゴリの概念モデルが結びつく。"@ja .
```
図らずも文末の言語タグ (`@en`, `@ja`) の使い方のサンプルにもなった。

### 「ブール値で真」の真っ当な使い方

以前、DBスキーマのテーブルや属性へのアノテーション・プロパティの「付け方」が、Neo4jでリレーションとして扱われなかった、という話があった。その時使ったのが、「このアノテーション・プロパティについてブール値で真である」という書き方だが、もちろんその書き方が有効な場面もある。例えば「これはPIIです」といった、「修飾を付ける」ケースである。Neo4jにノードとして表示されることもないのはすでに証明済みである。

```turtle
# アノテーション・プロパティの追加
meta:isPII rdf:type owl:AnnotationProperty ;
    rdfs:label "is PII" ;
    rdfs:comment "個人情報（PII）であるかどうかを示すためのアノテーション" .

# 別のアノテーション・プロパティ (この場合はEmployeesクラス) の修飾
ent:Employees rdf:type owl:AnnotationProperty;
    rdfs:subPropertyOf arc:Resource;
    rdfs:label "Employees Concept";
    meta:isPII "true"^^xsd:boolean . # <-この行を追記
```
同様に、例えば区分値も個々のインスタンスレベルで修飾することもできる。テーブル定義書の備考欄に平文を書き連ねる従来の方法では出来ない芸当である。
```turtle
# 「軽減税率対象」というアノテーションプロパティ
meta:isReducedTaxRateTarget rdf:type owl:AnnotationProperty .

# 飲料は軽減税率対象
ac-cat:Beverages rdf:type owl:AnnotationProperty;
    rdfs:subPropertyOf ac:CategoryID;
    rdfs:label "Beverages";
    meta:hasValueId 1;
    meta:isReducedTaxRateTarget "true"^^xsd:boolean .

# 調味料は軽減税率対象外
ac-cat:Condiments rdf:type owl:AnnotationProperty;
    rdfs:subPropertyOf ac:CategoryID;
    rdfs:label "Condiments";
    meta:hasValueId 2;
    meta:isReducedTaxRateTarget "false"^^xsd:boolean .
```
これらのサンプルはすべて[このファイル](https://github.com/Yoshiyuki-iasa/northwind/blob/main/northwind_ontology_pt8.ttl)に含まれている。

さて、RDFファイルをNeo4jへインポートできることは分かった。しかしNeo4jはSPARQLが使えないし、本業のLPGの傍らRDFもやってます、という感じが否めない。SPARQLをお手軽に試す方法は無いものだろうか？

([Part9につづく](part9.md))
