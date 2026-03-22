## ビジネスメタをオントロジーで書く - Part 27

### "northwind"をgistで書きなおし

前回の"university"はオリジナルのオントロジーがあったのでそれを尊重してgistにマッピングしたが、"northwind"はDDL→SHACL→RDFの変換をGenAIにやらせたもので、今回は「gistにマッピング」というよりは「gistベースで書きなおし」である。元ネタは[RDFに変換してすぐ](northwind_ontology_pt2.ttl)のものである。その後いろいろヘンなものを足したりしたが、それらは恐らく今後再度開かれることはないだろう。

まずはクラスである。全部で13個あるが、これはまあgistのサブクラスにそのままこんな感じに移せばよいだろう。

|(label@ja)|nw:|rdf:subClassOf|
|---|---|---|
|商品分類|nw:Categories|gist:ProductCategory|
|顧客分類割当|nw:CustomerCustomerDemo|gist:Assignment |
|顧客分類|nw:CustomerDemographics|gist:Category|
|顧客|nw:Customers|gist:Organization|
|従業員商圏割当|nw:EmployeeTerritories|gist:Assignment|
|従業員|nw:Employees|gist:Person|
|注文明細|nw:OrderDetails|gist:Commitment|
|注文|nw:Orders|gist:Agreement|
|商品|nw:Products|gist:ProductSpecification|
|地域|nw:Region|gist:GeoRegion|
|配送業者|nw:Shippers|gist:Organization|
|商品仕入先|nw:Suppliers|gist:Organization|
|商圏|nw:Territories|gist:GeoRegion|

これらのエンティティから察するに、northwindのビジネスオペレーションは「外回り物販営業」で、初出がMS Access95というnorthwindの長い歴史を感じさせる。

### gistでのクラス参照の表現

#### 「商品」と「商品分類」
さて、改めて[northwindのER図](https://github.com/Yoshiyuki-iasa/northwind/blob/main/northwind_schema.md)を眺めてみる。まずは「商品」と「商品分類」をどう紐付けるか、である。現状では`nw:CategoryID`なる変な名前 (本来動詞表現であるべき) のObject Propertyが存在する。これを`gist:isCategorizedBy`のSubPropertyにぶら下げれば良いのか？

しかしここでそもそも何故gistにはこんなに沢山のObject Propertyが有るのかを疑問に思わなければならない。これは「これだけ準備したのだから足りるでしょ」というgistの作成者の意図なのだ。しかし、だからといって`gist:isCategorizedBy`に直接「商品」Domainと「商品分類」Rangeを書くわけにも行かない。さてどうしたものか。

色々調べると、gistではSubclassOf制約を使いなさい、ということになっている。書き方自体は[前回の「院生」の絞り込み](part26.md#独自語彙を残す)と同一である。

```turtle
nw:Products rdf:type owl:Class ;
            rdfs:subClassOf gist:ProductSpecification ,
                            [ rdf:type owl:Restriction ;
                              owl:onProperty gist:isCategorizedBy ;
                              owl:someValuesFrom nw:Categories
                            ] .
```

`gist:isCategorizedBy`はここで使う為にあるのであり、適当な名前のObject Propertyを雑にぶら下げる為のものではない、ということだ。Protégéではこの入力はクラスタブのDescriptionペインで行う。Class Expression Editorで`isCategorizedBy some Categories`と直接書いても良いし、Object Restriction CreatorでPropertyとResourceを選択しても良い。

#### 「商圏」と「地域」
同様な関係がもうひとつ、`nw:Territories`と`nw:Region`がある。但しこちらでは`nw:Region`は`gist:Category`ではなく`nw:Territories`と並んで`gist:GeoRegion`のサブクラスになっている。この場合は`gist:isCategorizedBy`のかわりに`gist:isGeoContainedIn`プロパティを使う。Class expressionでいえば`isGeoContainedIn some Region`となる。

#### 「商品」と「仕入先」
商品と仕入先の紐付けはカテゴリ分けではないので、この場合は`gist:comesFromAgent`という語彙を使う。`comesFromAgent some Suppliers`である。

#### 「従業員」の自己参照
従業員には"reports to"という自己参照型の属性を持つ。これには`gist:isGovernedBy`が使えるだろう。語彙を増やす要否、という評価をその都度下すのがgistの使い方だ。ここは前回の`isSupervizedBy`のような微妙なニュアンスではなく、要は上下関係が示せればよいので、gist語彙をそのまま使おう。ソースではこのようになる。

```turtle
nw:Employees rdf:type owl:Class ;
             rdfs:subClassOf gist:Person ,
                             [ rdf:type owl:Restriction ;
                               owl:onProperty gist:isGovernedBy ;
                               owl:allValuesFrom nw:Employees
                             ] .
```

ここで他と異なるところが一点、`owl:allValuesFrom`である。`owl:someValuesFrom`はSQLではnot null制約で、ここでは過剰 (社長はどうするか、など) なのでallを使う。表現が感覚と反する感じもするが、この辺りはowlのドキュメントを読んで慣れて納得するしかない。

#### 「顧客」/ 「配送業者」と「注文」 

これらは全て「注文」のsubClassOf制約となる。他にも書き方はいくつか有り得る (意味的にもう少し特化した語彙をgistのサブプロパティとして追加する、次項の中間クラスを使う、など) が、まずはgist標準語彙+subClassOf制約で済ませることにする。

「顧客」：`gist:hasParty`を使う。

「配送業者」：`gist:comesFromAgent`を使う。`comesFromAgent some Shippers`である。

「従業員」：`gist:hasParticipant`である。`hasParticipant some Employees`となる。

### 中間クラスのマッピング

モデルには顧客と顧客分類、従業員と商圏とのn:n関係を表す中間クラス (`gist:Assignment`) が存在する。gistではsubClassOf制約をここにふたつ書いて両端のクラスへの関係を表現するようだ。Class expressionで書くと、顧客分類割当では、

顧客分類割当 (`nw:CustomerCustomerDemo`) については

`isAssignmentOf some Customers`
`isAssignmentTo some CustomerDemographics`

従業員商圏割当 (`nw:EmployeeTerritories`) では、

`isAssignmentOf some Employees`
`isAssignmentTo some Territories`

となる。"of"と"to"のどちらをどちらにすべきか混乱するかもしれない (`gist:Assignment`の2番目の`skos:scopeNote`にもわざわざ書いてある) が、リソース側がOf、区分値側がTo、と覚えるしかない。

注文明細 (`nw:OrderDetails`) も「中間クラス」である。

`isDirectPartOf some Orders`
`isAbout some Products`

### 可視化ができない

こうして、元々存在していたプロパティもとりあえず全部消して、クラスとsubClassOf制約によるクラス間結合だけにしたファイルが[こちら](northwind_ontology_pt27.ttl)である。

これをFusekiに読ませてGraph Explorerで可視化してみたが、複数のブランクノードの区別が付かないようで上手くいかない。

subClassOf制約をやめて普通のdomain / rangeで関係を書けばキレイに表現できるはずだが、今回は可視化の方はまたの機会としよう。一応GenAIにも見てもらったが、孤立ノードなどの書き損じはなさそうである。

次回はこのモデルにData Propertyを付け加えていく。さてどのようなgist語彙が登場するだろうか？

([Part28につづく](part28.md))
