---
html: get-started-using-http-websocket-apis.html
parent: http-websocket-apis-tutorials.html
blurb: XRP Ledgerの操作に使用できるAPIとライブラリを使い始めましょう。
cta_text: 開始しよう
labels:
  - 開発
top_nav_name: HTTP / WebSocket
top_nav_grouping: 始めましょう
showcase_icon: assets/img/logos/globe.svg
---
# HTTP / WebSocket APIの使用開始

XRP Ledgerのコアサーバーソフトウェアは[`rippled`](xrpl-servers.html)です。XRP Ledgerでの開発に進むには、`rippled`サーバーのAPIにアクセスします。

APIにアクセスする最も簡単な方法は、[**WebSocket API Tool**](websocket-api-tool.html)を使用するか、[XRP Ledger Explorer](https://livenet.xrpl.org/)を使用してレジャーの進行状況をその場で確認することです。

[`rippled`の独自のインスタンスを実行](install-rippled.html)したり、[公開サーバー](#公開サーバー)を使用したりすることもできます。

## 公開サーバー

Rippleは、XRP Ledgerコミュニティ向けにいくつかの公開サーバーを提供しています。

| 演算子 | [ネットワーク][] | JSON-RPC URL | WebSocket URL | 注記 |
|:----------|:----------|:----------|:----------|:----------|
| Ripple | **Mainnet** | `https://s1.ripple.com:51234/` | `wss://s1.ripple.com/` | 汎用サーバークラスター |
| Ripple | **Mainnet** | `https://s2.ripple.com:51234/` | `wss://s2.ripple.com/` | [すべての履歴が記録されるサーバー](ledger-history.html#すべての履歴)クラスター |
| Ripple | Testnet | `https://s.altnet.rippletest.net:51234/` | `wss://s.altnet.rippletest.net/` | Testnet公開サーバー |
| Ripple | Devnet | `https://s.devnet.rippletest.net:51234/` | `wss://s.devnet.rippletest.net/` | Devnet公開サーバー |

[ネットワーク]: parallel-networks.html

これらの公開サーバーは継続的な使用やビジネスでの使用を想定したものではなく、いつでも使用不可となる可能性があります。日常的な使用については、独自の`rippled`サーバーを自社で運用するか、信頼できる事業者と運用委託契約を締結します。


## 管理者アクセス権限

`rippled`サーバーの[管理メソッド](admin-api-methods.html)を使用するには、次のように行います。この場合、サーバーのバインド用として設定したIPアドレスとポートを使用する必要があります（例えば`127.0.0.1:54321`）。また、管理機能にアクセスするには、構成ファイルで管理用としてマークされているポートおよびIPアドレスから接続しなければなりません。

[構成ファイルの例](https://github.com/XRPLF/rippled/blob/8429dd67e60ba360da591bfa905b58a35638fda1/cfg/rippled-example.cfg#L1050-L1073)では、ローカルループバックネットワーク上（127.0.0.1）のポート5005でJSON-RPC（HTTP）、ポート6006でWebSocket（WS）の接続をリッスンし、接続されるすべてのクライアントを管理者として扱っています。


## WebSocket API

いくつかのメソッドをXRP Ledgerで試すことを予定している場合は、独自のWebSocketコードを記述することなく、[Ripple WebSocket APIツール](websocket-api-tool.html)でAPIをすぐに使用できます。後ほど、独自の`rippled`サーバーへの接続が必要となった時点で、[ブラウザー](monitor-incoming-payments-with-websocket.html)または[Node.jsで独自のクライアントをビルド](https://www.npmjs.com/package/ws)することが可能です。

### 要求フォーマット

`rippled`サーバーへのWebSocketを開いた後、以下の属性を使用して、コマンドを[JSON](https://en.wikipedia.org/wiki/JSON)オブジェクトとして送信できます。

* コマンド名を最上位の`"command"`フィールドに記述します。
* コマンドのすべての関連パラメーターも最上位に記述します。
* 任意の値を指定して`"id"`フィールドを記述します（省略可）。この要求への応答では、同一の`"id"`フィールドを使用します。そうすることで、応答が順不同で到達した場合も、どの要求によってどの応答を得られたのかがわかります。

応答はJSONオブジェクトとして返されます。

## JSON-RPC

任意のHTTPクライアント（[RESTED for Firefox](https://addons.mozilla.org/en-US/firefox/addon/rested/)、[Postman for Chrome](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)、[Online HTTP client ExtendsClass](https://extendsclass.com/rest-client-online.html)など）を使用して、JSON-RPCで`rippled`サーバーを呼び出すことができます。ほとんどのプログラミング言語には、HTTP要求を組み込むためのライブラリーが用意されています。

### 要求フォーマット

JSON-RPC要求を作成するには、`rippled`サーバーがJSON-RPC接続をリッスンしているポートおよびIPアドレス上で、HTTP **POST**要求をルートパス（`/`）に送信します。HTTP/1.0またはHTTP/1.1を使用できます。HTTPSを使用する場合は、TLS v1.2を使用してください。セキュリティーの維持を理由として、`rippled`はSSL v3以前を _サポートしていません_ 。

値を`application/json`として、`Content-Type`ヘッダーを常に記述してください。

複数の要求を作成することを予定している場合は、要求ごとに接続を閉じて再び開くことなく済むよう、[キープアライブ](http://tools.ietf.org/html/rfc7230#section-6.3)を使用します。

以下の属性を指定して、要求の本文を[JSON](https://en.wikipedia.org/wiki/JSON)オブジェクトとして送信します。

* コマンドを最上位の`"method"`フィールドに記述します。
* 最上位の`"params"`フィールドを記述します。このフィールドの内容は、コマンドのすべてのパラメーターが指定された1つの入れ子JSONオブジェクトのみを保持している**1要素配列**です。

応答もJSONオブジェクトになります。


## コマンドライン

このコマンドラインインターフェイスは、JSON-RPCのものと同一のサービスに接続するため、公開サーバーおよびサーバー構成は同一です。コマンドラインクライアントとして、`rippled`がローカルインスタンスに接続します。例:

```
rippled --conf=/etc/rippled.cfg server_info
```

**注記:** コマンドラインインターフェイスは、管理の目的でのみ使用されることを想定しています。 _サポートされるAPIではありません_。


### 要求フォーマット

コマンドラインでは、通常の（先頭にダッシュが付いた）コマンドラインオプションに続けてコマンドを記述した後、一連の限定的なパラメーターを空白文字で区切って記述します。空白文字などの特殊な文字が含まれている可能性があるパラメーター値は、一重引用符で囲みます。


## 要求の例

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```
{
  "id": 2,
  "command": "account_info",
  "account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
  "strict": true,
  "ledger_index": "validated"
}
```

*JSON-RPC*

```
POST http://s1.ripple.com:51234/
{
    "method": "account_info",
    "params": [
        {
           "account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
            "strict": true,
            "ledger_index": "validated"
        }
    ]
}
```

*コマンドライン*

```
rippled account_info r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59 validated true
```

<!-- MULTICODE_BLOCK_END -->


## 応答フォーマット

### 成功した場合の応答の例

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```
{
   "id": 2,
  "status": "success",
  "type": "response",
  "result": {
    "account_data": {
      "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
      "Balance": "27389517749",
      "Flags": 0,
      "LedgerEntryType": "AccountRoot",
      "OwnerCount": 18,
      "PreviousTxnID": "B6B410172C0B65575D89E464AF5B99937CC568822929ABF87DA75CBD11911932",
      "PreviousTxnLgrSeq": 6592159,
      "Sequence": 1400,
      "index": "4F83A2CF7E70F77F79A307E6A472BFC2585B806A70833CCD1C26105BAE0D6E05"
    },
    "ledger_index": 6760970
  }
}
```

*JSON-RPC*

```
HTTP Status:200 OK
{
    "result": {
        "account_data": {
            "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
            "Balance": "27389517749",
            "Flags": 0,
            "LedgerEntryType": "AccountRoot",
            "OwnerCount": 18,
            "PreviousTxnID": "B6B410172C0B65575D89E464AF5B99937CC568822929ABF87DA75CBD11911932",
            "PreviousTxnLgrSeq": 6592159,
            "Sequence": 1400,
            "index": "4F83A2CF7E70F77F79A307E6A472BFC2585B806A70833CCD1C26105BAE0D6E05"
        },
         "ledger_index": 6761012,
         "status": "success"
    }
}
```

*コマンドライン*

```
{
    "result": {
        "account_data": {
            "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
            "Balance": "27389517749",
            "Flags": 0,
            "LedgerEntryType": "AccountRoot",
            "OwnerCount": 18,
            "PreviousTxnID": "B6B410172C0B65575D89E464AF5B99937CC568822929ABF87DA75CBD11911932",
            "PreviousTxnLgrSeq": 6592159,
            "Sequence": 1400,
            "index": "4F83A2CF7E70F77F79A307E6A472BFC2585B806A70833CCD1C26105BAE0D6E05"
        },
        "ledger_index": 6761012,
        "status": "success"
    }
}
```

<!-- MULTICODE_BLOCK_END -->

成功した場合の応答に含まれているフィールドは、以下のとおりです。

| `Field` | 型 | 説明 |
|:----------|:----------|:----------|
| `id` | （場合により異なる） | （WebSocketのみ）この応答の要求元となった要求で提供されているID。 |
| `status` | 文字列 | （WebSocketのみ）値が`success`である場合、要求がサーバーによって正常に受信され、理解されたことを示します。 |
| `result.status` | 文字列 | （JSON-RPCおよびコマンドライン）値が`success`である場合、要求がサーバーによって正常に受信され、理解されたことを示します。 |
| `type` | 文字列 | （WebSocketのみ）値が`response`である場合、コマンドに対する正常な応答であることを示します。[非同期の通知](subscribe.html)では、`ledgerClosed`や`transaction`など異なる値が使用されます。 |
| `result` | オブジェクト | クエリーの結果。内容はコマンドによって異なります。 |

### コマンドライン

コマンドラインのメソッドはJSON-RPCと同一のインターフェイスを使用しているため、応答フォーマットはJSON-RPCの応答と同一です。

## 関連項目

- **コンセプト:**
  - [XRP Ledgerの概要](xrp-ledger-overview.html)
  - [ソフトウェアエコシステム](software-ecosystem.html)
  - [並列ネットワーク](parallel-networks.html)
- **チュートリアル:**
  - [xrpl.js for JavaScriptの使用開始](get-started-using-javascript.html)
  - [信頼できるトランザクションの送信](reliable-transaction-submission.html)
  - [rippledサーバーの管理](manage-the-rippled-server.html)
- **リファレンス:**
  - [rippled APIリファレンス](http-websocket-apis.html)
  - [Ripple Data API v2](data-api.html)
