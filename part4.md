## ビジネスメタをオントロジーで書く - Part 4

### エンティティと属性の概念を類型に分類する

前回ではエンティティと属性の「類型」をアノテーション・プロパティとしてオントロジーに追加した。次のステップははNorthwindのテーブルや属性がどの類型に当たるかを考える。

ただしこの段階では、分類の直接の対象のはテーブルや属性といった物理実体ではなく、その「概念」である。それらをアノテーション・プロパティの各類型のサブクラスとして定義する。いわゆるタクソノミーである。まずはテーブルを分類してみる。

```
  - Architype root
  |   - Resource
  |   |   - Employees Concept
  |   |   - Customers Concept
  |   |   - Shippers Concept
  |   |   - Suppliers Concept
  |       - Products Concept
  |   - Event
  |       - Orders Concept
  |   - Classification
  |   |   - Categories Concept
  |   |   - Region Concept
  |   |   - Territories Concept
  |       - Customer Demographics Concept
  |   - Rule
  |   - Summary
      - Association
      |   - Order Details Concept
      |   - Employee Territories Concept
          - Customer Customer Demo Concept
```

テーブルの分類はGeminiにやらせても一発で正解である。見ての通り、RuleとSummaryという類型に当たるテーブルはNorthwindにはなく、これらの類型には今回は出番がないことが分かる。

次に属性である。Old Schoolなデータモデリングのお作法では、これらの汎化された属性概念と型桁の組み合わせは「ドメイン」と呼ばれ、ERDエディタのUIで見たことがある、という人もいるかも知れない。逆に言えばERDエディタに触れる機会がないとこれを知ることもなく、ERDなんて「書かないのが当たり前」になった昨今では、ほとんど忘れ去られたプラクティスかもしれない (ドメインというワード自体は広く知られ、また用いられるようにはなったが)。そのいわば「失われたプラクティス」をメタデータ上で復活させるのである。

とりあえずGeminiにやってもらうと、ほとんどは正解だが、はずれもいくつか見受けられるのでそこは修正させ、以下の分類で進めることにする。Human in the loopである。

```
  - Semantic Type root
  |   - Identifier
  |   |   - ReportsTo Concept
  |   |   - CustomerID Concept
  |   |   - EmployeeID Concept
  |   |   - ShipVia Concept
  |   |   - SupplierID Concept
  |       - ProductID Concept
  |   - Sequence
  |       - OrderID Concept
  |   - Classifier
  |   |   - CategoryID Concept
  |   |   - RegionID Concept
  |   |   - TerritoryID Concept
  |       - CustomerTypeID Concept
  |   - Enum
  |   - Flag
  |       - Discontinued Concept
  |   - Name
  |   |   - LastName Concept
  |   |   - FirstName Concept
  |   |   - CategoryName Concept
  |   |   - CompanyName Concept
  |   |   - ContactName Concept
  |   |   - ShipName Concept
  |       - ProductName Concept
  |   - NumValue
  |   |   - Freight Concept
  |   |   - UnitPrice Concept
  |   |   - UnitsInStock Concept
  |   |   - UnitsOnOrder Concept
  |   |   - ReorderLevel Concept
  |   |   - Quantity Concept
  |       - Discount Concept
  |   - TimeStamp
  |   |   - BirthDate Concept
  |   |   - HireDate Concept
  |   |   - OrderDate Concept
  |   |   - RequiredDate Concept
  |       - ShippedDate Concept
  |   - Duration
  |   - ShortText
  |   |   - Title Concept
  |   |   - TitleOfCourtesy Concept
  |   |   - Extension Concept
  |   |   - PhotoPath Concept
  |   |   - ContactTitle Concept
  |       - QuantityPerUnit Concept
  |   - LongText
  |   |   - Notes Concept
  |   |   - Description Concept
  |   |   - RegionDescription Concept
  |   |   - TerritoryDescription Concept
  |       - CustomerDesc Concept
  |   - FormattedText
  |   |   - Phone Concept
  |   |   - Fax Concept
  |       - HomePage Concept
  |   - AddressComponent
  |   |   - Address Concept
  |   |   - City Concept
  |   |   - Region Concept
  |   |   - PostalCode Concept
  |   |   - Country Concept
  |   |   - ShipAddress Concept
  |   |   - ShipCity Concept
  |   |   - ShipRegion Concept
  |   |   - ShipPostalCode Concept
  |       - ShipCountry Concept
      - Binary
```

これらもテーブル同様Northwindでは出番がない類型もいくつか存在するのは見ての通りである。

テーブルも属性もそれらの「概念の定義」として、それぞれのスーパークラスの「汎化された」定義を引き継ぐ。つまり前もって用意された「スーパークラスの定義」に、「サブクラスの特徴」を加えればよいのである。

この定義をテーブルや属性側ではなく、アノテーション・プロパティ側で持つのがポイントなのではないだろうか。例えば`City`という属性は複数のテーブルに存在するが、その意味説明はアノテーション・プロパティに「自治体コード」などと一度書けば済む。一方DDL由来のクラスとプロパティの構造はそのまま保たれる。

前回作成したアノテーション・プロパティのスーパークラスのセットを含むオントロジーファイルに、上記のサブクラスを追記する。[出来たファイル](https://github.com/Yoshiyuki-iasa/northwind/blob/main/northwind_ontology_pt4.ttl)をProtégéで見比べてみると違いがわかる。

### Human in the loopの意義

前回までは生成AIが物理名称とスキーマ構造を手掛かりにオントロジー記述を進め、人間の介入はほぼなかったが、今回はそれぞれの概念 (の名前) の意味を、AIも支援こそすれ、人間が主体となって考えていく必要がある。しかしこれが役に立つのである。

定義を書いていくと、名前は違うが同じ意味、というケースにしばしば出くわすだろう。そういう場合は概念 (の名前) はひとつにすればよい。同音異義の排除はオントロジーの基本である (慣れれば同音異義そのものをオントロジーで定義することもできるが)。

上の例でいえば、例えば`ReportTo`という概念の取る値は`EmployeeID`だが、概念の意味 (セマンティック) は同じだろうか？どちらも「従業員番号です」としても間違いではないだろうが、`ReportsTo`では「上司の」が付く、つまり少し特化した概念である、という意味的な違いを表現することも可能である。

`ShipVia`はどうであろうか。この取り得る値は`Shippers`だが、「配達業者です」と定義を書いてみれば、どちらも同じで構わない、という結論になりえる。であれば、'ShipVia'という概念は余計であり、従いアノテーション・プロパティのタクソノミー階層から省いてしまえばよい、ということになる。

他にも例えば`Postal Code`はGeminiとしては`AddressComponent`であるという認識だが、人間としてはこれは`FormattedText`だ、と意義を唱えることも出来る。タクソノミーを決めたりするのは人間の役割である。

[(Part5に続く)](part5.md)
