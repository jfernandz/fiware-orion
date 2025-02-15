# <a name="top"></a>NGSIv2 実装ノート (NGSIv2 Implementation Notes)

* [禁止されている文字](#forbidden-characters)
* [属性値の更新演算子](#update-operators-for-attribute-values)
* [メタデータ更新セマンティクス](#metadata-update-semantics)
* [通知のカスタムペイロードデコード](#custom-payload-decoding-on-notifications)
* [カスタム通知を無効にするオプション](#option-to-disable-custom-notifications)
* [カスタム通知の変更不可能なヘッダ](#non-modifiable-headers-in-custom-notifications)
* [カスタム通知でのヘッダの削除](#header-removal-in-custom-notifications)
* [エンティティ・ロケーションの属性に制限](#limit-to-attributes-for-entity-location)
* [`geo:json` 属性でサポートされる GeoJSON タイプ](#supported-geojson-types-in-geojson-attributes)
* [通知の従来の属性フォーマット](#legacy-attribute-format-in-notifications)
* [日時サポート](#datetime-support)
* [ユーザ属性または組み込み名前と一致するメタデータ](#user-attributes-or-metadata-matching-builtin-name)
* [サブスクリプション・ペイロードの検証](#subscription-payload-validations)
* [`actionType` メタデータ](#actiontype-metadata)
* [`ignoreType` メタデータ](#ignoretype-metadata)
* [`noAttrDetail` オプション](#noattrdetail-option)
* [通知スロットリング](#notification-throttling)
* [異なる属性型間の順序付け](#ordering-between-different-attribute-value-types)
* [Oneshot サブスクリプション](#oneshot-subscriptions)
* [ペイロードなしのカスタム通知](#custom-notifications-without-payload)
* [MQTT 通知](#mqtt-notifications)
* [変更された属性のみを通知](#notify-only-attributes-that-change)
* [`timeout` サブスクリプション・オプション](#timeout-subscriptions-option)
* [`lastFailureReason` および `lastSuccessCode` のサブスクリプション・フィールド](#lastfailurereason-and-lastsuccesscode-subscriptions-fields)
* [`failsCounter` と `maxFailsLimit` サブスクリプション・フィールド](#failscounter-and-maxfailslimit-subscriptions-fields)
* [`flowControl` オプション](#flowcontrol-option)
* [あいまいなサブスクリプション・ステータス `failed` は使用されない](#ambiguous-subscription-status-failed-not-used)
* [`forcedUpdate` オプション](#forcedupdate-option)
* [レジストレーション](#registrations)
* [`skipForwarding` オプション](#skipforwarding-option)
* [DateTime および geolocation タイプでの `null` サポート](#null-support-in-datetime-and-geolocation-types)
* [`POST /v2/op/notify` でサポートされない `keyValues`](#keyvalues-not-supported-in-post-v2opnotify)
* [廃止予定の機能](#deprecated-features)

このドキュメントでは、Orion Context Broker が [NGSI v2 仕様](http://telefonicaid.github.io/fiware-orion/api/v2/stable/)で行った具体的な実装について考慮する必要があるいくつかの考慮事項について説明します。

<a name="forbidden-characters"></a>
## 禁止されている文字

NGSIv2 仕様の "フィールド構文の制限" セクションから :

> 上記のルールに加えて、NGSIv2 サーバの実装では、クロス・スクリプト注入攻撃を避けるために、それらのフィールドまたは他のフィールドに構文上の制限を追加することができます。

Orion に適用される追加の制限事項は、マニュアルの [禁止されている文字](forbidden_characters.md)のセクションに記載されているものです。

属性値で禁止文字のチェックをスキップするために、"TextUnrestricted" 属性タイプ (および
NGSIv2 仕様で定義されているもの以外の特別な属性タイプ) を使用できることに注意してください。
ただし、セキュリティ上の問題 (スクリプト・インジェクション攻撃の可能性) がある可能性がある
ため、自己責任で使用してください!

[トップ](#top)

<a name="update-operators-for-attribute-values"></a>

## 属性値の更新演算子

一部の属性値の更新には、NGSIv2 仕様で説明されているもの以外に特別なセマンティクスが
あります。特に、このようなリクエストを行うことができます:

```
POST /v2/entities/E/attrs/A
{
  "value": { "$inc": 3 },
  "type": "Number"
}
```

これは、*"属性Aの値を3増やす"* ことを意味します。

この機能は、アプリケーションの複雑さを軽減し、同じコンテキストに同時にアクセスする
アプリケーションの競合状態を回避するために役立ちます。
詳細は[このドキュメント](update_operators.md)を参照してください。

[トップ](#top)

<a name="metadata-update-semantics"></a>
## メタデータ更新セマンティクス

Orion Context Broker (および関連する `overrideMetadata` オプション)
で使用されるメタデータ更新セマンティクスについては、
[ドキュメントのこのセクション](metadata.md#updating-metadata)
で詳しく説明しています。

さらに、NGSIv2 仕様セクション "Partial Representations" (部分表現) から:

> 属性 `metadata` はリクエストで省略できます。つまり、属性に関連付けられた
> メタデータ要素はありません。
`overrideMetadata` が使用されているかどうかに応じて、この文には2つの解釈があります:

* `overrideMetadata` が使用されていない場合 (デフォルトの動作)、"...属性に
  関連付けられたメタデータ要素がないことを意味します。**更新する必要があります**"
  と解釈されます
* `overrideMetadata` が使用されている場合、"...属性に関連付けられたメタデータ要素が
  ないことを意味します。**属性の更新の結果として**" と解釈されます

[トップ](#top)

<a name="custom-payload-decoding-on-notifications"></a>
## 通知でのカスタムペイロードデコード

禁止された文字の制限のために、Orion は発信カスタム通知に追加のデコード・ステップを適用します。これについては、このマニュアルの [このセクション](forbidden_characters.md#custom-payload-special-treatment)で詳しく説明します。

[トップ](#top)

<a name="option-to-disable-custom-notifications"></a>
## カスタム通知を無効にするオプション

Orion は、`-disableCustomNotifications` [CLI パラメータ](../admin/cli.md)を使用してカスタム通知を無効にするように設定できます。

この場合 :

* `httpCustom` が `http` として解釈されます。すなわち、`url` を除くすべてのサブフィールドは無視されます。
* マクロ置換 `${...}` は実行されません。

[トップ](#top)

<a name="non-modifiable-headers-in-custom-notifications"></a>
## カスタム・通知の変更不可能なヘッダ

カスタム・通知で次のヘッダを上書きすることはできません :

* `Fiware-Correlator`
* `Ngsiv2-AttrsFormat`

そのような試み (例えば `"httpCustom": { ... "headers": {"Fiware-Correlator": "foo"} ...}`) は無視されます。

[トップ](#top)

<a name="header-removal-in-custom-notifications"></a>
## カスタム通知でのヘッダの削除

NGSIv2仕様 ("カスタム通知" セクション) で明示的に述べられていませんが、`headers` オブジェクトの
ヘッダ・キーの文字列値が空の場合、そのヘッダは通知から削除されます。たとえば、次の構成:

```
"httpCustom": {
   ...
   "headers": {"x-auth-token": ""}
}
```

サブスクリプションに関連付けられた通知の `x-auth-token` ヘッダが削除されます。

これは、Orion が通知に自動的に含めるヘッダを削除するのに役立ちます。例えば:

* 通知にデフォルトで含まれるヘッダを回避するため (例: `Accept`)
* 前述の `x-auth-token` などの (更新から通知までの) ヘッダの伝播を削減するため

[トップ](#top)

<a name="limit-to-attributes-for-entity-location"></a>
## エンティティ・ロケーションの属性に制限

NGSIv2 仕様の "エンティティの地理空間プロパティ" のセクションから :

> クライアントアプリケーションは、(適切な NGSI アトリビュート型を提供することによって) ジオスペース・プロパティを伝えるエンティティ属性を定義する責任があります。通常これは `location` という名前のついたエンティティ属性ですが、エンティティに複数の地理空間属性が含まれているユース・ケースはありません。たとえば、異なる粒度レベルで指定された場所、または異なる精度で異なる場所の方法によって提供された場所です。それにもかかわらず、空間特性には、バックエンド・データベースによって課せられたリソースの制約下にある特別なインデックスが必要であることは注目に値します。したがって、実装では、空間インデックスの制限を超えるとエラーが発生する可能性があります。これらの状況で推奨される HTTP ステータス・コードは `413` です。リクエスト・エンティティが大きすぎます。また、レスポンス・ペイロードで報告されたエラーは、`NoResourcesAvailable` である必要があります。

Orion の場合、その制限は1つの属性です。

ただし、`ignoreType` メタデータを `true` に設定して、特定の属性に追加の有益なロケーションが含まれるように
することができます (詳細については、[ドキュメントのこのセクション](#ignoretype-metadata)を参照してください。
これにより、Orion によるその属性のロケーションとしての解釈が無効になるため、制限にはカウントされません。

例えば:

```
{
  "id": "Hospital1",
  "type": "Hospital",
  ...
  "location": {
    "value": {
      "type": "Point",
      "coordinates": [ -3.68666, 40.48108 ]
    },
    "type": "geo:json"
  },
  "serviceArea": {
    "value": {
      "type": "Polygon",
      "coordinates": [ [ [-3.69807, 40.49029 ], [ -3.68640, 40.49100], [-3.68602, 40.50456], [-3.71192, 40.50420], [-3.69807, 40.49029 ] ] ]
    },
    "type": "geo:json",
    "metadata": {
      "ignoreType":{
        "value": true,
        "type": "Boolean"
      }
    }
  }
}
```

どちらの属性もタイプ `geo:json` ですが、`serviceArea` は `ignoreType` メタデータを `true` に使用するため、
情報を提供しない1つのロケーションの制限を超えません。

この方法で追加ロケーションを定義する場合は、ジオクエリの解決に使用されるロケーションが、 `ignoreType` が
`true` メタデータ (上記の例では `location` 属性) に設定されていない場所であることを考慮してください。
`ignoreType` を `true` に設定して定義されたすべてのロケーションは、Orion によって無視され、この意味で、
ジオクエリには加わりません。

[トップ](#top)

<a name="supported-geojson-types-in-geojson-attributes"></a>

## `geo:json` 属性でサポートされる GeoJSON タイプ

NGSIv2 仕様では、`geo:json` 属性に使用される可能性のある GeoJSON タイプに制限を設定していません。
ただし、現在の Orion の実装 (MongoDB の機能に基づく) にはいくつかの制限があります。

次のタイプのテストに成功しました :

* Point
* MultiPoint
* LineString
* MultiLineString
* Polygon
* MultiPolygon

その一方で、次のタイプは機能しません (使用しようとすると "Database Error" が発生します) :

* Feature
* GeometryCollection
* FeatureCollection

実施されたテストの詳細については、
[こちら](https://github.com/telefonicaid/fiware-orion/issues/3586)をご覧ください。

[トップ](#top)

<a name="legacy-attribute-format-in-notifications"></a>
## 通知の従来の属性フォーマット

NGSIv2 仕様の `attrsFormat` で述べている値とは別に、Orion は、NGSIv1 形式の通知を送信するために、`legacy` 値もサポートしています。このようにして、ユーザは NGSIv1 レガシー通知レシーバを使用した NGSIv2 サブスクリプション (フィルタリングなど) の拡張の恩恵を受けることができます。

NGSIv1 は非推奨であることに注意してください。したがって、`legacy` 通知形式をもう使用しないことを推奨します。

[トップ](#top)

<a name="datetime-support"></a>
## 日時のサポート

NGSIv2 仕様の "Special Attribute Types" セクションから :

> DateTime : 日付を ISO8601 形式で識別します。これらの属性は、より大きい、より小さい、より大きい、等しい、より小さい、等しいおよび範囲のクエリ演算子で使用できます。

次の考慮事項は、属性の作成/更新時間または `q` と `mq` フィルタで使用される時に考慮しなければならない :

* Datetimes は、date, time, timezone の指定子で構成され、次のいずれかのパターンで表されます :
    * `<date>`
    * `<date>T<time>`
    * `<date>T<time><timezone>`
    * フォーマット `<date><timezone>` は許可されていないことに注意してください。ISO8601 によると : *"タイムゾーン指定子が必要な場合、それは結合された日付と時刻に従います"*
* `<date>` ついては、それがパターンに従わなければならない : `YYYY-MM-DD`
    * `YYYY`: year (4桁)
    * `MM`: month (2桁)
    * `DD`: day (2桁)
* これについて `<time>` は、[ISO8601 仕様](https://en.wikipedia.org/wiki/ISO_8601#Times)に記述されているパターンのいずれかに従わなければならない :
    * `hh:mm:ss.sss` または `hhmmss.sss`
    * `hh:mm:ss` または `hhmmss`。この場合、ミリ秒は `000` に設定されます
    * ``hh:mm` または `hhmm`。この場合、秒は `00` に設定されます
    * `hh`。この場合、分と秒は `00` に設定されます
    * もし `<time>` 省略された場合、時、分、秒が `00` に設定されます
* `<timezones>` については、[ISO8601 仕様](https://en.wikipedia.org/wiki/ISO_8601#Time_zone_designators)に記述されているパターンのいずれかに従わなければならない :
    * `Z`
    * `±hh:mm`
    * `±hhmm`
    * `±hh`
* ISO8601 は、*" 時間表現で UTC 関係情報が与えられていない場合、その時間は現地時間であると想定される "* と規定している。ただし、クライアントとサーバが異なるゾーンにある場合、これはあいまいです。したがって、この曖昧さを解決するために、時間帯指定子が省略されている場合、Orion は常にタイムゾーン `Z` を想定します

Orion は常に、`YYYY-MM-DDThh:mm:ss.sssZ` という形式を使用して日時の属性/メタデータを提供します。ただし、Orion は
`YYYY-MM-DDThh:mm:ss.ssZ` 形式を使用して他のタイムスタンプ (レジストレーション/サブスクリプションの有効期限、
最後の通知/失敗/成功など) を提供することに注意してください
(これについては、[関連する問題](https://github.com/telefonicaid/fiware-orion/issues/3671)を参照してください)。

さらに、Orion は日付時刻を提供するときに常に UTC/Zulu タイムゾーンを使用することに注意してください (クライアント/
レシーバーが任意のタイムゾーンで実行されている可能性があるため、これは最良のデフォルト・オプションです)
これは将来変更される可能性があります ([関連する問題](https://github.com/telefonicaid/fiware-orion/issues/2663)を
参照してください)。

属性とメタデータの型としての文字列 "ISO8601" もサポートされています。この効果は、"DateTime" を使用した場合と同じです。

[トップ](#top)

<a name="user-attributes-or-metadata-matching-builtin-name"></a>
## ユーザ属性または組み込み名前と一致するメタデータ

(このセクションの内容は、`dateExpires` 属性を除くすべての組み込み関数に適用されます。`dateExpires` に関する特定の情報については、[一時的なエンティティ](transient_entities.md)のドキュメントを参照してください)。

まず、**NGSIv2 組み込み属性やメタデータと同じ名前のものを使用しないことを強く推奨します**。実際、NGSIv2 仕様では、これを禁止しています。仕様の "属性名の制限" と "メタデータ名の制限" のセクションを参照してください。

しかし、もし、あなたがそのような属性やメタデータを (おそらくレガシーの理由により) 持っていたとしても、以下の点を考慮に入れてください :

* NGSIv2 組み込みの属性および/またはメタデータと同じ名前のものを作成/更新することができます。Orion は、そうするでしょう
* ユーザ定義の属性および/またはメタデータは、GET リクエストまたはサブスクリプションで明示的に宣言する必要なく表示されます。 たとえば、エンティティ E1 に値 "2050-01-01" を持つ、`dateModified` 属性を作成した場合、`GET /v2/entities/E1` がそれを取得します。`?attrs=dateModified` を使う必要はありません。
* (GET オペレーションまたは通知にレスポンスして) レンダリングされると、明示的に宣言されていても、ユーザ定義の属性/メタデータが組み込み関数より優先されます。たとえば、エンティティ E1 に値 "2050-01-01" を持つ `dateModified` 属性を作成し、`GET /v2/entities?attrs=dateModified` をリクエストすると、"2050-01-01" が得られます。
* しかし、フィルタリング (すなわち `q` または `mq`) は組み込み関数の値に基づいています。 たとえば、エンティティ E1 に値 "2050-01-01" を持つ `dateModified` 属性を作成し、`GET /v2/entities?q=dateModified>2049-12-31` をリクエストした場合、エンティティは取得されません。"2050-01-01" は "2049-12-31" よりも大きいですが、エンティティを変更した日付 (2018年か2019年のいずれかの日付) は "2049-12-31" より大きくなりません。これは何らかの形で矛盾していることに注意してください (つまり、ユーザ定義はレンダリングでは優先されますがフィルタリングでは優先されません)、将来変更される可能性があります。

[トップ](#top)

<a name="subscription-payload-validations"></a>
## サブスクリプション・ペイロードの検証

Orion が NGSIv2 サブスクリプション・ペイロードで実装する特定の検証は、次のとおりです :

* **description**: オプション (最大長1024)
* **subject**: 必須
    * **entities**: 必須
        * **id** or **idPattern**: そのうちの1つは必須です (ただし、同時に両方は許可されません)。id は ID の NGSIv2 制限に従わなければなりません。idPattern は空ではなく、有効な正規表現でなければなりません
        * **type** or **typePattern**: 任意です (ただし、両方同時に許可されません)。type は ID の NGSIv2 制限に従う必要があります。型は空であってはいけません。typePattern は有効な正規表現で、空ではない必要があります
    * **condition**: オプション (但し、存在する場合は内容を持たなければなりません。つまり `{}` は許可されていません)
        * **attrs**: オプション (ただし、存在する場合はリストでなければなりません。空リストも許されます)
        * **expression**: オプション (ただし、存在する場合は内容を持たなければなりません。つまり {} は許可されていません)
            * **q**: オプション (ただし、存在する場合は空でなければなりません。つまり `""` は許可されていません)
            * **mq**: オプション (ただし、存在する場合は空でなければなりません。つまり `""` は許可されていません)
            * **georel**: オプション (ただし、存在する場合は空でなければなりません。つまり `""` は許可されていません)
            * **geometry**: オプション (ただし、存在する場合は空でなければなりません。つまり `""` は許可されていません)
            * **coords**: オプション (ただし、存在する場合は空でなければなりません。つまり `""` 許可されていません)
* **notification**:
    * **http**: `httpCustom` が省略された場合は存在しなければならず、そうでなければ禁止されています
        * **url**: 必須 (有効な URL である必要があります)
    * **httpCustom**: `http` が省略されている場合は存在し、そうでない場合は禁止されていなければなりません
        * **url**: 必須 (空でなければなりません)
        * **headers**: オプション (ただし、存在する場合はコンテンツが必要です。つまり `{}` は許可されていません)
        * **qs**: オプション (ただし、存在する場合はコンテンツが必要です。つまり `{}` は許可されていません)
        * **method**: オプション (存在する場合は有効な HTTP メソッドでなければなりません)
        * **payload**: オプション (空の文字列を使用できます)
    * **attrs**: オプション (ただし、存在する場合はリストでなければなりません。空リストも許可されます)
    * **metadata**: オプション (ただし、存在する場合はリストでなければなりません。空リストも許可されます)
    * **exceptAttrs**: オプションです (ただし、`attrs` も使用されている場合は存在できません。存在する場合は空でないリストでなければなりません)
    * **attrsFormat**: オプション (存在する場合は有効な `attrs` 形式のキーワードでなければなりません)
* **throttling**: オプション (整数でなければなりません)
* **expires**: オプション (日付または空の文字列 "" でなければなりません)
* **status**: オプション (有効なステータス・キーワードである必要があります)

[トップ](#top)

<a name="actiontype-metadata"></a>
## `actionType` メタデータ

NGSIv2 仕様のセクション "組み込みメタデータ" から `actionType` メタデータ関連 :

> その値はリクエストオペレーションの型によって異なります : 更新のための `update`, 作成のための `append`, 削除のための `delete`。その型は常に Text です。

現在の Orion の実装では、"update (更新)" と "append (追加)" がサポートされています。[この問題](https://github.com/telefonicaid/fiware-orion/issues/1494)が完了すると、"delete (削除)" のケースがサポートされます。

[トップ](#top)

<a name="ignoretype-metadata"></a>
## `ignoreType` メタデータ

NGSIv2 仕様の "Builtin metadata" セクションで説明されているメタデータから、Orion は `ignoreType`
メタデータを実装します。

値が `true` の `ignoreType` が属性に追加されると、Orion は属性タイプに関連付けられたセマンティクスを
無視します。Orion は一般に属性タイプを無視するため、このメタデータはほとんどの場合必要ありませんが、
属性タイプに Orion の特別なセマンティクスがある2つのケースがあります (詳細については NGSIv2 仕様を
確認してください):

* `DateTime`
* Geo-location タイプ (`geo:point`, `geo:line`, `geo:box`, `geo:polygon` and `geo:json`)

現時点では、`ignoreType` は Geo-location タイプでのみサポートされているため、メカニズムは
エンティティごとに1つのジオロケーションのみの制限を克服できます (詳細については、
[ドキュメントのこのセクション](#limit-to-attributes-for-entity-location)を参照)。`DateTime`
での `ignoreType` のサポートは将来的に行われる可能性があります。

[トップ](#top)

<a name="noattrdetail-option"></a>
## `noAttrDetail` オプション

URI param `options` の値 `noAttrDetail` は、NGSIv2 型のブラウジング・クエリ (`GET /v2/types` および `GET /v2/types/<type>`) が、属性型の詳細を提供しないようにするために使用されます。使用すると、各属性名に関連付けられた `types` リストが `[]` に設定されます。

このオプションを使用すると、Orion はこれらのクエリをはるかに迅速に解決します。特に、それぞれが異なる型の多数の属性の場合は、これは、ユース・ケースに属性型の詳細が必要ない場合に非常に便利です。場合によっては、`noAttrDetails` オプションで30秒から 0.5 秒の節約が検出されました。

[トップ](#top)

<a name="notification-throttling"></a>
## 通知スロットリング

サブスクリプション・スロットリングに関する NGSIv2 仕様から :

> throttling : 2つの連続した通知の間に経過する必要のある秒単位の最小限の時間。オプションです。

Orion がこれを実装する方法は、保護期間のスロットル中に通知を破棄することです。したがって、あまりにも近づいてしまえば、通知が失われる可能性があります。ユース・ケースがこのように通知を失うことをサポートしていない場合は、スロットリングを使用しないでください。

さらに、Orion はローカルでスロットリングを実装します。multi-CB 構成では、最終通知の尺度が各 Orion ノードに対してローカルであることを考慮してください。各ノードは DB と定期的に同期してより新しい値を取得しますが ([ここ](../admin/perf_tuning.md#subscription-cache)ではこれ以上)、特定のノードに古い値があることがありますので、スロットリングは100％正確ではありません。

[トップ](#top)

<a name="ordering-between-different-attribute-value-types"></a>
## 異なる属性値型間の順序付け

NGISv2 仕様 "Ordering Results" セクションから :

> エンティティのリストを取得するオペレーションでは、順序付けの結果を得る際に条件として使用される属性またはプロパティを `orderBy` URI パラメータで指定できます

これは、各型が他の型に関してどのように順序づけられるかの実装アスペクトです。Orion の場合、基本的な実装(MongoDB)で使用されているものと同じ基準を使用します。詳細については、[次のリンク](https://docs.mongodb.com/manual/reference/method/cursor.sort/#ascending-descending-sort)を参照してください。

最低から最高まで :

1. Null
2. Number
3. String
4. Object
5. Array
6. Boolean

[Top](#top)

<a name="oneshot-subscriptions"></a>
## Oneshot サブスクリプション

Orionは、NGSIv2 仕様のサブスクリプション用に定義された `status` 値の他に、
`oneshot`を使用することもできます。
[Oneshot サブスクリプションのドキュメント](oneshot_subscription.md)で詳細を確認してください

[トップ](#top)

<a name="custom-notifications-without-payload"></a>
## ペイロードなしのカスタム通知

カスタム通知の `httpCustom` フィールド内で `payload` が `null` に設定されている場合、そのサブスクリプションに
関連付けられた通知にはペイロードが含まれません (つまり、conten-length 0 の通知)。

これは、`payload` を `""` に設定したり、フィールドを省略したりすることと同じではないことに注意してください。
その場合、通知は NGSIv2 正規化形式を使用して送信されます。

[トップ](#top)

<a name="mqtt-notificationsmqtt-notificationsmqtt-notifications"></a>
## MQTT 通知

NGISv2 仕様で説明されているサブスクリプションの `notification` オブジェクト内の `http` および
`httpCustom` フィールドとは別に、Orion は MQTT 通知用の `mqtt` および `mqttCustom` もサポートして
います。このトピックは、[この特定のドキュメント](mqtt_notifications.md)でより詳細に説明されています。

[トップ](#top)

<a name="notify-only-attributes-that-change"></a>
## 変更された属性のみを通知

Orion は、NGSIv2 仕様で説明されているものとは別に、サブスクリプションの中で追加のフィールド `onlyChangedAttrs`
(`notification` 内に) をサポートしています。 このフィールドは `true` または `false` の値を取ります
(フィールドが省略された場合、デフォルトは `false` です)。 `true` に設定されている場合、サブスクリプションに
関連した通知は `attrs` または `exceptAttrs` フィールドと組み合わせて、トリガーしている更新リクエストで
変更された属性のみを含みます。

例えば、`attrs` が `[A、B、C]` のデフォルトの振る舞い (`onlyChangedAttrs` が `false` の場合) とトリガー更新が
A のみを修正した場合、A, B, C が通知されます(つまり、トリガー更新は関係ありません)。 しかし、`onlyChangedAttrs`
が `true` でトリガー更新が A のみを修正した場合、通知には A のみが含まれます。

[トップ](#top)

<a name="timeout-subscriptions-option"></a>
## `timeout` サブスクリプション・オプション

`GET/v2/subscriptions` および `GET/v2/subsets/subId` リクエストの NGSIv2 仕様で説明されているサブスクリプションフィールドとは別に、Orion は `http` または `httpCustom` フィールド内の `timeout` 追加パラメータをサポートします。このフィールドは、HTTP 通知を使用するときにサブスクリプションがレスポンスを待機する最大時間をミリ秒単位で指定します。

このパラメータに許可される最大値は1800000 (30分) です。`timeout` が0に設定されているか省略されている場合、`-httpTimeout` CLI パラメータとして渡された値が使用されます。詳細については、[コマンドライン・オプション](../admin/cli.md#command-line-options)のセクションを参照してください。

[トップ](#top)

<a name="lastfailurereason-and-lastsuccesscode-subscriptions-fields"></a>
## `lastFailureReason` および `lastSuccessCode` のサブスクリプション・フィールド

`GET /v2/subscriptions` および `GET /v2/subscriptions/subId` リクエストの
NGSIv2 仕様で説明されているサブスクリプション・フィールドとは別に、
Orion は通知フィールド内でこの2つの追加フィールドをサポートしています  :

* `lastFailureReason`: 最後の失敗の原因を記述するテキスト文字列
  (すなわち、失敗が `lastFailure` の時点で発生した)
* `lastSuccessCode`: 最後に成功した通知が送信されたときに
  受信エンドポイントによって返された HTTP コード (200, 400, 404, 500 など)
  (つまり、成功が `lastSuccess` の時点で発生した)

どちらも通知に関する問題の分析に使用できます。 詳しくは、
[問題診断ドキュメント](../admin/diagnosis.md#diagnose-notification-reception-problems)
のセクションを参照してください。

これらの2つのフィールドは HTTP サブスクリプションに含まれていますが、MQTT サブスクリプションには
含まれていないことに注意してください。詳細については、[MQTT 通知ドキュメント](#mqtt_notifications.md)
を参照してください。

[トップ](#top)

<a name="failscounter-and-maxfailslimit-subscriptions-fields"></a>
## `failsCounter` と `maxFailsLimit` サブスクリプション・フィールド

`GET /v2/subscriptions` および `GET /v2/subscriptions/subId` リクエストの NGSIv2 仕様で説明されている
サブスクリプション・フィールドとは別に、Orion は `notification` フィールド内の `failsCounter` フィールドを
サポートします。このフィールドの値は、サブスクリプションに関連付けられた連続して失敗した通知の数です。
`failsCounter` は、通知の試行が失敗するたびに1ずつ増加し、通知の試行が成功すると0にリセットされます
(この場合、`failsCounter` は省略されます)。

連続した失敗の最大許容数を確立するオプションのフィールド `maxFailsLimit` (これも `notification` フィールド内)
もあります。失敗の数が `maxFailsLimit` の値を超えた場合 (つまり、特定の瞬間に `failsCounter` が `maxFailsLimit`
より大きい場合)、Orion はサブスクリプションを自動的に `inactive` 状態に渡します。サブスクリプションを再度有効に
するには (状態を再び `active` に設定)、サブスクリプションの更新操作 (`PATCH /v2/subscription/subId`) が必要です。

さらに、Orion がサブスクリプションを自動的に無効にすると、WARN レベルのログトレースがこの形式で出力されます:

```
time=... | lvl=WARN | corr=... | trans=... | from=... | srv=... | subsrv=... | comp=Orion | op=... | msg= Subscription <subId> automatically disabled due to failsCounter (N) overpasses maxFailsLimit (M)
```

[Top](#top)

[トップ](#top)

<a name="flowcontrol-option"></a>
## `flowControl` オプション

これにより、更新操作でフロー制御を使用する必要があることを指定できます。これにより、パフォーマンスが
向上し、高負荷シナリオでの飽和を回避できます。これは、[`-notifFlowControl` パラメータ](../admin/cli.md)
を使用して ContextBroker が開始されている場合にのみ機能し、そうでない場合は無視されます。
フロー制御メカニズムは、[ドキュメントのこのセクション](../admin/perf_tuning.md#updates-flow-control-mechanism)
で説明しています。

次のリクエストでは、flowControl URI param オプションを使用できます :

* `POST /v2/entities/E/attrs?options=flowControl`
* `POST /v2/entities/E/attrs?options=append,flowControl`
* `POST /v2/op/update?options=flowControl`
* `PUT /v2/entities/E/attrs?options=flowControl`
* `PUT /v2/entities/E/attrs/A?options=flowControl`
* `PUT /v2/entities/E/attrs/A/value?options=flowControl`
* `PATCH /v2/entities/E/attrs?options=flowControl`

[トップ](#top)

<a name="ambiguous-subscription-status-failed-not-used"></a>
## あいまいなサブスクリプション・ステータス `failed` は使用されない

NGSIv2 仕様では、サブスクリプションの `status` フィールドの `failed` 値について説明しています:

> `status`: [...] また、通知で問題が発生しているサブスクリプションの場合、ステータスは `failed` に
> 設定されます。通知が再び機能し始めるとすぐに、ステータスは `active` に戻ります。

ステータス `failed` は、あいまいなため、Orion3.4.0 で削除されました。

* `failed` は、最後に送信された通知が失敗したアクティブなサブスクリプション (つまり、エンティティの
  更新時に通知をトリガーするサブスクリプション) を指す場合があります
* `failed` は、過去にアクティブであり、アクティブなときに送信された最後の通知が失敗した非アクティブな
  サブスクリプション (つまり、エンティティの更新時に通知をトリガーしないサブスクリプション)
  を指す場合があります

つまり、ステータス `failed` を確認しても、サブスクリプションが現在アクティブであるか非アクティブで
あるかを知ることはできません。

したがって、`failed` は Orion Context Broker によって使用されず、サブスクリプションのステータスは、
サブスクリプションが `active` (`failed` バリアント [`oneshot`](#oneshot-subscriptions) を含む) か
`inactive` (バリアント `expired` を含む）かを常に明確に指定します。サブスクリプションが最後の通知で
失敗したかどうかを知るために、`failsCounter` の値をチェックできます (つまり、`failedCounter` が
0より大きいことをチェックします)。

[トップ](#top)

<a name="forcedupdate-option"></a>
## `forcedUpdate` オプション

NGSIv2 仕様に含まれるものに対する追加の URI パラメータ・オプションとして、Orion は forcedUpdate
を実装します。実際の属性の更新があってもなくても、更新オペレーションが一致するサブスクリプションを
トリガして、対応する通知を送信する必要があることを指定するために使用できます。デフォルトの動作
(つまり、forcedUpdate URI param オプションを使用しない場合) は、属性が実際に更新された場合にのみ
更新されることに注意してください。

次のリクエストでは、forcedUpdate URI param オプションを使用できます :

* `POST /v2/entities/E/attrs?options=forcedUpdate`
* `POST /v2/entities/E/attrs?options=append,forcedUpdate`
* `POST /v2/op/update?options=forcedUpdate`
* `PUT /v2/entities/E/attrs?options=forcedUpdate`
* `PUT /v2/entities/E/attrs/A?options=forcedUpdate`
* `PUT /v2/entities/E/attrs/A/value?options=forcedUpdate`
* `PATCH /v2/entities/E/attrs?options=forcedUpdate`

[Top](#top)

<a name="registrations"></a>
## レジストレーション

Orion は、次の点を除いて、NGSIv2 仕様に記載されているレジストレーション管理を実装しています。

* `PATCH /v2/registration/<id>` は実装されていません。したがって、レジストレーションを直接更新することはできません。つまり、レジストレーションを削除して再作成する必要があります。[この issue](https://github.com/telefonicaid/fiware-orion/issues/3007)についてはこちらをご覧ください
* `idPattern` はサポートされています
* `typePattern` は実装されていません
* `dataProvided` 内での `expression` フィールドはサポートされていません。フィールドは単に無視されます。これについては [この issue](https://github.com/telefonicaid/fiware-orion/issues/3107) を見てください。
* `status` での `inactive` 値はサポートされていません。つまり、フィールドは正しく格納され/取得されますが、値が `inactive` の場合でもレジストレーションは常にアクティブです。これについては、[この issue](https://github.com/telefonicaid/fiware-orion/issues/3108) を見てください

NGSIv2 仕様によると :

> NGSIv2 サーバ実装は、コンテキスト情報ソースへのクエリまたは更新転送を実装することができます

Orion がこのような転送を実装する方法は次のとおりです :

* `POST /v2/op/query` クエリ転送のため
* `POST /v2/op/update` 更新転送のため

コンテキスト情報ソースへの転送に関するより多くの情報は、この[ドキュメント](context_providers.md)にあります。

Orion は NGSIv2 仕様に含まれていない追加フィールド (`provider` 内の) `legacyForwarding`を実装しています。`legacyForwarding` の値が `true` の場合、NGSIv1 ベースのクエリ/更新はそのレジストレーションに関連したリクエストを転送するために使用されます。NGSIv1 は廃止予定ですが、一部のコンテキスト・プロバイダはまだ NGSIv2 に移行されていない可能性があるため、このモードは便利です。

[Top](#top)

<a name="skipforwarding-option"></a>
## `skipForwarding` オプション

CPrs への転送をスキップするために、クエリで `skipForwarding` オプションを使用できます (例:
`GET /v2/entities?options=skipForwarding`)。この場合、クエリは CB ローカル・コンテキスト情報のみを使用して評価
されます。

`skipForwarding` を更新しても効果がないことに注意してください (更新をローカルで CB に解釈する場合は、追加/作成
セマンティクスを使用して更新要求を使用するだけです)。

[Top](#top)

<a name="null-support-in-datetime-and-geolocation-types"></a>
## DateTime および geolocation タイプでの `null` サポート

NGSIv2仕様によると:

* `DateTime` 属性およびメタデータ: ISO8601 形式の文字列である必要があります
* `geo:point`, `geo:line`, `geo:box`, `geo:polygon` および `geo:json`: 属性は特定のフォーマット・ルールに
  従う必要があります ("エンティティの地理空間プロパティ" セクションで定義されています)

これらの場合に `null` 値がサポートされているかどうかは、NGSIv2 仕様では明確ではありません。
明確にするために、Orion はその可能性をサポートします。

`DateTime` 属性およびメタデータに関して:

* `DateTime` 属性または `null` 値を持つメタデータはフィルターで考慮されません。つまり
  `GET /v2/entities?q=T>2021-04-21` 

`geo:` 属性に関して:

* `null` 値を持つ `geo:` 属性は、ジオ・クエリでは考慮されません。つまり、ジオ・クエリの結果として
  エンティティは返されません
* `null` 値を持つ `geo:` 属性は[属性の制限](#limit-to-attributes-for-entity-location)にはカウントされません

[Top](#top)

<a name="keyvalues-not-supported-in-post-v2opnotify"></a>
## `POST /v2/op/notify` でサポートされない `keyValues`

現在の Orion の実装は、`POST /v2/op/notify` オペレーションの `keyValues` オプションをサポートしていません。それを使用しようとすると、400 Bad Request エラーが発生します。

[Top](#top)


<a name="deprecated-features"></a>
## 廃止予定の機能

安定版の NGSIv2 仕様の変更を最小限に抑えようとしていますが、最後にいくつかの変更が必要でした。したがって、現在の NGSIv2 stable 仕様には現れない機能が変更されていますが、Orion は後方互換性を維持するために([非推奨の機能](../deprecated.md)として)サポートしています。

特に、`options` のパラメータ内の `dateCreated` 及び `dateModified` の使用はまだサポートされています (安定した RC-2016.05 で導入され、RC-2016.10 で除去されました)。例えば `options=dateModified` です。ただし、代わりに `attrs` の使用することを強くお勧めします (すなわち `attrs=dateModified,*`)。

[トップ](#top)
