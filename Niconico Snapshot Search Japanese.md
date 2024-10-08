# ニコニコ動画 『スナップショット検索API v2』 ガイド

ニコニコ動画のコンテンツを解析する目的で検索/取得する際に利用できます

## 告知

* FQDNがapi.search.nicovideo.jpからsnapshot.search.nicovideo.jpに変更になります
* 2024/3/1から変更先にアクセスできます。
* 2024/4/1から変更元のアクセスができなくなる予定です。

## はじめに

* キーワードやフィルタ条件を指定して、動画を検索できます。
* 当APIの検索結果は、毎日AM5:00に更新される動画検索インデックスから返されます。

### APIエンドポイント

* [https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search](https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search)

### ヘッダー

User-Agentにサービス名またはアプリケーション名を指定してください。

### クエリパラメータ

以下のパラメータを GET のクエリパラメータで与えます。

| パラメータ名 | 型     | 省略可能か | デフォルト値 | 例         | 説明                                                                                                  |
|--------------|--------|------------|--------------|------------|-------------------------------------------------------------------------------------------------------|
| q            | string | no         | -            | ゲーム     | 検索キーワードです。 書式の詳細は＊1を参照してください。                                                  |
| targets      | string | no         | -            | title,description,tags | 検索対象のフィールド（複数可、カンマ区切り）です。キーワード検索の場合、title,description,tagsを指定してください。タグ検索（キーワードに完全一致するタグがあるコンテンツをヒット）の場合、tagsExactを指定してください。キーワード無し検索の場合は省略できます。 |
| fields       | string | yes        | -            | contentId,title,description,tags | レスポンスに含みたいヒットしたコンテンツのフィールド（複数可、カンマ区切り）です。フィールド名は＊2を参照してください。 |
| filters      | string | yes        | -            | -          | 検索結果をフィルタの条件にマッチするコンテンツだけに絞ります。フィルタの書式には＊3を参照してください。空文字はnull扱いになります。 |
| jsonFilter   | string | yes        | -            | -          | OR や AND の入れ子など複雑なフィルター条件を使う場合のみに使用します。 OR / AND / NOT 単体で使用する場合は `filters` を使ってください。フィルタの書式には＊4を参照してください。 |
| _sort        | string | no         | -            | -viewCounter | ソート順をソートの方向の記号とフィールド名を連結したもので指定します。ソートの方向は昇順または降順かを'+'か'-'で指定し、ない場合はデフォルトとして'-'となります。使用できるフィールドは＊2を参考にしてください。nullになっているコンテンツはソートの方向に関わらず最後になります。 |
| _offset      | integer | yes        | 0            | 10        | 返ってくるコンテンツの取得オフセット。最大:100,000                                                              |
| _limit       | integer | yes        | 10           | 10        | 返ってくるコンテンツの最大数。最大:100                                                                    |
| _context     | string | no         | -            | apiguide  | サービスまたはアプリケーション名。最大:40文字                                                                  |


#### ＊1 クエリ文字列仕様

| 機能名        | 書式                  | 例                 | 説明                                                                                              | 備考                                                              |
|-----------------|-----------------------|----------------------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| AND検索       | 空白（半角）で区切る     | ミク ボーカル         | "ミク" かつ "ボーカル"を含むコンテンツ                                                              |                                                                    |
| OR検索        | ORで区切る             | ミク OR ボーカル       | "ミク"または"ボーカル"を含むコンテンツ                                                              | ORの前後には空白を入れてください。                                 |
| -             | -                     | ミク OR ボーカル OR ヴォーカル | "ミク"または"ボーカル"または"ヴォーカル"を含むコンテンツ                                                |                                                                    |
| AND・OR検索    | -                     | ボーカル OR 歌ってみた ミク | ("ボーカル" または"歌ってみた") でかつ "ミク" を含むコンテンツ                                            | ANDとORを組み合わせる場合OR検索を先に書きます。                     |
| フレーズ検索    | ダブルクオート" で囲む   | "ミク ボーカル"       | 空白を含む "ミク ボーカル" を含むコンテンツ                                                          | ダブルクオートで囲むことで空白や演算子も含めて検索できます。             |
| NOT検索       | 単語の前に-をつける       | ミク -ボーカル         | "ミク"を含んでボーカルを含まないコンテンツ。                                                           |                                                                    |
| -             | -                     | ミク - ボーカル       | "ミク" かつ "-" かつ "ボーカル"を含むコンテンツ                                                      | - と単語の間に空白をいれないでください。                             |
| -             | -                     | -ボーカル           | エラー                                                                                               | NOT検索だけ実行することはできません。                               |
| キーワード無し検索 | 空文字を指定する         | search?q=&fields=... | キーワードを使用せずに検索します。                                                                   | q=自体の省略はできません。負荷の高い検索となりますので、filtersと併用しヒット件数を10万件以内に絞り込んだ上でご利用ください。 |


#### ＊2 フィールド仕様

すべてのフィールドで値が入ってないなどの理由でnullが返ってくる場合があります。

| フィールド名    | 説明                                                                                                                                                                                                  | 型      | fieldsで取得可能か | _sortに指定可能か | filtersに指定可能か |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|--------------------|--------------------|--------------------|
| contentId      | コンテンツID。[https://nico.ms/](https://nico.ms/) の後に連結することでコンテンツへのURLになります。                                                                | string  | o                  | -                  | o                  |
| title          | タイトル                                                                                                                                                                                                   | string  | o                  | -                  | -                  |
| description    | コンテンツの説明文。                                                                                                                                                                                          | string  | o                  | -                  | -                  |
| userId         | ユーザー投稿動画の場合、投稿者のユーザーID                                                                                                                                                                        | integer | o                  | -                  | -                  |
| channelId      | チャンネル動画の場合、チャンネルID                                                                                                                                                                         | integer | o                  | -                  | -                  |
| viewCounter    | 再生数                                                                                                                                                                                                   | integer | o                  | o                  | o                  |
| mylistCounter  | マイリスト数またはお気に入り数。                                                                                                                                                                               | integer | o                  | o                  | o                  |
| likeCounter    | いいね！数                                                                                                                                                                                                  | integer | o                  | o                  | o                  |
| lengthSeconds  | 再生時間(秒)                                                                                                                                                                                               | integer | o                  | o                  | o                  |
| thumbnailUrl   | サムネイルのURL                                                                                                                                                                                              | string  | o                  | -                  | -                  |
| startTime      | コンテンツの投稿時間。                                                                                                                                                                                         | string (ISO8601形式)  | o                  | o                  | o                  |
| lastResBody    | 最新のコメント                                                                                                                                                                                               | string  | o                  | -                  | -                  |
| commentCounter | コメント数                                                                                                                                                                                                   | integer | o                  | o                  | o                  |
| lastCommentTime | 最終コメント時間                                                                                                                                                                                            | string (ISO8601形式)  | o                  | o                  | o                  |
| categoryTags   | カテゴリタグ                                                                                                                                                                                                 | string  | o                  | -                  | o                  |
| tags           | タグ(空白区切り)                                                                                                                                                                                             | string  | o                  | -                  | o                  |
| tagsExact      | タグ完全一致(空白区切り)                                                                                                                                                                                        | string  | -                  | -                  | o                  |
| genre          | ジャンル                                                                                                                                                                                                   | string  | o                  | -                  | o                  |
| genre.keyword  | ジャンル完全一致                                                                                                                                                                                           | string  | -                  | -                  | o                  |


#### ＊3 フィルタ指定仕様

フィールド field0 が val0 または val1 ... の値のいずれかに一致するコンテンツだけにフィルタリングする場合、以下のようにURLパラメータを指定します。

```
filters[field0][0]=val0&filters[field0][1]=val1
```

フィールド field1 が val0 〜 val1 の範囲（val0,val1含まず）に一致するコンテンツだけにフィルタリングする場合、以下のようにURLパラメータを指定します。

```
filters[field1][gt]=val0&filters[field1][lt]=val1
```

範囲の値も含めたい場合gteとlteを使用します。

フィルタにはinteger、またはstring(日付)のフィールドを利用できます。詳しくは＊2のフィールド仕様を参考にしてください。

例

* 再生数が100万のフィルタ

```
filters[viewCounter][0]=1000000
```

* マイリスト数が1000以上かつコメント数が1000以上のフィルタ

```
filters[mylistCounter][gte]=1000&filters[commentCounter][gte]=1000
```

* 2014年に投稿されたコンテンツでフィルタ（日時はISO 8601形式のうち、YYYY-MM-DDThh:mm:ss±hh:mmのフォーマットで指定してください）

```
filters[startTime][gte]=2014-01-01T00:00:00+09:00&filters[startTime][lt]=2015-01-01T00:00:00+09:00
```

* ジャンルがゲームのフィルタ

```
filters[genre][0]=ゲーム
```

#### ＊4 JSONフィルタ指定仕様

実際に使用する際はエンコードする必要があります。

|           | キー            | 説明                                                                     |
|-----------|-----------------|--------------------------------------------------------------------------|
| 共通      | type           | `equal,range,or,and,not`のいずれか                                      |
| type == equal | field          | equal で対象にしたいフィールド                                          |
|           | value          | equal で対象にしたい値                                                  |
| type == range | field          | range で対象にしたいフィールド                                          |
|           | from           | 範囲の始点の値                                                          |
|           | to             | 範囲の終点の値                                                          |
|           | include_lower | fromの値を範囲に含めるか                                                |
|           | include_upper | toの値を範囲に含めるか                                                  |
| type == or  | filters        | JSONフィルターの配列                                                      |
| type == and | filters        | JSONフィルターの配列                                                      |
| type == not | filter         | JSONフィルター                                                          |


* 2016年と2017年で 7月7日に投稿されたコンテンツのみフィルタ（日時はISO 8601形式のうち、YYYY-MM-DDThh:mm:ss±hh:mmのフォーマットで指定してください）

```json
{
  "type": "or",
  "filters": [
    {
      "type": "range",
      "field": "startTime",
      "from": "2017-07-07T00:00:00+09:00",
      "to": "2017-07-08T00:00:00+09:00",
      "include_lower": true
    },
    {
      "type": "range",
      "field": "startTime",
      "from": "2016-07-07T00:00:00+09:00",
      "to": "2016-07-08T00:00:00+09:00",
      "include_lower": true
    }
  ]
}
```

#### ＊5 _offsetの上限を越えてデータを取得したい場合

システムの制限により、巨大な_offsetを指定することはできません。制限を越えた件数のデータが必要な場合は、filtersを使って範囲をずらしていくことで取得できます。

* filtersにstartTimeを使う場合

```
# 2019年1月分を取得
filters[startTime][gte]=2019-01-01T00:00:00+09:00&filters[startTime][lt]=2019-02-01T00:00:00+09:00

# 2019年2月分を取得
filters[startTime][gte]=2019-02-01T00:00:00+09:00&filters[startTime][lt]=2019-03-01T00:00:00+09:00

...
```

---

## レスポンス

以下のJSONが返ってきます。

### 正常な場合

| フィールド名   | 型      | 例                                   | 説明                                         |
|---------------|---------|----------------------------------------|----------------------------------------------|
| meta         | object  | -                                        | レスポンスのメタ情報フィールド                  |
| meta.status   | integer | 200                                     | HTTPステータス（成功の場合200）                 |
| meta.id       | string  | 594513df-85ea-4122-9859-f4ec2701cacf | リクエストID                                   |
| meta.totalCount | integer | 12673                                  | ヒット件数                                     |
| data          | array   | -                                        | ヒットしたコンテンツ。要素の内容はパラメータfieldsによって異なります |

例

```json
{
  "meta": {
    "status": 200,
    "totalCount": 1,
    "id":"594513df-85ea-4122-9859-f4ec2701cacf"
  },
  "data": [
    {
      "contentId": "sm9",
      "title": "テスト",
      "description": "テスト",
      "startTime": "2016-11-03T02:09:11+09:00",
      "viewCounter": 1
    }
  ]
}
```

### エラーの場合

| フィールド名      | 型      | 例                         | 説明                                         |
|-------------------|---------|------------------------------|----------------------------------------------|
| meta            | object  | -                             | レスポンスのメタ情報フィールド                  |
| meta.status      | integer | 400                          | HTTPステータス（エラーの場合200以外）                 |
| meta.errorCode   | string  | QUERY_PARSE_ERROR            | エラーコード                                   |
| meta.errorMessage | string  | query parse error            | エラー内容                                     |


#### status: 400

不正なパラメータです。

```json
{
  "meta": {
    "status": 400,
    "errorCode": "QUERY_PARSE_ERROR",
    "errorMessage": "query parse error"
  }
}
```

#### status: 500

検索サーバの異常です。

```json
{
  "meta": {
    "status": 500,
    "errorCode": "INTERNAL_SERVER_ERROR",
    "errorMessage": "please retry later."
  }
}
```

#### status: 503

サービスがメンテナンス中です。メンテナンス終了までお待ち下さい。

```json
{
  "meta": {
    "status": 503,
    "errorCode": "MAINTENANCE",
    "errorMessage": "please retry later."
  }
}
```

## サンプルクエリと実行例

ここでは、以下のような条件の検索クエリを例示します。

* タイトルに「初音ミク」が含まれている動画
* 再生数が10000以上
* 取得する情報はコンテンツID・タイトル・再生数
* 再生数の多い順でソートした上位3件

#### URL

* 実行環境によっては，昇順を表す `+` は `%2B`にエンコードするなどの対応が必要になることがあります。

```
https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search?q=%E5%88%9D%E9%9F%B3%E3%83%9F%E3%82%AF&targets=title&fields=contentId,title,viewCounter&filters[viewCounter][gte]=10000&_sort=-viewCounter&_offset=0&_limit=3&_context=apiguide
```

#### curl での実行例

```bash
curl -A 'apiguide application' 'https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search?targets=title&fields=contentId,title,viewCounter&_sort=-viewCounter&_offset=0&_limit=3&_context=apiguide_application' --data-urlencode "q=初音ミク" --data-urlencode "filters[viewCounter][gte]=10000"
```

#### レスポンス

```json
{
  "meta": {
    "status": 200,
    "totalCount": 12673,
    "id": "0554268c-c0f3-4436-9098-7b9e07885623"
  },
  "data": [
    {
      "contentId": "sm1097445",
      "title": "【初音ミク】みくみくにしてあげる♪【してやんよ】",
      "viewCounter": 11904784
    },
    {
      "contentId": "sm1715919",
      "title": "初音ミク　が　オリジナル曲を歌ってくれたよ「メルト」",
      "viewCounter": 10230124
    },
    {
      "contentId": "sm15630734",
      "title": "『初音ミク』千本桜『オリジナル曲PV』",
      "viewCounter": 9557201
    }
  ]
}
```

---

## データの更新について

更新は一日に一回行われます。
当APIで参照できるデータは、AM5:00時点(日本標準時)でのデータですが、参照できる状態になるまでデータの更新作業が行われます。

以下のエンドポイントから、現在参照しているデータの切り替え日時を取得することができます。

#### 切り替え日時の取得のエンドポイント

[https://snapshot.search.nicovideo.jp/api/v2/snapshot/version](https://snapshot.search.nicovideo.jp/api/v2/snapshot/version)

#### レスポンス(JSON)

```json
{
  "last_modified" : "yyyy-MM-ddTHH:mm:ss+09:00"
}
```

---

## 注意事項

* 本APIはニコニコ動画ページにおけるキーワード検索およびタグ検索との、内容の一致を保証するものではありません
* 本APIの日時データおよび本ガイドに記載されている日時は、すべて日本標準時(JST)です
* 本APIによる検索にかかる時間が1秒を超える場合、フィルタなどを利用して分割して取得することをご検討ください。検索ヒット件数を下げることで、より高速にデータ取得できる可能性があります
* 本APIのレスポンスのステータスが503のときはサーバが高負荷になっているため、5分以上間隔を開けてからリトライするようお願いします

### 利用制限

本APIは、解析を目的とした負荷の高い利用法が想定されているため、

**同時接続数の制限**、および **同一ユーザーによる過度な利用に制限** などの利用制限を設けております。

制限を超えてAPIリクエストを行った場合、正常に検索結果が返されません。
繰り返しAPIリクエストを行う場合は、前回のAPIレスポンス時間と同じだけ待機時間を設けてご利用ください。

---

## API利用規約

```
このAPI利用規約（以下「本利用規約」といいます）は、株式会社ドワンゴ（以下「当社」といいます）が企画・運営するサービス「niconico」の動画検索として、ある時点の一貫性のある検索結果を提供するAPI『Snapshot API』（以下「本API」といいます）を本APIの利用者（以下「利用者」といいます）が利用するにあたっての諸条件について規定します。利用者は本APIを利用する前に、必ず本利用規約をご確認いただいた上、本利用規約に同意いただく必要があります。当社は、利用者が本APIを利用したことをもって本利用規約に同意したものとみなします。

なお、本利用規約は、当社の任意の判断により、変更されることがあります。変更後の利用条件は、変更後の本利用規約を当社が掲示したときから適用されるものとし、利用者はこれを予め承諾するものとします。

第1条（本APIの利用許諾）
1.    当社は、利用者に対し、本API及び本APIに関して提供されるドキュメント（以下「本ドキュメント」といいます）を利用することを許諾します。但し、利用者は、本利用規約及び本ドキュメントに記載された態様、方法に従って本APIを利用するものとします。
2.    本API及び本ドキュメントを、営利目的に利用することはできません。
3.    本APIの利用をするにあたり、利用者は、当社に対して、利用者が反社会勢力（暴力団、暴力団員、暴力団準構成員、暴力団関係企業、総会屋等、社会運動標榜ゴロ、政治活動標榜ゴロ、特殊知能暴力集団、反社会的勢力共生者、又はその他これらに準ずる者をいう。以下同じです。）に該当しないことを表明し、且つ将来にわたっても該当しないことを確約します。
4.    当社は、利用者が本利用規約に反した場合、利用者に催告なく、本条第１項に定める利用許諾を取り消すことができるものとします。

第2条（利用者の義務）
1.    利用者が本APIを利用者自身のアプリケーション(以下「アプリケーション」といいます)に利用して第三者に提供する場合(この場合の第三者を以下「エンドユーザー」といいます)、利用者はアプリケーションを提供する主体として、エンドユーザーに提示した利用条件に従ってエンドユーザーに適正にアプリケーションを提供する責任を負います。また、利用者は、アプリケーション及びこれに関するサービスが、利用者により制作し運営するものであり、利用者がその責任を負うものであることをエンドユーザーが理解できるよう明示するものとします。
2.    利用者は、アプリケーション及びこれに関わるサービスの運営にあたり、個人情報の保護に関する法律、特定商取引に関する法律、割賦販売法、不当景品類及び不当表示防止法、その他関係法令を遵守しなくてはなりません。
3.    利用者は、本APIの利用及びアプリケーションの提供にかかる一切の事項に関連して生じたエンドユーザー又は第三者との紛争（本利用規約に基づく利用者への本APIの提供が終了した後に生じた紛争を含む）について、利用者の責任と費用において一切を処理するものとし、当社は一切関与せず一切の責任も負いません。また、当該クレームや請求への対応に関連して当社に費用が発生した場合又は当社が損害賠償金等の支払いを行った場合については、利用者は当社に対し当該費用及び損害賠償金等（当社が支払った弁護士費用を含みます）を支払うものとします。

第3条（知的財産権等）
1.    「niconico」及び本APIに関する著作権、商標権、意匠権、特許権、実用新案権、ノウハウ、その他権利の一切（以下「知的財産権等」といいます）は、アプリケーションに組み込まれた場合であっても、当社に帰属します。利用者は、知的財産権等を第三者に譲渡、賃貸、サブライセンスその他の処分をしてはならないものとします。
2.    利用者は、本API及び当社の知的財産権等を、本利用規約において明示的に許諾している範囲を超えて利用したりすることはできません。
3.    前項に違反して、利用者が当社に損害を与えた場合、利用者はこの損害（間接的損害も含みます）を当社に対し賠償するものとします。

第4条（禁止事項）
利用者は、本APIの利用（本APIをアプリケーションに組み込む場合及びエンドユーザーによるアプリケーション利用に伴う場合を含む）にあたり、以下に該当する行為をすることはできません。利用者が以下の項目の一つにでも該当した場合、当社は、利用者に対して何等の催告もなく、本APIの利用を停止することができるものとし、またこれにより、当社に損害が生じた場合、利用者はこの損害（間接的損害も含みます）を当社に対し賠償するものとします。
(1) 本ドキュメントに記載されてない方法、態様での本APIの利用
(2) 営業活動、営利を目的とした利用またはその準備行為
(3) 本ドキュメントの複製、改変、翻訳、第三者への頒布、送信（自動公衆送信、送信可能化を含む）。但し、著作権法上の「引用」に該当する場合を除く。
(4) niconicoの機能・性能に悪影響を与えたり妨害したりする行為
(5) 公序良俗に反する行為（反社会的活動及びその宣伝活動も含む）
(6) 犯罪的行為（コンピュータウィルス・ジャンクメール・スパムメール・チェーンレターその他有害なファイルのアップロードや配布、殺人幇助、未成年者略取、ねずみ講にあたる行為を含む）及び、当該犯罪的行為を助長し又はその実行を暗示する行為
(7) 当社、他の利用者、又は第三者の知的所有権を侵害する行為
(8) 当社、他の利用者、又は第三者の財産・信用・名誉等を毀損する行為及び、プライバシーに関する権利、肖像権その他の権利を侵害する行為
(9) 故意、過失を問わず、法令、条約に反する行為
(10) 当社、他の利用者、又は第三者に経済的・精神的不利益を与える行為
(11) 当社、他の利用者、又は第三者に対する誹謗中傷、いやがらせの行為
(12) 公職選挙法に抵触する行為
(13) 未成年者に対し悪影響があると判断される行為
(14) 性風俗、宗教、政治に関する社会的行為であると判断される行為
(15) niconicoその他当社が提供する全てのサービスの運営を妨げる行為､又はそのおそれのある行為をしていると当社が判断した行為
(16) niconico及び当社が提供する全てのサービスの信用・名誉等を毀損する行為､又はそのおそれのある行為
(17)賭博・ギャンブルを行わせ、又は賭博・ギャンブルへの参加を勧誘する行為
(18) 本利用規約の条項に違反する行為
(19) 個人に関する情報の収集、蓄積行為
(20) 「niconico利用規約」の禁止事項に該当する場合
(21) 反社会的勢力への利益供与
(22) その他、当社が不適当と判断する行為

第5条（免責）
1.    利用者は、以下の各号をあらかじめ承諾の上、本APIを利用するものとし、以下の各号により利用者に損害が生じた場合であっても当社は一切責任を負わないものとします。
(1)    本APIは、明示又は黙示の有無にかかわらず、当社がその提供時において保有する状態で提供するものであること
(2)    本APIのエラーやバグ、論理的誤り、不具合、中断その他の瑕疵がないことについて当社が一切保証しないこと
(3)    利用者が予定している目的への適合性、有用性（有益性）、セキュリティ、権原、非侵害性及び正確性等について当社が一切保証しないこと
(4)    当社は本APIの修正又は改良義務を負わないこと
(5)    当社は、本APIの仕様の全部又は一部をいつでも変更することができること
(6)    当社は、本APIの提供をいつでも終了することができること
2.    利用者は、本APIを自己の責任と費用負担で利用するものとし、本APIの利用又は利用不能に関連して発生した損害について、当社は（当社が損害の発生する可能性を指摘されていた場合を含みます）一切の損害賠償責任を負わないことを承諾するものとします。ただし、当社の故意又は重大な過失に基づく利用者の損害についてはこの限りではありません。

第6条（機密保持）
利用者は、本API及び本ドキュメントの利用に関連して当社から受領した情報等の一切について、当社から事前に書面による承諾を得た場合を除き、機密として取り扱うものとし、第三者に開示してはならないものとします。

第7条（一般条項）
1.    利用者は、理由のいかんを問わず、本APIの利用を終了したときは、本APIを利用したアプリケーションのプログラムを消去するものとします。
2.    利用者は、本利用規約に基づく一切の権利義務を第三者に譲渡し、または承継させてはならないものとします
3.    本利用規約及び利用者と当社との本利用規約に基づく本APIの利用に関する関係には、日本法が適用されます。
4.    本利用規約に起因し又は関連する一切の紛争については、東京地方裁判所を第一審の専属的合意管轄裁判所とします。
5.    当社が、本利用規約に示される権利を行使又は実施しない場合にも、その権利を放棄するものではありません。

2014年10月15日制定
```

## 更新履歴

* 2016/12/07 初版公開
* 2017/05/29 サンプルクエリと実行例 に補足を追加、lastCommentTime の説明を修正
* 2017/08/03 jsonFilter についての説明を追加
* 2017/08/16 フィールド仕様の表記を修正
* 2018/09/06 キーワード無し検索についての説明を追加
* 2019/06/24 tagsExact, thumbnailUrl, lastResBody, genre, genre.keyword を追加
* 2019/07/02 _offsetの値を無制限から最大で100,000件に変更
* 2019/07/03 注意事項に1秒以上かかる検索リクエストについての説明を追加。利用制限に繰り返しAPIリクエストを行う場合についての説明を追加。ジャンルによるフィルタ例を追加。カテゴリタグによるフィルタ例を削除
* 2019/07/19 lastCommentTimeの型の記述を修正、フィールドの値がnullになっている場合の記述を追加
* 2021/01/05 genre.keywordの記述を修正
* 2021/03/05 日時のフォーマットについて記述を追加
* 2021/05/20 userId, channelId, likeCounter を追加
* 2021/06/03 contentIdをfilterで指定可能に変更
* 2021/07/01 likeCounterをsortとfilterで指定可能に変更
* 2023/11/14 APIエンドポイントをhttpsのみにする旨を告知
* 2024/03/01 FQDNの変更を告知し記述を修正

---