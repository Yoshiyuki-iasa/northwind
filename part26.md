## ビジネスメタをオントロジーで書く - Part 26

### (本当に) ビジネスメタをオントロジーで書く

これまで様々な技術的手段の試行錯誤を通じて、いわば「外から」オントロジーがどんなものであるかの理解を深めてきたが、今回はいよいよコンテンツとしてのオントロジーの具体的な書き方にチャレンジ、である。

これまでの試行錯誤で実際に分かったことは「物理からのボトムアップだけではオントロジーにならない」ということだ。ではトップダウンはどうするか、というと、実はちゃんと「既成のもの」があるのである。これらを上位概念として、ローカルな概念をマッピングしてゆけば良いのだ。

この「既成のもの」にはスコープやレベル違いで様々な種類が存在するが、一番とっつきやすいのが[gist](https://www.semanticarts.com/gist/)である。いわゆる上位オントロジーのひとつだが、ビジネス用途なら上位オントロジーとしてはこれ一択であるようだ。そのうえでインダストリー別のいわゆるドメインオントロジーも世の中には沢山あるので、必要に応じそれらとのマッピングも加えていく、という手順を踏む。

今回のマッピング元となるローカルのオントロジーには[Ontopのチュートリアル](part20.md#first-data-source-university-1)の`university.ttl`を使うことにする。クラス16個、プロパティ10個しかなく、手始めとしてはこのくらいで十分であろう。

マッピングにはProtégéを使う。同時にコードエディタで`university.ttl`の中身を観察しながら進めることにする。

ということで、まずは`university.ttl`をProtégéとエディタで開く。`university.ttl`をコピーしてエディタ上でオリジナルと比較できるようにしておくと良いだろう。

次に、マッピング先となるgistのインポートである。パープルのActive Ontology画面下のOntology ImportsにあるDirect importsの+ボタンを押す。gistはローカルで参照してもよいが、今回はURLを使おう。2番目のオプションを選択して、入力欄に`https://w3id.org/semanticarts/ontology/gistCore`を入力する。

エディタでttlファイルの最初にプレフィクスが列挙されている。その下を見ながらProtégéでsaveを実行すると以下のように2行目が追加される。

```turtle
<http://example.org/voc#> rdf:type owl:Ontology ;
                           owl:imports <https://w3id.org/semanticarts/ontology/gistCore14.0.0> .
```

プレフィクスにはgistはまだないので、これも追加しておこう (プレフィクスが無いとURIが長くなって可読性が落ちる)。ProtégéのOntology Importの右にOntology Prefixesがあるので、Prefix名に"gist"、Prefixに`https://w3id.org/semanticarts/ns/ontology/gist/`を入れてokを押す。セーブするとエディタ側でもgistのプレフィクスが追加されたことが確認できる。また、gistには[skos](https://www.w3.org/TR/skos-reference/)も使われているので、これも`http://www.w3.org/2004/02/skos/core#`としてプレフィクスに追加する。

Protégéのクラスツリーをみてよう。なにやら数が増えているが、これらがgistのクラスである。Viewメニューから"Show only active ontology"を選ぶと元々のuniversityクラスだけが表示される。

ビューを元に戻し、Reasonerメニューで"HermiT"を選択しReasonerを開始する。これで準備完了である。

### Studentクラスのお引越し

Protégéのクラスツリーを見ると"Student"クラスは`foaf:Person`のサブクラスになっている。gistではPersonクラスはどこにいるのかと、ProtégéでCtrl+Fを押して検索すると"Physical Identifiable Item"→"Living Thing"のサブクラスに`gist:Person`が発見できる。

では、クラスツリー上でStudentクラスを元の場所からここにdrag&dropしてみよう。セーブしたのちエディタで見ると正しく`gist:Person`のサブクラスとなった。

```turtle
:Student rdf:type owl:Class ;
         rdfs:subClassOf gist:Person .
```
`foaf:Person`のサブクラスには"Faculty Member"も居るので、こちらも同様に`gist:Person`へお引越しする。

### "Course"とは何か？

"Students"や"Faculty Member"は、foafでもgistでも同じPersonクラス、ということで単純な横移動である。しかしuniversityのルートに居る他のクラス、たとえば"Educational Institution"はgistの何のサブクラスなのかといえば、これも単に"Organization"で問題ないだろう。一方、もうひとつの"Course"クラスはどうだろうか？

困ったときにはGenAI、である。別途ダウンロードしてあるgist14coreも読ませて提案させると`gist:ServiceSpecification`ではないか、とのご意見である。

gist上は相当深い階層にいる要素だが、なぜそう考えたのだろうか。階層上のそれぞれの定義 (`skos:definition`のリテラルとして記述してある) を見てみよう。

|level|URI|skos:description|
|---|---|---|
|1|gist:Intention|A goal, desire, or aspiration.|
|2|gist:Specification|One or more characteristics that specify what it means to be a particular type of thing, such as a material, product, service or event. A specification is sufficiently precise to allow evaluating conformance to the specification.|
|3|gist:CatalogItem|A description of a product or service to be delivered, given in a sufficient level of detail that a receiver could determine whether delivery constituted discharge of the obligation to deliver.|
|4|gist:ServiceSpecification|A description of something that can be done for a person or organization (which produces some form of an act).|
|5|:Course|-|

GenAIに、これらの定義を継承する形で"Course"の定義作成を指示するとこんなのが出来る。上のコンテキストが無くても書けるんじゃないか、とも思えるが、本当にそうだろうか？

> 講義 / 課程 / コース: 特定の学習目標を達成するために設計された教育サービスの仕様。学習内容、単位数、評価基準など、提供される教育活動の具体的な要件を定義したものを指す。

これはマトモな「ビジネスメタ」といえるのではないか。定義文にちゃんと「仕様 ("Specification")」が入っているのがミソである。階層の上下できちんと汎化 - 特化で論理整合している。

データ要素の定義の記述は慣れないと中々大変だが、description込みでパブリックに提供されるオントロジーを、それらの前提知識を持つGenAIと組み合わせれば、その負担はかなり軽減されるのではないか。

### 冗長なローカル排他を始末する

現状では以下のように、「Educational InstitutionはPersonではない」「CourseはEducational InstitutionでもPersonでもない」という排他論理が記述されている。

```turtle
:Course rdf:type owl:Class ;
        rdfs:subClassOf gist:ServiceSpecification ;
        owl:disjointWith :EducationalInstitution ,
                         foaf:Person .

:EducationalInstitution rdf:type owl:Class ;
                        rdfs:subClassOf gist:Organization ;
                        owl:disjointWith foaf:Person .
```

しかし、このような排他はすでにgistにも書いてあるので不要である。残しておくと推論に悪影響も及びかねないので削除した方が良い。

一旦Reasonerをオフにして、それぞれからdisjointの記述を削除する。Reasonerを再びオンにすれば、gistの排他が上位クラス ("Educational Institution" の "Organization"、"Course" の "Intention") ですでに排他が掛かっていることがわかる。

### プロパティのgistへのマッピング

クラスは親の数が限られていたので簡単である。次はプロパティだが、３つあるデータプロパティ (姓・名・講義のタイトル) は全て`gist:Name`のサブプロパティである。姓・名についてはドメインも上記に合わせて`foaf:Person`から`gist:Person`に変更する。ここまではあまり悩むこともない。

しかしオブジェクトプロパティはそうは行かないようだ。やはり「振る舞い」の概念は本質的に発散傾向である。例えば"attends"についてどうするのが良いかGenAIに聞いてみると、`gist:isMemberOf` `gist:hasParticipant`の何れかではないか、という返事である。gist上のそれぞれの定義は以下となっている。

> `gist:isMemberOf`:"Relates a member individual to the thing, such as a collection or organization, that it is a member of. (メンバーである個人が、所属する集合体や組織などの対象に関連付けられる。)"

> `gist:hasParticipant`: "Relates something (e.g. an agreement) to things that play a role, or take part or are otherwise involved in some way. (何か（例：合意）を、役割を果たすもの、関与するもの、あるいは何らかの形で関わるものに関連付ける。)"

universityの元々の記述は"Person attends Course"なので、語順的には前者であろう。ということで`:attends`は`gist:isMemberOf`のサブプロパティに付け替える。Domainもfoafからgistに変更する。

`:isGivenAt`も汎化概念としては`:attends`同様`gist:isMemberOf`とするしか無かろう。あまりしっくりこないが、これがトップレベルオントロジーというものだ。教育ドメインのオントロジーも種々有りそうで、そういうものを持ってくればフィットする語彙が見つかるのだろう。あるいはむしろドメインオントロジーの定義済み語彙をそのまま使用するのではないか。

### 逆プロパティは維持できるか

オリジナルのuniversityでの明示的な記述は"teacher teaches course"で、`:isTaughtBy`は`:teaches`の逆プロパティ (`owl:inverseOf`) とだけ書いてある。一方gistを見ると`owl:inverseOf`は使われていないので、Reasonerでエラーにならないか心配である。universityは元々Ontopの推論機能の検証用で、チュートリアルにはこの逆プロパティを使う例は無かったので、そういったケースを試そうとしない限りは問題ないが、書き手の意図を尊重して残しておきたい。

Reasonerを一旦止めて`:teaches`を`gist:contributesTo`、`:isTaughtBy`を`gist:hasGiver`のサブプロパティにそれぞれ移してみる。`:teaches`のinverseはそのままである。しかる後にReasonerを起動しても特にエラーとならないのでこのままとしよう。

### 独自語彙を残す

さて、残るは`:isSupervisedBy`である。これは以下の記述で使われている。

```turtle
:GraduateStudent rdf:type owl:Class ;
            rdfs:subClassOf :Student ,
                            [ rdf:type owl:Restriction ;
                              owl:onProperty :isSupervisedBy ;
                              owl:someValuesFrom :Professor
                            ] .
```

これを日本語でいえば、「院生とは、一人以上の教授に supervised される、という制約を持つクラスのサブクラスである」となるが、この`:isSupervisedBy`の汎化概念がgistにあるか、というと、これが無いのである。GenAI曰く、最初からgistベースで作っていればRoleクラスで関係モデル化するが、gistはポリシーとして 「ユーザーはドメイン固有プロパティを自由に定義してよい」 ということになっているので問題ない、とのことだ。上位オントロジーを使うに当たってはこういう割り切りも必要ということか。また、ドメインオントロジー標準の必要性もこんなところに現れるのだろう。

最後にProtégéで`foaf:Person`クラスが使われていないことを"Usage"タブで確認し、これは削除してしまおう。但しfoafのプレフィクスは姓名で使われているのでこれは残留である。

折角なのでOntopでクエリが動くか、オリジナルと比較しながらチュートリアルの課題や、`university.q`のクエリサンプルも試してみたが、結果は完全に一致した。但しマッピング、サンプルクエリともgistとskosのプレフィクスは追記、`foaf:Person`となっているところは`gist:Person`に置き換えるのはいうまでもない。

今回は比較的小さなオントロジーを例にして、それにどのように上位オントロジーを継承させるかを試した。おそらくこれが「ビジネスメタをオントロジーで書く」本当の方法なのだろう。なんだか 「服を着せる」 ようなイメージだ。着ている人 (= 企業組織) はそれぞれ固有の存在だが、服は他人と被ったりする。上位オントロジーは無課金キャラのようなものか。ドメインオントロジーはTPOにマッチしたスタイルだろう。オントロジーはデジタル時代の身だしなみ、なのかもしれない。

次回も引き続きgistを深堀りする。northwindの復活である。

([Part27に続く](part27.md))
