## ビジネスメタをオントロジーで書く - Part 5

### アノテーション・プロパティを割り当てる

前回作成したアノテーション・プロパティは、今のところDDLから起こした「素の」オントロジーとは切り離された存在である。人間がなんとなく読む分にはそれでも良いかも知れバイが、機械的なテータ処理に繋げるためには明示的な関連付けが必要である。

テーブルについては、クラスとアノテーションプロパティは1:1なので、関連付けは簡単である。以下において、`nw:`はクラスそのもの、`ent:(= entity concepts)`はそれに紐付くアノテーション・プロパティ、`arc:`はそのスーパークラス (いわゆる「類型」) である・

*   `nw:Employees` -> `ent:Employees` (subPropertyOf `arc:Resource`)
*   `nw:Categories` -> `ent:Categories` (subPropertyOf `arc:Classification`)
*   `nw:Customers` -> `ent:Customers` (subPropertyOf `arc:Resource`)
*   `nw:Shippers` -> `ent:Shippers` (subPropertyOf `arc:Resource`)
*   `nw:Suppliers` -> `ent:Suppliers` (subPropertyOf `arc:Resource`)
*   `nw:Orders` -> `ent:Orders` (subPropertyOf `arc:Event`)
*   `nw:Order_Details` -> `ent:Order_Details` (subPropertyOf `arc:Association`)
*   `nw:Products` -> `ent:Products` (subPropertyOf `arc:Resource`)
*   `nw:Region` -> `ent:Region` (subPropertyOf `arc:Classification`)
*   `nw:Territories` -> `ent:Territories` (subPropertyOf `arc:Classification`)
*   `nw:EmployeeTerritories` -> `ent:EmployeeTerritories` (subPropertyOf `arc:Association`)
*   `nw:CustomerDemographics` -> `ent:CustomerDemographics` (subPropertyOf `arc:Classification`)
*   `nw:CustomerCustomerDemo` -> `ent:CustomerCustomerDemo` (subPropertyOf `arc:Association`)

続いてそれぞれのクラスの中身を確認し、アノテーション・プロパティとの紐づけが間違っていたら修正する。属性のアノテーション・プロパティの接頭辞は`ac:(= attribute concept)`である。

**Class: nw:Employees**
*   `nw:EmployeeID_` -> `ac:EmployeeID` (subPropertyOf `ac:ID`)
*   `nw:ReportsTo` -> `ac:ReportsTo` (subPropertyOf `ac:ID`)
*   `nw:LastName` -> `ac:LastName` (subPropertyOf `ac:Name`)
*   `nw:FirstName` -> `ac:FirstName` (subPropertyOf `ac:Name`)
*   `nw:Title` -> `ac:Title` (subPropertyOf `ac:ShortText`)
*   `nw:TitleOfCourtesy` -> `ac:TitleOfCourtesy` (subPropertyOf `ac:ShortText`)
*   `nw:BirthDate` -> `ac:BirthDate` (subPropertyOf `ac:TimeStamp`)
*   `nw:HireDate` -> `ac:HireDate` (subPropertyOf `ac:TimeStamp`)
*   `nw:Address` -> `ac:Address` (subPropertyOf `ac:AddressComponent`)
*   `nw:City` -> `ac:City` (subPropertyOf `ac:AddressComponent`)
*   `nw:Region` -> `ac:Region` (subPropertyOf `ac:AddressComponent`)
*   `nw:PostalCode` -> `ac:PostalCode` (subPropertyOf `ac:FormattedText`)
*   `nw:Country` -> `ac:Country` (subPropertyOf `ac:AddressComponent`)
*   `nw:HomePhone` -> `ac:Phone` (subPropertyOf `ac:FormattedText`)
*   `nw:Extension` -> `ac:Extension` (subPropertyOf `ac:ShortText`)
*   `nw:Notes` -> `ac:Notes` (subPropertyOf `ac:LongText`)
*   `nw:PhotoPath` -> `ac:PhotoPath` (subPropertyOf `ac:FormattedText`)

**Class: nw:Categories**
*   `nw:CategoryID_` -> `ac:CategoryID` (subPropertyOf `ac:Classifier`)
*   `nw:CategoryName` -> `ac:CategoryName` (subPropertyOf `ac:Name`)
*   `nw:Description` -> `ac:Description` (subPropertyOf `ac:LongText`)
*   `nw:Picture` -> `ac:Picture` (subPropertyOf `ac:Binary`)

**Class: nw:Customers**
*   `nw:CustomerID_` -> `ac:CustomerID` (subPropertyOf `ac:ID`)
*   `nw:CompanyName` -> `ac:CompanyName` (subPropertyOf `ac:Name`)
*   `nw:ContactName` -> `ac:ContactName` (subPropertyOf `ac:Name`)
*   `nw:ContactTitle` -> `ac:ContactTitle` (subPropertyOf `ac:ShortText`)
*   `nw:Address` -> `ac:Address` (subPropertyOf `ac:AddressComponent`)
*   `nw:City` -> `ac:City` (subPropertyOf `ac:AddressComponent`)
*   `nw:Region` -> `ac:Region` (subPropertyOf `ac:AddressComponent`)
*   `nw:PostalCode` -> `ac:PostalCode` (subPropertyOf `ac:FormattedText`)
*   `nw:Country` -> `ac:Country` (subPropertyOf `ac:AddressComponent`)
*   `nw:Phone` -> `ac:Phone` (subPropertyOf `ac:FormattedText`)
*   `nw:Fax` -> `ac:Fax` (subPropertyOf `ac:FormattedText`)

**Class: nw:Shippers**
*   `nw:ShipperID_` -> `ac:ShipperID` (subPropertyOf `ac:ID`)
*   `nw:CompanyName` -> `ac:CompanyName` (subPropertyOf `ac:Name`)
*   `nw:Phone` -> `ac:Phone` (subPropertyOf `ac:FormattedText`)

**Class: nw:Suppliers**
*   `nw:SupplierID_` -> `ac:SupplierID` (subPropertyOf `ac:ID`)
*   `nw:CompanyName` -> `ac:CompanyName` (subPropertyOf `ac:Name`)
*   `nw:ContactName` -> `ac:ContactName` (subPropertyOf `ac:Name`)
*   `nw:ContactTitle` -> `ac:ContactTitle` (subPropertyOf `ac:ShortText`)
*   `nw:Address` -> `ac:Address` (subPropertyOf `ac:AddressComponent`)
*   `nw:City` -> `ac:City` (subPropertyOf `ac:AddressComponent`)
*   `nw:Region` -> `ac:Region` (subPropertyOf `ac:AddressComponent`)
*   `nw:PostalCode` -> `ac:PostalCode` (subPropertyOf `ac:FormattedText`)
*   `nw:Country` -> `ac:Country` (subPropertyOf `ac:AddressComponent`)
*   `nw:Phone` -> `ac:Phone` (subPropertyOf `ac:FormattedText`)
*   `nw:Fax` -> `ac:Fax` (subPropertyOf `ac:FormattedText`)
*   `nw:HomePage` -> `ac:HomePage` (subPropertyOf `ac:FormattedText`)

**Class: nw:Orders**
*   `nw:OrderID_` -> `ac:OrderID` (subPropertyOf `ac:Sequence`)
*   `nw:CustomerID` -> `ac:CustomerID` (subPropertyOf `ac:ID`)
*   `nw:EmployeeID` -> `ac:EmployeeID` (subPropertyOf `ac:ID`)
*   `nw:ShipVia` -> `ac:ShipperID` (subPropertyOf `ac:ID`)
*   `nw:OrderDate` -> `ac:OrderDate` (subPropertyOf `ac:TimeStamp`)
*   `nw:RequiredDate` -> `ac:RequiredDate` (subPropertyOf `ac:TimeStamp`)
*   `nw:ShippedDate` -> `ac:ShippedDate` (subPropertyOf `ac:TimeStamp`)
*   `nw:Freight` -> `ac:Freight` (subPropertyOf `ac:NumValue`)
*   `nw:ShipName` -> `ac:ShipName` (subPropertyOf `ac:Name`)
*   `nw:ShipAddress` -> `ac:ShipAddress` (subPropertyOf `ac:AddressComponent`)
*   `nw:ShipCity` -> `ac:ShipCity` (subPropertyOf `ac:AddressComponent`)
*   `nw:ShipRegion` -> `ac:ShipRegion` (subPropertyOf `ac:AddressComponent`)
*   `nw:ShipPostalCode` -> `ac:ShipPostalCode` (subPropertyOf `ac:AddressComponent`)
*   `nw:ShipCountry` -> `ac:ShipCountry` (subPropertyOf `ac:AddressComponent`)

**Class: nw:Order_Details**
*   `nw:OrderID` -> `ac:OrderID` (subPropertyOf `ac:Sequence`)
*   `nw:ProductID` -> `ac:ProductID` (subPropertyOf `ac:ID`)
*   `nw:UnitPrice` -> `ac:UnitPrice` (subPropertyOf `ac:NumValue`)
*   `nw:Quantity` -> `ac:Quantity` (subPropertyOf `ac:NumValue`)
*   `nw:Discount` -> `ac:Discount` (subPropertyOf `ac:NumValue`)

**Class: nw:Products**
*   `nw:ProductID_` -> `ac:ProductID` (subPropertyOf `ac:ID`)
*   `nw:SupplierID` -> `ac:SupplierID` (subPropertyOf `ac:ID`)
*   `nw:CategoryID` -> `ac:CategoryID` (subPropertyOf `ac:Classifier`)
*   `nw:ProductName` -> `ac:ProductName` (subPropertyOf `ac:Name`)
*   `nw:QuantityPerUnit` -> `ac:QuantityPerUnit` (subPropertyOf `ac:ShortText`)
*   `nw:UnitPrice` -> `ac:UnitPrice` (subPropertyOf `ac:NumValue`)
*   `nw:UnitsInStock` -> `ac:UnitsInStock` (subPropertyOf `ac:NumValue`)
*   `nw:UnitsOnOrder` -> `ac:UnitsOnOrder` (subPropertyOf `ac:NumValue`)
*   `nw:ReorderLevel` -> `ac:ReorderLevel` (subPropertyOf `ac:NumValue`)
*   `nw:Discontinued` -> `ac:Discontinued` (subPropertyOf `ac:Flag`)

**Class: nw:Region**
*   `nw:RegionID_` -> `ac:RegionID` (subPropertyOf `ac:Classifier`)
*   `nw:RegionDescription` -> `ac:RegionDescription` (subPropertyOf `ac:LongText`)

**Class: nw:Territories**
*   `nw:TerritoryID_` -> `ac:TerritoryID` (subPropertyOf `ac:Classifier`)
*   `nw:RegionID` -> `ac:RegionID` (subPropertyOf `ac:Classifier`)
*   `nw:TerritoryDescription` -> `ac:TerritoryDescription` (subPropertyOf `ac:LongText`)

**Class: nw:EmployeeTerritories**
*   `nw:EmployeeID` -> `ac:EmployeeID` (subPropertyOf `ac:ID`)
*   `nw:TerritoryID` -> `ac:TerritoryID` (subPropertyOf `ac:Classifier`)

**Class: nw:CustomerDemographics**
*   `nw:CustomerTypeID_` -> `ac:CustomerTypeID` (subPropertyOf `ac:Classifier`)
*   `nw:CustomerDesc` -> `ac:CustomerDesc` (subPropertyOf `ac:LongText`)

**Class: nw:CustomerCustomerDemo**
*   `nw:CustomerID` -> `ac:CustomerID` (subPropertyOf `ac:ID`)
*   `nw:CustomerTypeID` -> `ac:CustomerTypeID` (subPropertyOf `ac:Classifier`)

[出来たオントロジーファイル](https://github.com/Yoshiyuki-iasa/northwind/blob/main/northwind_ontology_pt5.ttl)をProtégéで開いて、「素の」オントロジーと比較してみよう。元々のクラスやプロパティーはそのまま、アノテーション・プロパティのみが追加されているのが分かる。

今のところ属性ごとの意味説明は未記述 (それぞれのアノテーション・プロパティに`rdfs:comment`として書き加えることになる) だが、このくらいのファイルの読み込みなら、LLMにとっては全部で何百ページもあるような社内文書を読むよりはるかに楽だろうし、大抵の質問にもちゃんとした答えを返してくれるのではないだろうか。

しかし実際の企業ではテーブルの数はこんなものではない。オントロジーファイルもProtégéで実用的に読み書き出来るサイズでは済まないだろう。エンタープライズのスケールでのメタデータストアとしては、やはりグラフデータベースも考える必要があるのではないか。このオントロジーファイルをグラフデータベースにインポートできるか、試してみることにする。

(Part6に続く)