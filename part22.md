## ビジネスメタをオントロジーで書く - Part 22

### Second data source: university 2

[Ontopチュートリアルの後半](https://ontop-vkg.org/tutorial/basic/university-2.html)である。H2のスキーマ`uni2`のテーブルが題材に加わる。テーブルは3つ、`person`と`course`はマスタ、`registration`は交差表である。

マッピングではいくつか新たな発見がある。例えばuni1では気が付かなかったが、`uni2-course`のマッピングに、

`:uni1/course/{c_id} :isGivenAt :uni1/university`

とある (最初と2番目のトリプルは省略してある)。Protégéのクラス階層を見ても`university`は無いので、これはマッピングのみで必要なリテラルである。マッピングはturtle形式のR2RMLでオントロジーと混ぜて書くより、OntopのOBDAファイルのように、別のアーティファクトとして外出しにした方がやはり良さそうである。

そもそもオントロジーとマッピングではオーナーが異なる。「大学」という概念は本来オントロジーにあっても然るべきではあるが、マッピングの都合でリテラルを書かれてしまうと、オントロジストの方は「汚された」と感じてしまうかも知れない。ここは「適度な距離感」があった方がよさそうだ。

### 物理が違っても同じ意味

マッピングを進めると、何やらuni1と同じクラスにマップされているものがあることに気づく。例えば`:Course`である (上記の`university`行は省略)

```
# uni1の「科目」マッピング
mappingId	uni1-course
target		:uni1/course/{c_id} a :Course ;
                :title {title} ;
source		SELECT * FROM "uni1"."course"

# uni2の「科目」マッピング

mappingId	uni2-course
target		:uni2/course/{cid} a :Course ;
               :title {topic}^^xsd:string ;
source		SELECT * FROM "uni2"."course"

```

これらはテーブル名は同じだが、H2で見るとuni1は2列、uni2は4列で全く別物、列名も異なる。これぞ「世界の現実」であり、膝を打つ瞬間である。そしてこれがまさに「テーブルをそのまま概念クラス扱いする」のが現実には意味を為さない証だ。それは「クラス爆発」の結果であって、「概念収束」を旨とするオントロジーとは根本的に相いれない。「物理名は同じだがスキーマが異なるので別概念」ではない。同じか別かを決めるのはシステムの都合やAIではなく、あくまで人間である。

上の例では`c_id`と`cid`が`:Course`、`title`と`topic`が`:title`にそれぞれマップされている。これを以下のSPARQLで全件抜くとどうなるか。この程度なら「科目一覧見せて」のひと言で、普通のLLMで問題なく書ける筈だ。

```sparql
PREFIX : <http://example.org/voc#>
SELECT ?course ?title WHERE {
  ?course a :Course ;
          :title ?title .
}
```

結果はご覧の通りである。ちゃんと`uni1`と`uni2`通して検索された「科目一覧」になるのである。

|Course|title|
|---|---|
|`<http://example.org/voc#uni1/course/1234>`|"Linear Algebra"^^xsd:string|
|`<http://example.org/voc#uni1/course/1235>`|"Analysis"^^xsd:string|
|`<http://example.org/voc#uni1/course/1236>`|"Operating Systems"^^xsd:string|
|`<http://example.org/voc#uni1/course/1500>`|"Data Mining"^^xsd:string|
|`<http://example.org/voc#uni1/course/1501>`|"Theory of Computing"^^xsd:string|
|`<http://example.org/voc#uni1/course/1502>`|"Research Methods"^^xsd:string|
|`<http://example.org/voc#uni2/course/1>`|"Information security"^^xsd:string|
|`<http://example.org/voc#uni2/course/2>`|"Software factory"^^xsd:string|
|`<http://example.org/voc#uni2/course/3>`|"Software process management"^^xsd:string|
|`<http://example.org/voc#uni2/course/4>`|"Introduction to programming"^^xsd:string|
|`<http://example.org/voc#uni2/course/5>`|"Discrete mathematics and logic"^^xsd:string|
|`<http://example.org/voc#uni2/course/6>`|"Intelligent Systems"^^xsd:string|

### スーパークラスは別、でもサブクラスはひとつ

Ontopの芸当はまだある。「スーパークラスが別概念でもサブクラスはひとつ」というマッピングも出来るのだ。H2のテーブル`uni1.academic`には生徒は含まれないが、`uni2.people`は生徒も含む。しかし例えば"Full Professor"というサブクラスは互いに共通である。ただし区分値が異なっていて、これも「世界の現実」あるあるである。そんな現実に引きずられると「区分値が違うので別概念」となるところだが、ここではそんなことに悩む必要はない。淡々と以下の如くマッピングするだけだ。

```
mappingId	uni1-fullProfessor
target		:uni1/academic/{a_id} a :FullProfessor . 
source		SELECT * FROM "uni1"."academic" WHERE "position" = 1

mappingId	uni2-fullProfessor
target		:uni2/person/{pid} a :FullProfessor . 
source		SELECT * FROM "uni2"."person" WHERE "status" = 7

```

これで前半チュートリアルと同じ[「FullProfessorの姓は何?」クエリ](part20.md#sparqlクエリを試す)を流すと以下の結果となる。

|prof|lastName|
|---|---|
|`<http://example.org/voc#uni1/academic/1>`|"Chambers"^^xsd:string|
|`<http://example.org/voc#uni1/academic/12>`|"Josephina"^^xsd:string|
|`<http://example.org/voc#uni2/person/6>`|"Scott"^^xsd:string|

…しかし「科目一覧」もそうだが、違うテーブルの違うPKのカラムが一列になっているのは、SQLしか知らない向きにはどうも解せない。試しに"SQL Results"タブお隣の"SQL translation"タブで、OntopがH2に投げているクエリをコピーしてH2で実行するとこうなる。

|a_id1m1|last_name4m9|pid1m2|v0|
|---|---|---|---|
|1|Chambers|null|0|
|12|Josephina|null|0|
|null|Scott|6|1|

関西でいうところの「てれこ」である。それはそうだ。SQLならこうにしかならない。そしてここでURIの意義を、ようやく理解するのである。URLと同様、URIも「世界で一つだけ」なのだ。そしてこのクエリ結果は、`<http://example.org/voc#uni2/person/6>`というURIが指し示す「スコットさん」という「個体」は (この世に) 「存在する」と言っているのだ。だから「存在論」、すなわち「オントロジー」なのである。

科目一覧もFullProfessorも、元のテーブルのキーの値は「たまたま」被ってないが、被っていてもURI全体でユニークなのでひとつの列に並べることが出来る。Ontopは、行きは「SPARQL→SQL変換」、帰りは「テーブルスキーマ→RDFパーサー」の「行ってこい」で仕事をしているのである。

### ドメイン推論

前回の推論サンプルも試してみよう。お題は[「講師(`teacher`)一覧」](part20.md#inference-kicks-in)である。実行するとuni2.personの一部がちゃんと追加されていることがわかる。Ontopが書いたSQLを見てみよう。以下の部分はuni1単独と同様`:Teacher`クラスからの単純な階層推論である。

```sql
(
  SELECT V14."pid" AS "lecturer1m3"
  FROM "uni2"."person" V14, (VALUES (7), (9), (8)) AS V15 ("v13m1")
  WHERE V14."status" = V15."v13m1"
)
```

しかし結果にはGraduate Student (#3と#9) およびPostSoc (#5) が含まれている。これは以下のSQLによる。

```sql
(
  SELECT V12."lab_teacher" AS "lecturer1m3" 
  FROM "uni2"."course" V12
)
```
このSQLの元はこのマッピングである。

```
mappingId    uni2-lab-teacher
target       :uni2/person/{lab_teacher} :givesLab :uni2/course/{cid} .
source       SELECT * FROM "uni2"."course"

```

このマップに至る推論経路をProtégé上のTBoxで読み解くと、`:Teacher`は`:teaches`する (ドメイン推論) → `:teaches`のサブクラスは`:givesLab`(階層推論) 、ということになろう。上のマップで`givesLab`している人のインスタンスURIに、URIテンプレートを使って`uni2.course`テーブルの`lab_teacher`列の値を代入している。

TBox側では他にも複数の推論が行われている筈だ。ただしマッピングにヒットしなければSQL生成は行われない。

### AIがもはや「人間並み」ということは…

こういうのを見ると、何だか欧州合理主義の底力といったものを感じる。何やら昔の巨大で豪華なアメ車と、小さく合理的な欧州車のようだ。GenAIにOntopの真似事が出来るかを尋ねてみたが、ちょっとしたデモならバレない自信はあるが、本番の大量データへの複雑なクエリは非現実的 (10倍のコストで1/10のスピード) だそうである。

何よりOntopの動作原理は決定論的、つまり「関数 (functio)」であり、SPARQLからSQLがどう生成されたかは上記のように検証可能だし、同じSPARQL入力に対しては必ず同じSQLが出力される。これなら例えば元帳 (General Ledger - GL) のすぐ手前で使われても不思議ではない。

(そうすると逆にLLMの使いどころも「元帳から遠く離れたところ」であることが見えてくる。実際、今日のAIの活用領域は調査や戦略立案、プレゼン作成など、GLに至るはるか手前だ。GLに近いところでAIに数字の訳を聞いて「その視点鋭すぎます」とか返されても困る)

しかしその一方で、何だか最近「SaaSの死」とか言われて、大手パッケージベンダの株価が下がったりしているが、これはまるで内部統制など皆忘れてしまったかの如き、ではないだろうか。元々人間はインチキしたり間違えたり忘れたりするからITのautomated controlが必要、という「設定」だったのに、いまやLLMは忘れはしないにしろ「間違えたりインチキをするのも人間並み」と考えたりしなくて良いのだろうか。ただ、もしそう考えたとしても、ではそれを一体どうやって監査するのか、というのも見ものではある。

「UIの提供」「アカウント毎課金」という現状のSaaSの外的特徴についての話なのだろうが、それを以て「SaaSの死」とは如何にも大げさである。「決定論的関数」としてのSaaSは無くならない。そして関数でエラーとして撥ねられない、まともな仕訳伝票をLLMが起票できるかどうかは、最終的にはそれが見るTBoxとABoxの出来次第である。

さて、R2RMLは「RDFのTBox」と「SQLテーブルのABox」のマッピング手法だが、リアルビジネスではABoxは全部SQLテーブルとは限らんだろう、というツッコミもあるだろう。しかし心配無用、世の中にはR2RMLのNOSQL拡張である[RML](https://rml.io/) なるものが既にちゃんと存在している (但しこちらはW3C勧告には未採用)。ベルギーのゲント大学発の標準だが、再び欧州の底力を見せつけられるのだろうか。

([Part23につづく](part23.md))

