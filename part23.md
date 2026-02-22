## ビジネスメタをオントロジーで書く - Part 23

### R2RMLのおさらい

[RML](https://rml.io/)はSQL専用であるR2RMLをCSV、JSON、XMLでも使えるように拡張したものなのだが、その原理を理解する上で、まずはR2RMLがどんなものだったのかを改めて確認したい。もっといえば、そもそも表形式のデータがどのようにRDFトリプルに「変形」されるか、から見直してみたいと思う。

これは一度[Part13](part13.md#プロパティドメインレンジ) で取り上げているが、一言でいえば 「s (p,o) の表 → s,p,oのグラフ」、となる。もう少し分かりやすく表現すると、「主語s → 述語p → 目的語o」の並びは、表形式にはこのように変形可能である。

||述語p|
|:---:|:---:|
|主語s|目的語o|

ここで、
- 「学生 (s)」は「科目 (o)」を「履修している (p)」

というのをRDFトリプルの例としてみよう。英語ならpsoは昔学校で習ったsvoと同じ語順になるが、ここでは分かりやすいように日本語に合わせてみた。表にするとこうなる。

|(学籍番号)|履修 (p)|
|:---:|:---:|
|学生 (s)|科目 (o)|

ここまではメタデータ (TBox - 概念定義) である。つまり「Aさんは…」といった実データ (ABox - 具体的事実) は含まれない。

このメタデータを意味的にもっと豊かにすれば (課目履修生も学生に入る？など) データの利用価値が上がるだろう、というのがこの書き物における最大のテーマで、これはその前提知識、ということになる。中々先に進まないがまあ仕方がない。

R2RMLとは、このRDFトリプルで書かれたTBoxをSQLテーブル上のABoxにマップする方法である。どのようなものだったを思い出してみよう。細かいところを端折るとこのように整理できる。列 (P,O) の部分は`rr:predicateObjectMap`で括られている。

`rr:logicalTable`

||`rr:predicateObjectMap`<br>`rr:predicate`|
|:---:|:---:|
|`rr:subjectMap`|`rr:objectMap`|

履修登録の例でいえば、

`<#履修登録マッピング> rr:logicalTable [ rr:tableName "履修登録" ]`

||`rr:predicateObjectMap`<br>`rr:predicate`<br>履修|
|:---:|:---:|
|`rr:subjectMap`<br>「学生」クラスの<br>"学籍番号URI"|`rr:objectMap`<br>「科目」クラスの<br>"科目番号URI"|

である。ソースを見るとブランクノードなどで複雑に見えるが、こうしてみると割と単純である。

### RMLは何が違うのか

ここで例えば、履修登録データが中身は同じでもこんなJSONだったとする (キーがダブルバイトだが、実際に処理に掛けたりしないので良しとする)。

```json
{
  "履修登録": [
    { "学籍番号": "S001", "履修科目": "ABC-123" },
    { "学籍番号": "S002", "履修科目": "DEF-456" }
  ]
}
```

これをRMLでマップする。基本的な構造は変わらないが`rml:`という語彙がちらほら使われている。

```turtle
<#RegistrationMapping> a rr:TriplesMap;
  rml:logicalSource [
    rml:source "registration.json";
    rml:referenceFormulation ql:JSONPath;
    rml:iterator "$.履修登録[*]" 
  ];
  
  rr:subjectMap [
    rr:template "http://example.ac.jp/student/{学籍番号}";
    rr:class ex:Student
  ];

  rr:predicateObjectMap [
    rr:predicate ex:takesCourse;
    rr:objectMap [
      rml:reference "履修科目"
    ]
  ].

```

となる。SQLテーブル用のR2RMLマッピングと比較してみよう。学籍番号=STUDENT_ID、履修科目=COURSE_IDで、表記の違いはここでは無視する。

```turtle
<#RegistrationMapping> a rr:TriplesMap;
  rr:logicalTable [ 
    rr:tableName "CourseRegistration" 
  ];
  
  rr:subjectMap [
    rr:template "http://example.ac.jp/student/{STUDENT_ID}";
    rr:class ex:Student
  ];

  rr:predicateObjectMap [
    rr:predicate ex:takesCourse;
    rr:objectMap [
      rr:column "COURSE_ID"
    ]
  ].

```

殆ど同じ、と言っても良いくらいだが、以下のような違いがある。

1. `rml:referenceFormulation` - R2RMLはSQL専用だが、RMLはJSONの他XMLやCSVも扱うので、どの形式かを指定するための語彙が必要になる。

2. `ql:JSONPath` - `ql:`とはQuery Language Ontologyの略で、どのクエリ言語を使うのかを明示する。XMLなら`ql:XPath`、CSVなら`ql:csv`となる。

3. `rml:iterator` - JSONの場合は、RDBの行に当たる部分がどこかは明示しないとわからない。`"$.履修登録[*]`で、その履修登録の配列を行として繰り返す、という意味になる (`$`はルートの意味)。CSVの場合は行は見れば分かるので省略、或いは`""`である。

4. `rml:reference` - `rr:objectMap`で、R2RMLでは`rr:column`となるところである。`rml:referenceFormulation`で指定したJSONPathに従い取得した値を代入する。

RMLは他にもREST APIやストリーミング (kafka、MQTT) などもRDF化できるようである。また、階層が深かったり、といったもっと複雑なJSONやXMLのマッピングなど色々あるだろう。上記はほんのサワリ、ということになるが、今はこれでお腹一杯である。

### R2RMLとRMLは使い方が違う

このように両者は原理的には同じだが、ではRMLを使ってR2RML/Ontopで試したように、SPARQLで透過的にJSONをクエリするか、というとそれは無いだろうな、ということに気づく。つまりR2RMLは「仮想化 (virtualization)」、RMLは「実体化 (materialization)」で、それぞれの使い方は明確に異なる。RMLはJSONやXMLをRDFの「ABox」として実体化するためのものなのだ。

しかしJSONをわざわざRDFトリプルストアにに入れてからSPARQLを投げる、というニーズがそんなにあるのか、というと、それはJSONそのままでは困る人がどれだけ居るのかに依るのではないか。もしRMLが役立つ場面があるとすれば、それはJSONとCSVなど、フォーマット違いの複数のデータセットをまとめてRDFトリプルストアに寄せたい、というケースは考えられる。ETLの代わりのようなものだ。ひょっとすると欧州ではそこまで進んでいるのかも知れない。

それでも「それならSQLテーブルにロードして、R2RMLマッピングでSPARQL投げれば良いじゃん」とか「LPGに入れてOpenCypher」という意見もまた御尤も、ではある。

しかし忘れてはいけない。本来の狙いは「メタデータ」である。どんな意味のデータがどこに実体として存在するかが (人間に限らずAIにも) 分かるようにしたいのだ。そしてJSONといえば大抵のREST APIも、最近だとMCPだってJSONである。"Semantic API"、"Semantic MCP"なんて最高にcoolではないか！GenAIに訊くとこんなセールストークのような浮かれた答えが返ってくる。しかし最近巷で盛り上がっているContext Graphとはこういうハナシなのかも知れない。

> **コンテキストのグラフ化**
>
> MCP経由で取得したバラバラのJSON（Slackの発言、GitHubのコード、カレンダーの予定）を、RMLでリアルタイムにRDF（ABox）化してグラフデータベースに放り込めば「先週の会議で話していたバグ修正のプルリクエスト」といった、時間と文脈を跨いだ複雑な関係性をAIがグラフ探索（SPARQL等）で一瞬で見つけ出せるようになります

しかしここでも「記号と意味とは別」という原則は不変だろう。。RMLがそのまま使えれば素晴らしいし、もしかするとコンテキストグラフ用の新しいマッピング言語がそのうち現れるのかもしれない。TBoxはそういうのも見越して準備すべきだろう。"Future Proof"である。

ところでRMLにはOntop (R2RML) でいうところの"OBDAファイル"にあたる"[YARRRML](https://rml.io/yarrrml/#)"なる記法が存在する。これはやはり知っておくべき、ということで、次回で取り上げることにする。

([Prt24に続く](part24.md))

