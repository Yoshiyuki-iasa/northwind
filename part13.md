## ビジネスメタをオントロジーで書く - Part 13

### 「プロパティ」「ドメイン」「レンジ」

RDBになじみがあれば、クラスといえばエンティティとかテーブルのことだよね、と大体の察しは付く。プロパティというのは属性であろう。しかしそれらはRDFトリプルではどのように表現されるのだろうか。なにやらこの辺りから認知負荷がぐっと上がりそうな気配である。

このシリーズの最初の方でMermaidで書いた[ER図](northwind_schema.md)なり[クラス図](northwind_ontology_diagram.md)なりでデータモデルを見て、属性の少ないテーブルを見繕い、そのRDFでの書きっぷりを見ることにしよう。"Shippers"など良さそうである。turtle形式ではこう書いてある。

```turtle
nw:Shippers  rdf:type   owl:Class;
```

たったこれだけ、である (ラベルなどはここでは関係ないので除く)。このクラスがどんな属性を持っているか、といったことはここには何も書かれていない。

では属性はどこでどのように書かれているのか。ER図ではPKとして"ShipperID"、あとは会社名と電話番号である。

```turtle
nw:ShipperID_  rdf:type           owl:DatatypeProperty;
        rdfs:domain               nw:Shippers;
        rdfs:range                xsd:integer;

nw:CompanyName  rdf:type          owl:DatatypeProperty;
        rdfs:domain               nw:Suppliers , nw:Shippers , nw:Customers;
        rdfs:range                xsd:string;

nw:Phone  rdf:type                owl:DatatypeProperty;
        rdfs:domain               nw:Suppliers , nw:Shippers , nw:Customers;
        rdfs:range                xsd:string;
```

どれもタイプは`owl:DatatypeProperty`で、`rdfs:domain`と`rdfs:range`という2つの述語を伴う。「ドメイン」の目的語はクラスのようだ。`nw:CompanyName`や`nw:Phone`のごとくクラスが複数、ということは、ここはSQLテーブルと違い注意が必要だが、複数のテーブルにある同一の属性、ということになる。そして「レンジ」は整数とか文字列であることがわかる。

また、データモデルでは"Shippers"テーブルと"Orders"テーブル間にリレーションが引かれている。これはRDFでどう表現されているのか。

```turtle
nw:ShipVia  rdf:type              owl:ObjectProperty;
        rdfs:domain               nw:Orders;
        rdfs:range                nw:Shippers;
```

こちらの`rdf:type`は`owl:ObjectProperty`である。「ドメイン」は"Shippers"の例と同じく、その属性をもつクラスという意味だろう。「レンジ」は「整数」や「文字列」ではなく、マスタとして参照しているクラス`nw:Shippers`の名前が入っている。

このように書くと、SQLテーブルを見慣れた身からすると随分とややこしく聞こえる。しかしこうすると少しわかりやすくはならないだろうか。

|  |列<br>述語 (Predicate)<br>プロパティ |
|---|---|
| 行<br>主語 (Subject)<br>クラス |値<br>目的語 (Object)<br>クラスまたはリテラル |

「言われてみればその通り」としか言いようがない。RDFトリプルもSQLテーブルも表現のカタチが違うだけで、言っていることは「行 (クラス) の列 (プロパティ) は値 (クラスorリテラル) である」と、全く同じである。もうわかったも同然だが、これに`owl:DatatypeProperty`と`owl:ObjectProperty`を一応当てはめてみるとこうなる。

|  | 列<br>述語 (Predicate)<br>"Datatype"プロパティ|列<br>述語 (Predicate)<br>"Object"プロパティ  |
|---|---|---|
| 行<br>主語 (Subject)<br>クラス|値<br>目的語 (Object)<br>リテラル | 値<br>目的語 (Object)<br>クラス |

となると、`nw:ShipperID_`のトリプル3行ならこうである。

|  |`rdf:type`|`rdfs:domain`|`rdfs:range`
|---|---|---|---|
|`nw:ShipperID_`| `owl:DatatypeProperty` | `nw:Shippers` |`xsd:integer` |

参照先テーブル ("Orders") だとこうなる。

|  |`rdf:type`|`rdfs:domain`|`rdfs:range`
|---|---|---|---|
|`nw:ShipVia`| `owl:ObjectProperty` | `nw:Orders` |`nw:Shippers` |

今更だが、これこそがRDFを理解する上で最初にすべき説明であろう。ここでのテーマはメタデータなのでインスタンスは登場しないが、それらも同様にこんな感じでRDFに変換されるであろうことは想像に難くない。またSQLテーブルにカラムを足すより、RDFデータセットにトリプルを足す方がずっと簡単なことも明らかだ。特にメタデータの場合は、その対象となるデータ要素の種類によって必要なメタデータは異なるだろうしし、そもそも事前に全部決めるのは無理である。

そういうメタデータを書くのに便利なプロパティがOWLにはあって、NorthwindのRDFでもビジネスメタで使われている。`owl:AnnotationProperty`である。

### アノテーション・プロパティ

オントロジーというと日本では「学術用語」と取られがちで、その学術的関心のひとつに「記号論的推論」というものがり、毎年コンペも開かれているようである。しかしアノテーション・プロパティはこの推論の役に立たず、従って使い道に関する参照情報も乏しい。しかし逆に言えば、推論に影響しないので自由度が高い、ともとれる。ビジネスメタをはじめ、様々なメタデータをオントロジーに持たせるにはうってつけであろう。GenAIにとってもビジネスメタを読解する上で有用な情報と出来る筈だ。もちろん推論に関わる部分のオントロジーに論理的破綻がなければ、だが。

NorthwindのRDFデータセットには、すでに[Part4](part4.md)で追加したアノテーション・プロパティが含まれている。それぞれのテーブル (クラス) や属性 (データタイプ or オブジェクト・プロパティ) に直接説明コメントを書く代わりの手段として用いられている。例えば「カスタマー」テーブルについて、これらがturtleでは実際にはどのように書かれているかを見てみよう。まずは`ent:Customers`なるアノテーション・プロパティである。

```turtle
ent:Customers  rdf:type     owl:AnnotationProperty;
        rdfs:label          "Customers Concept";
        rdfs:subPropertyOf  arc:Resource;
        meta:isPII          true .
```

3行目は「リソーステーブルに分類される」、4行目は「PIIに該当する」という意味である。本来の趣旨でいえば、ここにカスタマーという概念についての自然言語による説明が`rdfs:comment`として加わることになる。

一方カスタマーテーブルは`nw:Customers`というクラスである。そこでは上記のアノテーション・プロパティ`ent:Customers`との関係が、`meta:hasEntityConcept`という述語で以下のように表現されている。

```turtle
nw:Customers  rdf:type         owl:Class;
        rdfs:label             "Customers";
        meta:hasEntityConcept  ent:Customers .
```

属性も見てみよう。例として`ac:CustomerID`というアノテーション・プロパティの内容を以下に示す。3行目には属性としての分類 (ID、つまりリソースのキー) が記されている。また`rdfs:comment`が (まだ) 付いていないのもテーブルと同様である。

```turtle
ac:CustomerID  rdf:type     owl:AnnotationProperty;
        rdfs:label          "CustomerID Concept";
        rdfs:subPropertyOf  ac:ID .
```
このアノテーション・プロパティを、以下のデータタイプとオブジェクトのふたつのプロパティが共有している。説明はアノテーション・プロパティ側に一回書けば、それぞれに2度書くという手間が省けるし、定義の不整合も排除できる、という発想である。属性の関連付けに使われている述語は`meta:hasAttributeConcept`である。

```turtle
nw:CustomerID_  rdf:type          owl:DatatypeProperty;
        rdfs:label                "CustomerID_";
        rdfs:domain               nw:Customers;
        rdfs:range                xsd:string;
        meta:hasAttributeConcept  ac:CustomerID .

nw:CustomerID  rdf:type           owl:ObjectProperty;
        rdfs:label                "CustomerID";
        rdfs:domain               nw:CustomerCustomerDemo , nw:Orders;
        rdfs:range                nw:Customers;
        meta:hasAttributeConcept  ac:CustomerID .
```

マスタ、属性どちらのアノテーション・プロパティにも述語`rdfs:subPropertyOf`が存在するが、その目的語はアノテーション・プロパティである。クラスでは以下のように、上から`arc:architype -> arc:Resource -> ent:Customers`という階層、すなわちタクソノミーを辿る。

```turtle
arc:architype  rdf:type  owl:AnnotationProperty;
        rdfs:label  "Architype root" .

arc:Resource  rdf:type      owl:AnnotationProperty;
        rdfs:comment        "主語または目的語となる名詞の集合。顧客、従業員、商品、勘定科目など。";
        rdfs:label          "Resource";
        rdfs:subPropertyOf  arc:architype .
```

属性も同様、`ac:semanticType -> ac:ID -> ac:CustomerID`という階層が存在する。

```turtle
ac:semanticType  rdf:type  owl:AnnotationProperty;
        rdfs:label  "Semantic Type root" .

ac:ID   rdf:type            owl:AnnotationProperty;
        rdfs:comment        "リソースのキー";
        rdfs:label          "Identifier";
        rdfs:subPropertyOf  ac:semanticType .

```

なお、属性の関連付けに使われている述語は共に、オブジェクト・プロパティである。

```turtle
meta:hasEntityConcept
        rdf:type    owl:ObjectProperty;
        rdfs:label  "has entity concept" .

meta:hasAttributeConcept
        rdf:type    owl:ObjectProperty;
        rdfs:label  "has attribute concept" .
```

---

しかし、RDFが少し解ってから見直すと、GenAIに書かせたオントロジーには色々とアラがありそうである。まず、名前空間が細切れに過ぎる。Northwind程度の大きさなら、アノテーション・プロパティの名前空間はひとつで済むのではないか。現実世界では事業のすべての領域に精通しているヒトなどいないので、ドメイン・エキスパート毎に分割するにしても、である。また、オブジェクト・プロパティの名前は`meta:hasEntityConcept`のように動詞句であるべきだろう。`nw:CustomerID`ではなく、`nw:hasCustomer`である。そうすれば `nw:Orders -> nw:hasCustomer -> nw:Customers` という、如何にもオントロジーらしい表現になる。

今回はturtleファイル上でトリプルの例を観察したわけだが、次回は複数のリソースなりトリプルなりをSPARQLで抽出してみよう。RDFのキホンが大体解ったところで、次はSPARQLのキホンである。

(Part14につづく)
