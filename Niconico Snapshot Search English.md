# Niconico Douga "Snapshot Search API v2" Guide

This API can be used to search for and retrieve Niconico Douga content for analysis purposes.

## Announcement

* The FQDN will be changed from api.search.nicovideo.jp to snapshot.search.nicovideo.jp.
* Access to the new address will be available from March 1, 2024.
* Access to the old address is scheduled to be discontinued from April 1, 2024.

## Introduction

* You can search for videos by specifying keywords and filter conditions.
* The search results of this API are returned from the video search index, which is updated daily at 5:00 AM JST.

### API Endpoint

* [https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search](https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search)

### Header

Please specify the service name or application name in the User-Agent.

### Query Parameters

The following parameters are given as GET query parameters.

| Parameter Name | Type     | Optional | Default Value | Example   | Description                                                                                                                                                                                                 |
|--------------|--------|------------|--------------|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| q            | string | no         | -            | ゲーム     | Search keyword. See *1 for details on the format.                                                                                                                                                             |
| targets      | string | no         | -            | title,description,tags | Search target fields (multiple allowed, comma-separated). For keyword search, specify title, description, tags. For tag search (hits content with tags that exactly match the keyword), specify tagsExact. Can be omitted for keywordless searches. |
| fields       | string | yes        | -            | contentId,title,description,tags | Fields of the hit content to be included in the response (multiple allowed, comma-separated). See *2 for field names.                                                                                               |
| filters      | string | yes        | -            | -          | Narrows down the search results to content that matches the filter conditions. See *3 for the filter format. An empty string is treated as null.                                                                  |
| jsonFilter   | string | yes        | -            | -          | Used only when using complex filter conditions such as nesting OR and AND. Use `filters` for single OR / AND / NOT. See *4 for the filter format.                                                              |
| _sort        | string | no         | -            | -viewCounter | Specifies the sort order by concatenating the sign of the sort direction and the field name. The sort direction is specified as '+' for ascending or '-' for descending, and '-' is the default if not specified. Refer to *2 for usable fields. Content that is null will be placed last regardless of the sort direction. |
| _offset      | integer | yes        | 0            | 10        | Retrieval offset of the returned content. Maximum: 100,000                                                                                                                                                    |
| _limit       | integer | yes        | 10           | 10        | Maximum number of content to be returned. Maximum: 100                                                                                                                                                         |
| _context     | string | no         | -            | apiguide  | Service or application name. Maximum: 40 characters                                                                                                                                                           |


#### *1 Query String Specification

| Feature Name     | Format                  | Example                 | Description                                                                                              | Remarks                                                              |
|-----------------|-----------------------|----------------------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| AND Search      | Separate with spaces    | ミク ボーカル         | Content that includes both "ミク" and "ボーカル"                                                              |                                                                    |
| OR Search       | Separate with OR        | ミク OR ボーカル       | Content that includes either "ミク" or "ボーカル"                                                              | Please insert spaces before and after OR.                                 |
| -             | -                     | ミク OR ボーカル OR ヴォーカル | Content that includes "ミク" or "ボーカル" or "ヴォーカル"                                                |                                                                    |
| AND/OR Search   | -                     | ボーカル OR 歌ってみた ミク | Content that includes ("ボーカル" or "歌ってみた") and "ミク"                                            | Write OR search first when combining AND and OR.                     |
| Phrase Search   | Enclose in double quotes | "ミク ボーカル"       | Content that includes "ミク ボーカル" including spaces                                                          | You can search including spaces and operators by enclosing them in double quotes. |
| NOT Search      | Prefix with -           | ミク -ボーカル         | Content that includes "ミク" but does not include "ボーカル".                                                           |                                                                    |
| -             | -                     | ミク - ボーカル       | Content that includes "ミク" and "-" and "ボーカル"                                                      | Do not insert a space between - and the word.                             |
| -             | -                     | -ボーカル           | Error                                                                                               | You cannot execute NOT search alone.                               |
| Keywordless Search | Specify an empty string | search?q=&fields=... | Searches without using keywords.                                                                   | You cannot omit q= itself. This is a high-load search, so please use it in conjunction with filters to narrow down the number of hits to within 100,000. |


#### *2 Field Specifications

Null may be returned for all fields due to reasons such as no value being entered.

| Field Name    | Description                                                                                                                                                                                                  | Type      | Retrievable with fields | Specifiable in _sort | Specifiable in filters |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|--------------------|--------------------|--------------------|
| contentId      | Content ID. Concatenating this after [https://nico.ms/](https://nico.ms/) will result in the URL to the content.                                                                | string  | o                  | -                  | o                  |
| title          | Title                                                                                                                                                                                                   | string  | o                  | -                  | -                  |
| description    | Content description.                                                                                                                                                                                          | string  | o                  | -                  | -                  |
| userId         | User ID of the uploader if it is a user-submitted video                                                                                                                                                                        | integer | o                  | -                  | o                  |
| channelId      | Channel ID if it is a channel video                                                                                                                                                                         | integer | o                  | -                  | o                  |
| viewCounter    | Number of views                                                                                                                                                                                                   | integer | o                  | o                  | o                  |
| mylistCounter  | Number of My Lists or favorites.                                                                                                                                                                               | integer | o                  | o                  | o                  |
| likeCounter    | Number of likes.                                                                                                                                                                                                  | integer | o                  | o                  | o                  |
| lengthSeconds  | Playback time (seconds)                                                                                                                                                                                               | integer | o                  | o                  | o                  |
| thumbnailUrl   | Thumbnail URL                                                                                                                                                                                              | string  | o                  | -                  | -                  |
| startTime      | Content upload time.                                                                                                                                                                                         | string (ISO8601 format)  | o                  | o                  | o                  |
| lastResBody    | Latest comment                                                                                                                                                                                               | string  | o                  | -                  | -                  |
| commentCounter | Number of comments                                                                                                                                                                                                   | integer | o                  | o                  | o                  |
| lastCommentTime | Last comment time                                                                                                                                                                                            | string (ISO8601 format)  | o                  | o                  | o                  |
| categoryTags   | Category tags                                                                                                                                                                                                 | string  | o                  | -                  | o                  |
| tags           | Tags (space-separated)                                                                                                                                                                                             | string  | o                  | -                  | o                  |
| tagsExact      | Exact match tags (space-separated)                                                                                                                                                                                        | string  | -                  | -                  | o                  |
| genre          | Genre                                                                                                                                                                                                   | string  | o                  | -                  | o                  |
| genre.keyword  | Exact match genre                                                                                                                                                                                           | string  | -                  | -                  | o                  |


#### *3 Filter Specification

To filter only content where field field0 matches any of the values val0, val1..., specify the URL parameters as follows:

```
filters[field0][0]=val0&filters[field0][1]=val1
```

To filter only content where field field1 matches the range val0 to val1 (excluding val0 and val1), specify the URL parameters as follows:

```
filters[field1][gt]=val0&filters[field1][lt]=val1
```

Use gte and lte if you want to include the range values.

You can use integer or string (date) fields for filters. Please refer to *2 Field Specifications for details.

Examples

* Filter for videos with 1 million views

```
filters[viewCounter][0]=1000000
```

* Filter for videos with 1000 or more My Lists and 1000 or more comments

```
filters[mylistCounter][gte]=1000&filters[commentCounter][gte]=1000
```

* Filter for content posted in 2014 (Specify the date and time in the YYYY-MM-DDThh:mm:ss±hh:mm format of ISO 8601 format)

```
filters[startTime][gte]=2014-01-01T00:00:00+09:00&filters[startTime][lt]=2015-01-01T00:00:00+09:00
```

* Filter for videos in the "Game" genre

```
filters[genre][0]=ゲーム
```

#### *4 JSON Filter Specification

You need to encode it when you actually use it.

|           | Key            | Description                                                                     |
|-----------|-----------------|--------------------------------------------------------------------------|
| Common      | type           | One of `equal,range,or,and,not`                                      |
| type == equal | field          | Field to target with equal                                          |
|           | value          | Value to target with equal                                                  |
| type == range | field          | Field to target with range                                          |
|           | from           | Starting value of the range                                                          |
|           | to             | Ending value of the range                                                          |
|           | include_lower | Whether to include the value of from in the range                                                |
|           | include_upper | Whether to include the value of to in the range                                                  |
| type == or  | filters        | Array of JSON filters                                                      |
| type == and | filters        | Array of JSON filters                                                      |
| type == not | filter         | JSON filter                                                          |


* Filter only for content posted on July 7th in 2016 and 2017 (Specify the date and time in the YYYY-MM-DDThh:mm:ss±hh:mm format of ISO 8601 format)

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

#### *5 When you want to retrieve data beyond the upper limit of _offset

Due to system limitations, you cannot specify a huge _offset. If you need more data than the limit, you can retrieve it by shifting the range using filters.

* When using startTime in filters

```
# Get January 2019 data
filters[startTime][gte]=2019-01-01T00:00:00+09:00&filters[startTime][lt]=2019-02-01T00:00:00+09:00

# Get February 2019 data
filters[startTime][gte]=2019-02-01T00:00:00+09:00&filters[startTime][lt]=2019-03-01T00:00:00+09:00

...
```

---

## Response

The following JSON will be returned.

### Success

| Field Name   | Type      | Example                                   | Description                                         |
|---------------|---------|----------------------------------------|----------------------------------------------|
| meta         | object  | -                                        | Response metadata field                  |
| meta.status   | integer | 200                                     | HTTP status (200 if successful)                 |
| meta.id       | string  | 594513df-85ea-4122-9859-f4ec2701cacf | Request ID                                   |
| meta.totalCount | integer | 12673                                  | Number of hits                                     |
| data          | array   | -                                        | Hit content. The contents of the elements vary depending on the parameter fields |

Example

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

### Error

| Field Name      | Type      | Example                         | Description                                         |
|-------------------|---------|------------------------------|----------------------------------------------|
| meta            | object  | -                             | Response metadata field                  |
| meta.status      | integer | 400                          | HTTP status (other than 200 if error)                 |
| meta.errorCode   | string  | QUERY_PARSE_ERROR            | Error code                                   |
| meta.errorMessage | string  | query parse error            | Error message                                     |


#### status: 400

Invalid parameter.

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

Search server error.

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

The service is under maintenance. Please wait until the maintenance is complete.

```json
{
  "meta": {
    "status": 503,
    "errorCode": "MAINTENANCE",
    "errorMessage": "please retry later."
  }
}
```

## Sample Query and Execution Example

Here are some examples of search queries with the following conditions:

* Videos with "初音ミク" in the title
* Videos with 10,000 or more views
* Information to retrieve: Content ID, Title, Number of views
* Top 3 videos sorted by number of views in descending order

#### URL

* Depending on the execution environment, you may need to encode `+` which represents ascending order to `%2B`.

```
https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search?q=%E5%88%9D%E9%9F%B3%E3%83%9F%E3%82%AF&targets=title&fields=contentId,title,viewCounter&filters[viewCounter][gte]=10000&_sort=-viewCounter&_offset=0&_limit=3&_context=apiguide
```

#### Execution Example with curl

```bash
curl -A 'apiguide application' 'https://snapshot.search.nicovideo.jp/api/v2/snapshot/video/contents/search?targets=title&fields=contentId,title,viewCounter&_sort=-viewCounter&_offset=0&_limit=3&_context=apiguide_application' --data-urlencode "q=初音ミク" --data-urlencode "filters[viewCounter][gte]=10000"
```

#### Response

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

## Data Update

Updates are performed once a day.
The data that can be referenced by this API is the data as of 5:00 AM JST, but the data update process will be performed until it can be referenced.

You can get the switching date and time of the data currently being referenced from the following endpoint.

#### Endpoint for Obtaining Switching Date and Time

[https://snapshot.search.nicovideo.jp/api/v2/snapshot/version](https://snapshot.search.nicovideo.jp/api/v2/snapshot/version)

#### Response (JSON)

```json
{
  "last_modified" : "yyyy-MM-ddTHH:mm:ss+09:00"
}
```

---

## Notes

* This API does not guarantee that the content matches the keyword search and tag search on the Niconico Douga page.
* All date and time data in this API and this guide are in Japan Standard Time (JST).
* If the search using this API takes more than 1 second, please consider dividing the search using filters, etc. Reducing the number of search hits may allow you to retrieve data faster.
* When the status of the response of this API is 503, the server is under heavy load, so please wait for more than 5 minutes before retrying.

### Usage Restrictions

Since this API is intended for high-load usage for analysis purposes,

**Limitations on the number of simultaneous connections**, and **restrictions on excessive use by the same user** are in place.

If you make an API request that exceeds the limit, the search results will not be returned correctly.
When making repeated API requests, please wait the same amount of time as the previous API response time.

---

## API Terms of Use

```
These API Terms of Use (hereinafter referred to as the "Terms of Use") govern the terms and conditions for the use of the Snapshot API (hereinafter referred to as the "API") by users of the API (hereinafter referred to as the "Users") to provide consistent search results at a certain point in time as a video search for the service "niconico" planned and operated by DWANGO Co., Ltd. (hereinafter referred to as the "Company"). Users must read and agree to these Terms of Use before using the API. The Company shall deem that the User has agreed to these Terms of Use by using the API.

These Terms of Use may be changed at the Company's sole discretion. The revised terms and conditions of use shall apply from the time the Company posts the revised Terms of Use, and the User shall agree to this in advance.

Article 1 (License to Use the API)
1. The Company grants the User a license to use the API and the documentation provided in connection with the API (hereinafter referred to as the "Documentation"). However, the User shall use the API in accordance with the manner and method described in these Terms of Use and the Documentation.
2. The API and the Documentation may not be used for commercial purposes.
3. In using the API, the User represents and warrants to the Company that the User is not an anti-social force (including, but not limited to, organized crime groups, members of organized crime groups, quasi-members of organized crime groups, organized crime group-related companies, corporate racketeers, social movement racketeers, political movement racketeers, special intelligence violent groups, and persons who associate with anti-social forces; hereinafter the same) and will not be so in the future.
4. The Company may revoke the license to use the API granted in paragraph 1 of this Article without notice to the User if the User violates these Terms of Use.

Article 2 (User's Obligations)
1. When the User provides the API to a third party (hereinafter referred to as the "End User") by using the API in the User's own application (hereinafter referred to as the "Application"), the User shall be responsible for properly providing the Application to the End User in accordance with the terms and conditions of use presented to the End User as the entity providing the Application. In addition, the User shall clearly indicate to the End User that the Application and the services related thereto are created and operated by the User and that the User is responsible for them.
2. The User shall comply with the Act on the Protection of Personal Information, the Act on Specified Commercial Transactions, the Installment Sales Act, the Act against Unjustifiable Premiums and Misleading Representations, and other relevant laws and regulations in operating the Application and related services.
3. The User shall, at its own responsibility and expense, handle all disputes with End Users or third parties arising in connection with the use of the API and the provision of the Application (including disputes arising after the termination of the provision of the API to the User under these Terms of Use), and the Company shall not be involved in any way and shall not be liable for any such disputes. In addition, if the Company incurs any costs in connection with responding to such claims or demands, or if the Company pays any damages, the User shall pay the Company such costs and damages (including attorneys' fees paid by the Company).

Article 3 (Intellectual Property Rights, etc.)
1. All copyrights, trademark rights, design rights, patent rights, utility model rights, know-how, and other rights (hereinafter referred to as "Intellectual Property Rights") relating to "niconico" and the API shall belong to the Company, even if they are incorporated into the Application. The User shall not assign, lease, sublicense, or otherwise dispose of the Intellectual Property Rights to any third party.
2. The User shall not use the API and the Company's Intellectual Property Rights beyond the scope expressly permitted in these Terms of Use.
3. If the User causes damage to the Company in violation of the preceding paragraph, the User shall compensate the Company for such damage (including indirect damage).

Article 4 (Prohibited Matters)
The User shall not engage in any of the following acts when using the API (including when incorporating the API into the Application and when the End User uses the Application). If the User engages in any of the following acts, the Company may suspend the User's use of the API without any notice to the User, and if the Company incurs any damage as a result, the User shall compensate the Company for such damage (including indirect damage).
(1) Use of the API in a manner or method not described in the Documentation
(2) Use for business activities or commercial purposes, or preparatory acts for such use
(3) Reproduction, modification, translation, distribution to third parties, or transmission (including automatic public transmission and enabling transmission) of the Documentation. However, this shall not apply to cases where it falls under "quotation" under the Copyright Act.
(4) Acts that adversely affect or interfere with the functions or performance of niconico
(5) Acts that are contrary to public order and morals (including anti-social activities and publicity activities for such activities)
(6) Criminal acts (including uploading or distributing computer viruses, junk mail, spam mail, chain letters, or other harmful files, aiding and abetting murder, abduction of minors, and pyramid schemes) and acts that promote or imply the commission of such criminal acts
(7) Acts that infringe the intellectual property rights of the Company, other Users, or third parties
(8) Acts that damage the property, credit, or reputation of the Company, other Users, or third parties, and acts that infringe the right to privacy, portrait rights, or other rights
(9) Acts that violate laws and regulations or treaties, whether intentionally or negligently
(10) Acts that cause economic or mental disadvantage to the Company, other Users, or third parties
(11) Acts of slander, defamation, or harassment against the Company, other Users, or third parties
(12) Acts that violate the Public Offices Election Act
(13) Acts that are deemed to have a harmful effect on minors
(14) Acts that are deemed to be social acts related to sex, religion, or politics
(15) Acts that obstruct or may obstruct the operation of niconico or any other services provided by the Company
(16) Acts that damage or may damage the credit or reputation of niconico or any other services provided by the Company
(17) Acts that cause gambling or gambling, or solicit participation in gambling or gambling
(18) Acts that violate the provisions of these Terms of Use
(19) Acts of collecting or accumulating personal information
(20) Cases that fall under the prohibited matters of the "niconico Terms of Use"
(21) Provision of benefits to anti-social forces
(22) Other acts that the Company deems inappropriate

Article 5 (Disclaimer)
1. The User shall use the API after agreeing to the following items in advance, and the Company shall not be liable for any damage caused to the User by the following items.
(1) The API is provided in the state in which the Company possesses it at the time of provision, regardless of whether it is expressly or implicitly stated.
(2) The Company does not guarantee that the API is free from errors, bugs, logical errors, defects, interruptions, or other defects.
(3) The Company does not guarantee the suitability, usefulness (benefit), security, title, non-infringement, accuracy, etc. of the API for the User's intended purpose.
(4) The Company shall not be obligated to modify or improve the API.
(5) The Company may change all or part of the specifications of the API at any time.
(6) The Company may terminate the provision of the API at any time.
2. The User shall use the API at its own risk and expense, and the Company shall not be liable for any damages arising in connection with the use or inability to use the API (including cases where the Company has been advised of the possibility of such damages). However, this shall not apply to damages to the User caused by the Company's intentional or gross negligence.

Article 6 (Confidentiality)
The User shall treat all information received from the Company in connection with the use of the API and the Documentation as confidential and shall not disclose it to any third party without the prior written consent of the Company.

Article 7 (General Provisions)
1. The User shall delete the program of the Application using the API when the User terminates the use of the API for any reason.
2. The User shall not assign or transfer to any third party any rights or obligations under these Terms of Use.
3. These Terms of Use and the relationship between the User and the Company regarding the use of the API based on these Terms of Use shall be governed by the laws of Japan.
4. Any and all disputes arising out of or relating to these Terms of Use shall be subject to the exclusive jurisdiction of the Tokyo District Court in the first instance.
5. The Company's failure to exercise or enforce any right set forth in these Terms of Use shall not constitute a waiver of such right.

Established on October 15, 2014
```

## Update History

* 2016/12/07 Initial release
* 2017/05/29 Added a note to Sample Query and Execution Example, and corrected the description of lastCommentTime
* 2017/08/03 Added explanation about jsonFilter
* 2017/08/16 Corrected the notation of field specifications
* 2018/09/06 Added explanation about keywordless search
* 2019/06/24 Added tagsExact, thumbnailUrl, lastResBody, genre, genre.keyword
* 2019/07/02 Changed the value of _offset from unlimited to a maximum of 100,000
* 2019/07/03 Added explanation about search requests that take more than 1 second to Notes. Added explanation about repeated API requests to Usage Restrictions. Added filter example by genre. Removed filter example by category tag
* 2019/07/19 Corrected the type description of lastCommentTime, and added description for cases where the field value is null
* 2021/01/05 Corrected the description of genre.keyword
* 2021/03/05 Added description about date and time format
* 2021/05/20 Added userId, channelId, likeCounter
* 2021/06/03 Changed contentId to be specifiable in filter
* 2021/07/01 Changed likeCounter to be specifiable in sort and filter
* 2023/11/14 Announced that the API endpoint will be https only
* 2024/03/01 Announced the FQDN change and revised the description

---