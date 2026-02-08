## ビジネスメタをオントロジーで書く - Part 20

### Ontopチュートリアル環境の準備

[Ontop](https://ontop-vkg.org/)を一言でいえば「[関係データベース(RDB)などの非RDFデータに対してSPARQLで問合せを可能とするミドルウェア](https://wiki.lifesciencedb.jp/mw/Ontop.html#:~:text=%E9%96%A2%E4%BF%82%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9(RDB)%E3%81%AA%E3%81%A9%E3%81%AE%E9%9D%9ERDF%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AB%E5%AF%BE%E3%81%97%E3%81%A6SPARQL%E3%81%A7%E5%95%8F%E5%90%88%E3%81%9B%E3%82%92%E5%8F%AF%E8%83%BD%E3%81%A8%E3%81%99%E3%82%8B%E3%83%9F%E3%83%89%E3%83%AB%E3%82%A6%E3%82%A7%E3%82%A2)」だそうである。北イタリアにあるボルツァーノ自由大学 (Free University of Bozen-Bolzano - Bozenは同じ町のドイツ語名。イタリアでもこの辺はドイツ語圏) の研究から生まれ、 Apache2.0ライセンスで公開されている。これをベースとした[有償プロダクト](https://ontopic.ai/en/)も存在するようだ。

ちなみにトップページに"A Virtual Knowledge Graph System"とあるが、これはFusekiのようなRDFトリプルストア無しに、R2RMLマッピングとRDBMSでナレッジグラフを動的に再現している、といった意味であろう。これで普通のSQLデータセットにSPARQLクエリを投げられるようになるわけだが、あくまでSQLテーブルの中身が「読める」だけで、少なくとも現在のところSQLテーブルの更新までは出来ない。

初出は2009年ごろの様だが、アップデートは今日まで継続しており、PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, DB2, Snowflake, Databricks, Google BigQuery, AWS Redshift / DynamoDB などののポピュラーな商用RDBMSの最新版、またDenodo, Apache Spark, AWS AthenaなどのDBフェデレーターもサポートしている。

如何にも大学発のプロダクトらしく、学習教材としての実装ガイドと、マッピングのチュートリアルが提供されている。学習教材の実装には以下のふたつのツールが含まれている。

- Protégé：スタンフォード版のオリジナルに、Ontopプラグインをバンドルした実行ファイルが用意されている。
- H2 DB：一言でいえばFusekiのRDB版ともいうべき「ブラウザだけで動くRDBMS+SQLクライアント」。SQLの学習や検証の手段として広く一般的に使われているようで、UIも日本語化されている。チュートリアル用のサンプルデータを含むzipファイルが用意されている。

H2はもちろんDockerで動かしても良い (直接起動は止め方が面倒くさい) が、どのみちサンプルデータのダンプを抜くために、素のH2を一度は上げる必要はある (チュートリアル版はバージョンが古く、Dockerイメージ版最新バージョンのデータストア形式とは互換性がない)。

Ontopホームページ画面上の"Tutorial"メニューを選択し、まずは"Presentation"章の"Clone this repository"を実行する。続いて左側のアウトラインから"Basics"章の"Database and Ontop Setup"を参考にH2とProtégéを準備する。ただしH2についてはHP上のリンクでなくとも、先ほどのGit Cloneに含まれるZIPファイルを解凍すればよい。JDBCドライバもZIPファイルに入っている (Dockerの場合は新バージョン用のJDBCドライバが必要)。

Git Cloneしたフォルダから`university.ttl`をProtégéで開いて、JDBC接続が上手くいったら、ReasonerメニューでOntopをオンにして、Ontop SPARQLタブから以下のコマンドを流し`<http://example.org/voc#uni1/student/1>`から`5`までの5行が返って来れば準備完了である。

```sparql
PREFIX : <http://example.org/voc#>
SELECT ?s WHERE {
   ?s a :Student .
}
```

### First data source: university 1

左のアウトラインから"First data source: university 1"を読んでみる。最初はH2のテーブルの紹介である。`uni1`と`uni2`のふたつのDBスキーマが切ってあり、このセッションでは`uni1`を使うらしい。`uni1.student`, `uni1.academic`, `uni1.course`と、マスタが3つ。`uni1.teaching`と`uni1.course-registration`のふたつは交差表である。

つぎは"Ontology: "classes and properties"セクションである。`university.ttl`の内容をProtégéの画面上でも確認する。

そして最後は"Mappings"である。先ほどのGit cloneしたフォルダをエディタでも見てみよう、画像ファイル3つの他、以下のファイルが確認できる。

- `dataset-no-ssn.sql` - データダンプの様である。チュートリアルには関係なさそう。
- `university.ttl` - 上で見たもの。「マップ元」になる。
- `university.properties` - JDBC接続設定。Protégéの親タブ"Ontop Mappings"内の子タブ"Connection parameters"の中身。
- `university.obda` - マッピング本体。初期状態では`uni1-student`だけがサンプルで入っている。先ほどのJDBC接続テストのSPARQL実行にはこのマッピングが使われている。親タブ"Ontop Mappings"内の子タブ"Ontology Mappings"を使ってマッピングを追記するのがこのチュートリアルの内容。
- `university.q` - サンプルのSPARQLクエリ。親タブ"Ontop SPARQL"のクエリウィンドウの左に一覧が表示される。

手順に従い"Ontop Mappings"内の子タブ"Ontology Mappings"を開き、残りの`academic`, `course`, `teaching`, `registration`そして `fullProfessor`(およびそのバリエーション) のマッピングを追加する。エディタを見るとエントリが増えていく様子が確認できる。R2RMLとは異なるOBDAというOntop独自書式だが、原理的にはおそらく同じで相互変換も出来るのだろう。この書き方ならブランクノードが無い分、R2RMLの標準記法より分かりやすい感じもする。例えばリソースだとこうである (OBDAファイルから拾って、見やすく成型してある)。

```
mappingId	uni1-student

target		:uni1/student/{s_id} a :Student ; 
               foaf:firstName {first_name}^^xsd:string ;
               foaf:lastName {last_name}^^xsd:string . 

source		SELECT * FROM "uni1"."student"
```

`target`はトリプル3つである。1行目は「スキーマuni1のstudentテーブルがstudentクラス」、2行目と3行目は実際のテーブルの属性(プロパティ)で、それぞれカラムのリテラル文字列を代入 (R2RMLでも使うURIテンプレート) 、である。`foaf:`は"Friend of a friend"、`xsd:`は"XML Schema Definition Language"という、どちらもパブリックに定義済みのオントロジー語彙である。

`source`は条件式やカラム指定のない、普通のSQLというだけである。つまりRDFのstudentリソースはstudentテーブル全件のこと、と言っているわけである。

これが交差表になるとどうなるか。

```
mappingId	uni1-teaching

target		:uni1/academic/{a_id}
            :teaches 
            :uni1/course/{c_id} . 

source		SELECT * FROM "uni1"."teaching"

```

こちらの`target`は、「先生は生徒を教える」トリプルひとつだけである。主語と目的語はURIテンプレートを使ったリテラルのリソースである。

`source`は交差表を全件抜く、というだけのSQLである。

最後の`fullProfessor`というのはサブクラスのマッピングである。`academic`テーブルは`position`属性でサブクラスに分割されているので、属性値の数だけマッピングを追加する。`position`属性の値と意味 (全部で5つある) はチュートリアルの`academic`テーブルの説明に書いてある。

```
mappingId	uni1-fullProfessor

target		:uni1/academic/{a_id} a :FullProfessor . 

source		SELECT * FROM "uni1"."academic"
		   	WHERE "position" = 1
```

`target`は`academic`マッピングの1行目の述語が違うだけである。`source`も`academic`マッピングの1行目と同じ、但しWHERE句が追加されている。

これなら少々属性が多くとも、素人でも力仕事で出来そうに見える、というか、AIにだって出来そう、などと思ってしまう。しかし我々が抱える本質的課題は「世の中はナゾの物理テーブルばかり」という「世界の現実」である、ということをを忘れてはならない。世の中がこんな単純なオントロジーと正規化されたテーブルばかりなら何の苦労もないのだ。

### SPARQLクエリを試す

サブクラス分割も含め全部のマッピングが入ったら、チュートリアルの指示を読んでクエリを投げてみよう。上手くいかないときはReasonerのオンオフを試してみるとよい。

SPARQLの内容は、「FullProfessorの姓は何?」である。

```sparql
SELECT DISTINCT ?prof ?lastName {
  ?prof a :FullProfessor ;
  foaf:lastName ?lastName .
}
```
こういう結果が返ってくる。

|prof|lastName|
|---|---|
|<http://example.org/voc#uni1/academic/1>|"Chambers"^^xsd:string|
|<http://example.org/voc#uni1/academic/12>|"Josephina"^^xsd:string|

なんだか余計な文字列が邪魔だが、これらはむしろRDFとして正しくURIやリテラルを返している、と捉えるべきである。不要であれば後続処理で消すなり、クエリに`(STR(?lName)`とか書き足せばよい。

このクエリ結果が表示されている"SPARQL results"の右に"SQL translation"というタブがあるので見てみよう。`SELECT`句から先、

```sql
SELECT V1."a_id" AS "<略1>", V1."last_name" AS "<略2>"
FROM "uni1"."academic" V1
WHERE 1 = V1."position"
```

この部分をコピーしてH2のクエリ欄に貼り付け実行すると「1番チェンバース先生」「12番ジョセフィーナ先生」がちゃんと表示される。

### "Inference" KICKS IN!

uni1チュートリアルのお終いは"inference (推論)"である。これは簡単に言えば「AがBでBがCならAもC」といった「三段論法」みたいなもので、文字列が似ているから意味が近い、とかではない。この三段論法をリアルタイムで処理するのがReasonerで、そのためにはオントロジーをインメモリで展開しておく必要がある。当然オントロジーが変わったらリロードが必要で、メニュー中にSynchronize reasonerがあるが不活性なので、代わりにするのがオンオフ操作である。

サンプルクエリの意味は「講師(`teacher`)一覧」である。

```sparql
PREFIX : <http://example.org/voc#>

SELECT DISTINCT ?teacher {
  ?teacher a :Teacher .
}
```

`teacher`クラス用の明示的なマッピングは書かれていない。しかしそのサブクラスにはいくつかのマッピングが存在する。そこでオントロジー側ではクエリ対象のスーパークラスからサブクラスを「推論」し、OntopがそれをSQLにマップする。

```SQL
SELECT DISTINCT V6."a_id0m12" AS "a_id0m12"
FROM ((SELECT V1."a_id" AS "a_id0m12"
FROM "uni1"."teaching" V1
)UNION ALL 
(SELECT V3."a_id" AS "a_id0m12"
FROM "uni1"."academic" V3, (VALUES  (1), (3), (2), (8)) AS V4 ("v0m0")
WHERE V3."position" = V4."v0m0"
)) V6

```

Protégéのクラスツリーを見るとわかる通り、マッピング9番のポスドクは`teacher`のサブクラスではないので、、`WHERE`句に9は含めない、というわけだ。もちろんSPARQLもSQLも12行で結果は同じ、違いはSPARQLはURI、SQLは数字のみという表現形式だけである。

これは「ビジネスメタをRDFで書く方法」の正解のひとつなのではないか。しかもテーブルやカラムが「見つかる」だけではなく、SQLを直接投げてくれるのである (マッピング - 「ナゾ解き」が出来れば、だが)。GenAIと組み合わせれば、雑な質問でも結果を直接答えてくれる。

そうするとお次はGenAIとの組み合わせで、アーキテクチャ的にどんな課題がありそうで、それにはどんなソリューションが要りそうか、というイメージも湧いてくる。次回はそれらを忘れないうちに整理しておきたい (uni2スキーマのチュートリアルがまだだが)。

(Part21につづく)
