## ビジネスメタをオントロジーで書く - Part 18

### 本当はどうやって書くのか？

これまで正解は良く解らないまま、とりあえず「DLL → SHACL → TTL」でGenAIにファイル変換させて出来たオントロジーでいろいろと試してきたわけだが、Graph Explorerもゲット出来たことだし、そろそろ「本当の正しい書き方」に向き合うことにしたい。

しかし現実は誠に厳しいもので、ネットで「SQLテーブルのRDFによる表現」のサンプルを検索しても一向に見つからない。ネット上であれだけ「メタデータはオントロジーで書くのが正義！」のような主張があふれているにも関わらず、である。

と、愚痴を垂れても始まらないので、まずは[R2RML (RDB to RDF Mapping Language)](https://www.w3.org/TR/r2rml/)なるW3C勧告にチャレンジである。GeAIの助けも借りつつ、少しづつ解き明かしていきたい。

### R2RMLを試す

まず、「顧客が注文」という単純なRDFがあるとする (`a`は`rdf:type`の省略形)。

```turtle
ex:Customer a owl:Class ;
    rdfs:label "顧客" .

ex:Order a owl:Class ;
    rdfs:label "注文" .

ex:placedBy a owl:ObjectProperty ;
    rdfs:label "注文者" ;
    rdfs:domain ex:Order ;
    rdfs:range ex:Customer .

```

Graph Explorerではこのようになる (白抜き数字は非表示ノードの数だがここでは無視である)。RDB的に言えば「注文者」は「注文」の属性で、「顧客」クラスのインスタンスである。

![p18fig1](/media/p18fig1.png)


さて、ここからR2RMLの出番である。まず"Triple Map"なるSubject - 主語を作る。

`<#OrderMapping> a rr:TriplesMap ;`

`<>`の部分は「相対URI」と呼ばれるRDFの文法である。フルで書くと例えば`http://localhost:3030/sql/data#OrderMapping`などとなる。`rr:`はR2RML語彙である。つまりこれは「Order Mappingという、R2RMLでいうところのTriple Mapのクラス」である。

このTriple Mapは「3本足 (=3つのpredicate - 述語)」をもつ。

```turtle
<#OrderMapping> a rr:TriplesMap ;
    rr:logicalTable [ ] ;           # 1本目
    rr:subjectMap [ ] ;             # 2本目
    rr:predicateObjectMap [ [ ] ] . # 3本目

```

3つのR2RML語彙がpredicate - 述語に使われている。これらのobject - 目的語になっている角括弧は「ブランクノード」といって、これもRDFの文法である。ネットを検索すると「匿名リソースや構造化された詳細情報を表現する際に使用されます」とある。3本目のブランクノードははネスト (入れ子 / 階層) 構造になっている。

ここで新たなチャレンジが生じる。ブランクノードにはURIがない (「匿名」) ので、トリプルストアでは適当な番号が振られてしまい、どれがどれかはpredicateでしか判別できない。また、ブランクノードの内部構造はGraph Explorerでは表示不可能である。なのでR2RMLはturtleなりで書かれたソースを読んで理解する他ない。今やろうとしているのはまさにそれである。とはいえ、とりあえず3本足はこのように表示可能である。絵の下の方には最初に書いたRDFがあって、それがこのR2RMLのターゲットになる。

![p18fig2](/media/p18fig2.png)

#### 1本目：logicalTable

それでは3本足を順に見て行く。まずは`rr:logicalTable`、上記図中では真ん中の線である。ブランクノードは図中では`b0`と表示されている。これはFusekiがttlロード時に勝手に振ったリソース名で、ttlのコードには登場しない。

さて、この`rr:logicalTable`のブランクノードの中には何があるのだろうか？

```turtle
<#OrderMapping> a rr:TriplesMap ;
# 1本目
    rr:logicalTable [ rr:tableName "ORDERS_TABLE" ] ; 
```

ブランクノードの中の`rr:tableName`はpredicate、"ORDERS_TABLE"はobjectのリテラルである。subject - 主語が見当たらない、のではなく、ブランクノードそれそのものがここではsubjectである。

となると、カンが効けばブランクノードの中に述語・目的語を複数書き並べるくことも出来そう、ということにも気付く。

ここでは"ORDERS_TABLE"というSQLテーブルの物理名が登場するのがミソである。RDFから直接SQL文を生成させるのがR2RMLの最終的な狙い、ということである。

#### 2本目：subjectMap

さてお次は図中の右側の足、`rr:subjectMap`である。こちらの中身は以下の如しである。

```turtle
<#OrderMapping> a rr:TriplesMap ;
# 2本目
    rr:subjectMap [ 
        rr:class ex:Order ; 
        rr:template "http://example.com/order/{ORDER_ID}" ;
    ] ; 

```

ここではすでに「ブランクノードに複数トリプル」が使われている。ここに書かれたことの意味をGenAIに聞けば易しく(?)解説してくれるが、それを元に自分なりに端的に整理すれば、`rr:class`はテーブルの、`rr:template`はカラムの、それぞれマッピング、ということになる。テーブル名は`order`だろう、というのはブランクノード2行目のURIを見ればわかりそうなものだが、コードで書く、というのはそんなものである。

URIの最後の`{ORDER_ID}`はいうまでもなくカラムの物理名である。ここに検索値を代入して、WHERE句を直接生成してくれる、という「ガチの」マッピング仕様がR2RMLである。

#### 3本目：predicateObjectMap

そして3本目、`rr:predicateObjectMap`である。ネストしたブランクノードには何が入っているのか。

```turtle
<#OrderMapping> a rr:TriplesMap ;
# 3本目
    rr:predicateObjectMap [
        rr:predicate ex:placedBy ; 
        rr:objectMap [
            rr:template "http://example.com/customer/{CUST_ID}" ;
        ]
    ] . 
```

ここでも先ほどと同様、最初の (外側の) ブランクノードはふたつのトリプルが書かれている。ひとつめは`rr:predicate ex:placedBy`、これは2本目の`rr:class ex:Order`とペアでなる。2本目は「注文」というエンティティだったが、3本目のこちらは「発注者」というそのエンティティの属性である。

ふたつ目のトリプル`rr:objectMap`のobjectは、さらにブランクノードになっているが、よく見れば2本目の`rr:subjectMap`と相似形で、こちらははcustomerテーブルの`{CUST_ID}`属性、という表現であることがわかる。中のpredicateとobjectMapをあわせてpredicateObjectMap、というわけだ。

コードはこれでおしまいである。最終的には以下の内容となる。

```turtle
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rr: <http://www.w3.org/ns/r2rml#> .
@prefix ex: <http://example.com/ns#> .

# ドメインオントロジーの記述

ex:Customer a owl:Class ;
    rdfs:label "顧客" .

ex:Order a owl:Class ;
    rdfs:label "注文" .

ex:placedBy a owl:ObjectProperty ;
    rdfs:label "注文者" ;
    rdfs:domain ex:Order ;
    rdfs:range ex:Customer .

# R2RMLマッピングの記述

<#OrderMapping>
    a rr:TriplesMap ;
# 1本目
    rr:logicalTable [ rr:tableName "ORDERS_TABLE" ] ; 
# 2本目
    rr:subjectMap [
        rr:class ex:Order ; 
        rr:template "http://example.com/order/{ORDER_ID}" ;
    ] ;
# 1本目
    rr:predicateObjectMap [
        rr:predicate ex:placedBy ; 
        rr:objectMap [
            rr:template "http://example.com/customer/{CUST_ID}" ;
        ]
    ] .

```

さて、このttlをFusekiにロードしてGraph Explorerに読ませると、このような絵が出来る。

![p18fig3](/media/p18fig3.png)

一応全部繋がっている。孤立ノードが無いのは目出度いことである。しかしこれはどのような理屈で繋がっているのか。今回は長くなったので、その読み解きはまた次回。

([Part19につづく](part19.md))
