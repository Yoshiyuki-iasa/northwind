## ビジネスメタをオントロジーで書く - Part 25

### "Quad"とは何か

今回は[YARRRMLのビギナー向けチュートリアルの後半](https://rml.io/yarrrml/tutorial/getting-started/#how-to-add-triples-to-a-graph)である。「episodesがJSONやXMLのケース」について、12章以降に解説があるが、9章、10章についてもMateyでの実習はしない (9章の内容はそもそもMateyが対応していない)、 一応目を通しておくことにする。なお11章は前述の通りこのチュートリアルでのYARRRML完成版である。

まず9章だが、最初にやっているのは「peopleエンティティを`ex:Characters`という名前付きグラフ (Named Graph) に割り当てる」というものである。

結果`ex:0 a schema:Person ex:Characters .`といったquadが出来る、と書いてある。これはTurtle上でN-Quadsと呼ばれる形式の表記なのだが、

- `ex:0` - Subject (URIの短縮形)
- `a` - Predicate (`rdf:type`の略)
- `schema:Person` - Object (`exe:0`のクラス)
- `ex:Characters` - Graph (このデータが属するコンテキスト)

という「SPOのトリプルにGを加えた4つ」なのでQuad、という。つまりgraphそのものもリソース扱い、ということである。これは実際にSPARQLを掛けるときに、クエリをシンプルに出来たり、出し先を振り分けたり、大量のトリプルをグラフごと一発で削除したり、アクセス制御したり、といった諸々の役に立つ。ちょっとABox寄りの話ではあるが、TBoxでもドメイン分割などに使えるかも知れない。

### "po-specific triples"って何!?

9章の後半である。これは最初何を言っているのかさっぱり分からなかったが、要するに"triples with specific PO-pairs"、つまり「特定のPO (つまりRDBでいえば列と値) を持つトリプル」を「別の名前付きグラフにも割り当てる」という話である。"po-specific (PO固有の)"が主語なら"graph assignment (グラフ割り当て)"といった表現が妥当だろう。

そうと分かればここで言っていることは単純である。9章前半ではpeopleエンティティのマッピングで「全て`ex:Characters`という名前付きグラフ」という定義を追加している。ここで忘れてはいけないのは、「その他のトリプルはdefault graphに居る」ということである。例えばepisodeエンティティのマッピングには`graphs:`の定義はないので、これらはdefault graphにある。

そして後半でpeopleマッピング内の「ある特定のPO」、つまり`p: e:debutEpisode`と`p: e:appearsIn`のふたつについては、そのトリプルは`ex:Episodes`という名前付きグラフ「にも」属する、という定義を書き足している、ということである。`ex:Characters`から`ex:Episodes`に「引っ越した」のではない、という点も、忘れてはいけないもうひとつのポイントである。

章末のサンプルを見ると`e:debutEpisode`と`e:appearsIn`の行は右端列のグラフ違いで2行ずつ存在することがわかる。変な英語表現 (native speakerはああは言わないのでは?) で少々振り回されたが、どうということはない。

### GREL関数とYARRRML完成版

さて10章だが、ここは例の「乱雑なデータを扱うための強力で無料のオープンソースツール」である[GREL](https://openrefine.org/)のライブラリを使ってデータをキレイにする、という、TBoxにはあまり関係ない話である。例として`grel:toUpperCase`という関数を使ってあるリテラルの文字列を大文字にする、という処理が説明されている。RMLをETLとして使用する際には役に立つだろう。しかしここではあまり本筋ではないので深堀りは止めておく。

11章は前述の通り本チュートリアルで出来るYARRRMLの完成版である。ただしこれで最終的にどんなturtleが出来るか、という「答え」は提供されていない。各章にQuadの出力例などはあるが、サンプルのYARRRMLはスニペットで、実際にMateyで動作させるには完全なYARRMLに補完する必要がある。全体として親切なのか投げやりなのか良く解らないチュートリアルではある。

### JSONとXMLの変換マッピング

12章でお待ちかね「異なる種類」の登場である。「複数ソース」では「CSVとCSV」を試したが、今度は「CSVとJSON」および「CSVとXML」である。まずは「CSVとCSV」で使ったYARRMLを貼っておく。11章の完成版と異なり、こちらはMateyでも動かせるように、名前付きグラフに関する記述を抜いてある。GREL関数による文字変換もここでは省略した。

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
      - p: e:appearsIn
        o:
          mapping: episode
          condition:
            function: equal
            parameters:
              - [str1, $(debut episode), s]
              - [str2, $(number), o]
  episode:
    sources:
      - ['episodes.csv~csv']
    s: ex:episode_$(number)
    po:
      - [a, schema:Episode]
      - [schema:title, $(title)]
```

JSONやXMLを試すのはepisodesエンティティである。チュートリアルで提供されている夫々のサンプルファイルをMateyのソースに追加で登録し、上のYARRMLの最後にある`episode:`マッピングの`sources:`部分をチュートリアルの例に都度入れ替えて、"Create RDF"を実行してみれば良い。差分だけを対比用に並べるとこうである。

```yaml
- ['episodes.csv~csv'] # csv用
- [episodes.json~jsonpath, "$.episodes[*]"] # JSON用
- [episodes.xml~xpath, /episodes/episode] # XML用
```

チルダの後ろは参照形式 (referenceFormulation) で、CSVはそこで終わりだが、JSONとXMLではカンマの後にiteratorの指定が入る。

ファイル形式は違えどepisodeの内容自体はどのファイルも同じなので、turtle出力結果自体は変わらない。

さて、RMLの基本はこれでお終いである。Graph Explorerから少しテクニカルなトピックが続いたので、次回はオントロジーの中身や書き方に戻ることにする。

(Part26につづく)
