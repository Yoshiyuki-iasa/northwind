## ビジネスメタをオントロジーで書く - Part 24

### YARRMLとは

[YARRML](https://rml.io/yarrrml/) はYAMLのサブセットで、R2RMLやRMLのマッピングを本来のturtle形式ではなくYAMLを使って書く方法である。OntopのOBDA形式とおそらく同じ狙いだが、あちらはOntopでしか使えない独自形式である。

ページの最初に"Linked Data Generation Rule"である、と書いてある。開発元がRMLと同じという出自もあり、目的はOntopのような仮想化ではなく、JSONなどのRDFへの実体化である。JSONにYARRMLをブツけると個体入りのRDFが出来る。こちらとしては逆方向でdata discoveryに使いたいのだが、これとRDFメタデータをどう絡ませると美味しくなるのかは追々考えるとしよう。

ページの上部にいくつかリンクがあって、チュートリアルメニューが3つ並んでいる。一番右にある"Matey"というのはオンラインのデータ変換ツールである。まずはこれを使って"Tutorial: getting started"の内容を試すことにする。

### オンラインツールによるCSV→RDF変換を試す

こののチュートリアルでは「手書きのYARRMLルールで、異なる形式の複数のデータソースからRDFを生成できるようになる」ことを学習目標に掲げている。しかし「複数のデータソース」、しかも「異なる形式」とはまた随分とハードルを上げたものである。まずは単一のデータソースから始めるのが筋ではないか。

チュートリアル全体をざっと流し読みすると、[1.3.1](https://rml.io/yarrrml/tutorial/getting-started/#writing-rules-in-your-browser)の注意書きのところに「現時点ではMateyはグラフ(g)キーワードを含むYARRRMLルールに対してのRDF出力は出来ません」と書いてあるが、[11章の完成版YARRML](https://rml.io/yarrrml/tutorial/getting-started/#complete-yarrrml-document)にはきっちり`graphs:`が使われている (9章で解説されている)。つまりこのチュートリアルを完走するには、1.3.2以降で長々と書かれているlocal installationが必須、ということになる。そういうのはとりあえず"Getting started"できてからにしたい。

ということで、まずはオンラインツール上で単一データソースのRDF変換から試すことにする。ソースには3章の"[サンプルデータ (people.csv)](https://rml.io/yarrrml/tutorial/getting-started/#example)、YARRMLは11章の完全版から"people"以外のマッピングを抜いたものを以下のように準備する。

```yaml
prefixes:
  ex: http://www.example.com/
  e: http://myontology.com/
  dbo: http://dbpedia.org/ontology/
  grel: http://users.ugent.be/~bjdmeest/function/grel.ttl#

mappings:
  people:
    sources:
      - ['people.csv~csv']
    s: ex:$(id)
    po:
      - [a, schema:Person]
      - [schema:givenName, $(firstname)]
      - [schema:familyName, $(lastname)]
      - p: e:debutEpisode
        o:
         value: $(debut episode)
         datatype: xsd:integer
      - p: dbo:hairColor
        o:
         value: $(hair color)
         language: en
```

YARRMLの全体の構成を見るとざっと以下の様になっていることが分かる。`s:`が`rr:subjectMap`、`po:`が`rr:predicateObjectMap`、`p:`は`rr:predicate`、`o:`が`rr:objectMap`にそれぞれ対応するYAMLキーである (チュートリアルの例で`e:`なるプレフィックスがあって見た目が紛らわしいがそれとは別物である)。属性はフロー(`[]`)なりブロック(`-`) なりのシーケンス (配列) で並べる。なるほど簡単といえば簡単である。YAMLなのでエディタの入力補完も (煩わしいくらい) 効く。

```yaml
mappings:
  entity:
    sources: # ソースファイルの名前と形式、エンティティとユニークキー
    s: # 主語
    po: # 述語・目的語 (属性)
      - p: # 述語
        o: # 目的語
      - [# 述語, # 目的語]
```

Mateyの画面を開いて、`people.csv`のファイル名とその内容を"Input: Data sources"の"Create new source"で登録し、上記のYARRMLを"Input: YARRML rules"に貼って、緑の"Generate RDF" ボタンを押すと、無事以下のRDFが生成された。"Getting started"である。

```rdf
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix ma: <http://www.w3.org/ns/ma-ont#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix sd: <http://www.w3.org/ns/sparql-service-description#> .
@prefix schema: <http://schema.org/> .
@prefix ex: <http://www.example.com/> .
@prefix e: <http://myontology.com/> .
@prefix dbo: <http://dbpedia.org/ontology/> .
@prefix grel: <http://users.ugent.be/~bjdmeest/function/grel.ttl#> .

ex:0 dbo:hairColor "pink"@en ;
	e:debutEpisode "1"^^xsd:integer ;
	schema:familyName "Dragneel" ;
	schema:givenName "Natsu" ;
	rdf:type schema:Person .

ex:1 dbo:hairColor "dark blue"@en ;
	e:debutEpisode "2"^^xsd:integer ;
	schema:familyName "Fullbuster" ;
	schema:givenName "Gray" ;
	rdf:type schema:Person .

ex:2 dbo:hairColor "black"@en ;
	e:debutEpisode "21"^^xsd:integer ;
	schema:familyName "Redfox" ;
	schema:givenName "Gajeel" ;
	rdf:type schema:Person .

ex:3 dbo:hairColor "blonde"@en ;
	e:debutEpisode "1"^^xsd:integer ;
	schema:familyName "Heartfilia" ;
	schema:givenName "Lucy" ;
	rdf:type schema:Person .

ex:4 dbo:hairColor "scarlet"@en ;
	e:debutEpisode "4"^^xsd:integer ;
	schema:familyName "Scarlet" ;
	schema:givenName "Erza" ;
	rdf:type schema:Person .
```

出来たturtleをダウンロードしてProtégéで開くと、ソースのcsvの各行からIndividuals (個体) が5つ出来ていることが分かる。属性は全てAnnotation Propertyである。RDFで書かれたABoxだ。

MateyのRDF出力欄のプルダウンを押すと、YARRMLから生成されたRMLのソースも見ることが出来る。こちらはTBoxと言えなくもないだろうが、見た目はマッピング定義を個体にもつABoxである。

### 複数ソースの結合

単一ソースをMateyを使ってゃんと変換できることは分かった。お次は「複数ソース」である。[8章の5](https://rml.io/yarrrml/tutorial/getting-started/#how-to-link-two-entities)の説明に従い、ファイルの後ろに"episodes"のマッピングを追加する。

"people"マッピングには"appearIn"属性を追加する。値は"people"の"debut episode"にマッチする"episodes"の"number"で埋める。ここで出てくる`str1` `str2`は`grel:valueParameter` `grel:valueParameter2`のショートカットである。YARRMLで`function: equal` と書くと呼び出されるGREL (Google Refine Expression Language) というライブラリで定義されたパラメータである。

これがYARRMLファイルの最初の`prefixes:`に`grel:`と書いてある理由でもある (オントロジーにそういう技術的概念を持ち込むのはどうか、という感じはしなくもない)。URIがなぜゲント大学 (ugent.be) なのか、というと、GRELをRDFで使おうと考えたのがそこだけだったから、ということのようだ。Refine自体はすでにGoogleを離れ、[OpenRefine](https://openrefine.org/)としてオープンソース化され今に至るようである。これ自体も「乱雑なデータを扱うための強力で無料のオープンソースツール」だそうで、何だか面白そうだが今回はスルー、である。

YARRMLが書けたら、Mateyのソースファイルに"episode.csv"を追加して、RDFを生成させてみる。確かに8章の最後に書かれているのと同じ`ex:0 e:appearsIn ex:episode_1 ;`といったトリプルが追加されている。元々peopleのAnnotation Propertyだった値をトリプルとして実体化した、ということか。

折角なので久々にFuseki / Graph Explorerを起動して、出来たturtleをロードするとこういう風になる。ラベルは付けてないが上がpeople、下がepisodeでそれぞれ左から順番に並べてある。元のcsvと見比べてみれば確かにその通り、である。

![p24fig1.png](/media/p24fig1.png)

一応これで「複数のデータソース」からのRDF変換までは上手くいったことになる。次回は「異なる種類」にチャレンジである。

(Part25につづく)
