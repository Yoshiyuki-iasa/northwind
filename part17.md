## ビジネスメタをオントロジーで書く - Part 17

### aws Graph ExplorerをFusekiに繋ぐ

懸案だった「RDFのneo4j風visualization」だが、ようやく解決である。使うのは"[aws Graph Explorer](https://docs.aws.amazon.com/ja_jp/neptune/latest/userguide/visualization-graph-explorer.html)"だ。Docker Desktopさえ入っていれば、適当なフォルダに以下の`docker-compose.yml`を書いて`docker compose up -d`と叩くだけで、ひとつのコンテナからFusekiとGraph Explorerがいっぺんに起動する。停止や次回以降の起動はDocker DesktopのGUIからボタンひとつである。但しFusekiのデータはvolumeで外出しにしてはあるものの、コンテナを削除してしまうとデータは再ロードが必要なので要注意である。

```yaml
services:
  explorer:
    image: public.ecr.aws/neptune/graph-explorer:latest
    ports:
      - "8080:80"
    environment:
      - HOST=localhost
      - PROXY_SERVER_HTTPS_CONNECTION=false
      - GRAPH_EXP_HTTPS_CONNECTION=false

  fuseki:
    image: stain/jena-fuseki:latest
    ports:
      - "3030:3030"
    environment:
      - ADMIN_PASSWORD=pw123
    volumes:
      - ./fuseki-data:/fuseki/databases
volumes:
  fuseki-data:

```

FusekiのURLはで直接実行のときと同じ http://localhost:3030/ 。yamlに書いてあるパスワードのユーザ名は`admin`である。ブラウザUIの挙動が変なときはアドレスバーの左側でlocalhostのキャッシュをクリアすれば大体オーケーである。

Fusekiにデータセットをロードして (最新版では[default graphでもエラー除けに名前が要る](part9.md#fusekiを試す)、というバグは解消した模様)、サンプルSPARQLなどで値が返ることを確認したら、http://localhost:8080/explorer でGraph Explorerを開いて、SPARQLエンドポイントに `http://localhost:3030/{FusekiのDatabase名}/` を入れ、同期が成功すればば準備完了である。

とりあえずGraph Explorerの動作確認として、接続設定画面右上の"Open Graph Explorer"ボタンを押してグラフ表示画面に切り替える ("Open Connections"ボタンを押すと接続設定画面に戻る)。表示エリアの左上隅にある虫眼鏡 ("Search") ボタンを押して、表示された"Results"の右側にある"Add All"ボタンをとりあえず押してしまおう。グラフ表示画面にノードがいくつか現れるはずだ。各ノードがクリックで展開したりするのを試したら、まずはこれで動作確認まで終了である。

別のデータセットのためにコンテナを分けたいときは、別のフォルダにyamlファイルを置いてdocker composeを実行すれば別のコンテナが出来る。ポートは競合するので起動するコンテナはひとつだけにして、データセットはFusekiとGraph ExplorerのDB名 (=エンドポイント名) で他のコンテナと区別する。

### なぜいままでこの方法に気付けなかったのか!?

Graph ExplorerによるRDFの探求はまだこれからだが、少し触ってみた限りではneo4j DesktopのRDF版として申し分ない。どうやって作ったかと言うと、GenAIにちょっとお題を出したらyamlを書いてくれて、最初はイメージのpull先を間違えたりなど色々壁はあったものの、いったんyamlが出来上がってしまえば構築は拍子抜けするほど簡単である。

知っていれば最初からこうしていたのに、ネットを検索しても日本語では同様の事例がひとつもヒットしないのが不思議である。Protégéとセットで、オントロジーやRDFに興味がある人、情報技術系の学生などに「超お手軽なオントロジー/RDF/SPARQL学習環境」として大変お勧めなのではないか。

Graph Explorerは「SPARQLを書けなくてもRDFグラフを観察できる」とはいえ、これはこれで見たいノードの絞り込み、表示ラベルの設定やの色分けなどいろいろ覚えるべき操作は数多い。次回以降ゆるゆると進めていくことにしよう。

(Part18につづく)

