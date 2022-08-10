参考：https://qiita.com/t-yama-3/items/572fabc873b4b6a0fc7c

1. ajax で非同期通信を行う

- テンプレート
  ```js
  $.ajax({
    url: /* リクエストを送信するURLを指定 */,
    type: /* HTTPメソッドを指定 */,
    data: /* 送信データを指定 */,
    dataType: /* レスポンスデータの形式を指定 */
  })
  .done(function(data) {
    // (1) リクエストが成功した場合に行う処理
  })
  .fail(function() {
    // (2) リクエストが成功しなかった場合に行う処理
  })
  .always(function() {
    // (3) リクエストの成功・失敗に関わらず行う処理
  });
  ```
- 入力例
  ```js
  $(function () {
    $("#note_form").on("submit", function (e) {
      e.preventDefault(); // デフォルトのイベント(ページの遷移やデータ送信など)を無効にする
      $.ajax({
        url: $(this).attr("action"), // リクエストを送信するURLを指定（action属性のurlを抽出）
        type: "POST", // HTTPメソッドを指定（デフォルトはGET）
        data: {
          note: $("#note").val(), // 送信データ
        },
      })
        .done(function (data) {
          $(".notes").append(`<div>${data}</div>`); // HTMLを追加
          $("#note").val(""); // 入力欄を空にする
        })
        .fail(function () {
          alert("error!"); // 通信に失敗した場合の処理
        });
    });
  });
  ```
  - ＜補足＞
    `$(this).attr("action") `で、action 属性の URL 文字列（"/test"）を取り出しています。
  - `type: "POST"` で、HTTP メソッドを「POST」と指定しています。
  - `data: { note: $("#note").val() }` で、リクエストで送るデータを「キー/値のペア」で指定しています。
  - `$("#note")`により、id="note" の要素を探し出しています。
  - `$(".notes").append([HTML文]);` で、class="notes" のタグの子要素として [HTML 文] を追加しています。<br><br>
- HTML は次のように記述しておきます。

  ```html
  <!DOCTYPE html>
  <html>
    <head>
      <meta charset="UTF-8" />
      <title>Ajax Test</title>
      <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
      <script src="/js/note.js"></script>
    </head>
    <body>
      <form method="post" action="/test" id="note_form">
        <input type="text" id="note" />
        <button type="submit">送信</button>
      </form>
      <div class="notes"></div>
    </body>
  </html>
  ```

- ＜参考：CSRF 対策が有効な場合＞

  - なお、Spring Security などを導入していて、CSRF 対策が有効になっている場合は、Thymeleaf を使用して、form タグの action 属性を次のようにする必要があります。
  - この、CSRF トークンがないまま POST メソッド等でリクエストをしても、サーバ側で拒否されてしまいます。

  ```html
  <html xmlns:th="http://www.thymeleaf.org">
    <!-- 略 -->
    <form method="post" th:action="@{/test}" id="note_form">
      <!-- 略 -->
    </form>
  </html>
  ```

- jQuery で実装（CSRF 対策あり）
- pring Security などを導入して CSRF 対策を有効にしている場合は、CSRF トークンを併せて送信しないと、認可エラーが生じます。
- 具体的には、送信するデータの項目に、`_csrf: $("*[name=_csrf]").val()` と追加することで、POST メソッドによるリクエストが可能となります。

```js
$(function () {
  $("#note_form").on("submit", function (e) {
    e.preventDefault();
    $.ajax({
      url: $(this).attr("action"),
      type: "POST",
      data: {
        note: $("#note").val(),
        _csrf: $("*[name=_csrf]").val(), // CSRFトークンを送信
      },
    })
      .done(function (data) {
        $(".notes").append(`<div>${data}</div>`);
        $("#note").val("");
      })
      .fail(function () {
        alert("error!");
      });
  });
});
```

- ネイティブ JavaScript で実装

```js
window.addEventListener("load", function () {
  document.getElementById("note_form").addEventListener(
    "submit",
    function (e) {
      e.preventDefault(); // デフォルトのイベントを無効にする
      let inputText = document.getElementById("note").value; // 入力値を取得
      let url = document.getElementById("note_form").getAttribute("action"); // action属性のurlを抽出
      let data = "note=" + encodeURIComponent(inputText); // URIエンコード
      let httpRequest = new XMLHttpRequest(); // XMLHttpRequestのインスタンス作成
      httpRequest.open("POST", url, true); // open(HTTPメソッド, URL, 非同期通信[true:default]か同期通信[false]か）
      httpRequest.setRequestHeader(
        "Content-Type",
        "application/x-www-form-urlencoded"
      ); // リクエストヘッダーを追加(URIエンコードで送信)
      httpRequest.send(data); // sendメソッドでサーバに送信
      httpRequest.onreadystatechange = function () {
        if (httpRequest.readyState === 4) {
          // readyStateが4になればデータの読込み完了
          if (httpRequest.status === 200) {
            // statusが200の場合はリクエストが成功
            let divElm = document.createElement("div"); // 追加用のdivタグを作成
            divElm.appendChild(document.createTextNode(httpRequest.response)); // レスポンスで得た値をdivタグの子要素に追加
            document.getElementsByClassName("notes")[0].appendChild(divElm); // class="notes" の子要素として追加
            document.getElementById("note").value = ""; // テキストエリアを空白に戻す
          } else {
            // statusが200以外の場合はリクエストが適切でなかったとしてエラー表示
            alert("error");
          }
        }
      };
    },
    false
  );
});
```

- ＜ CSRF 対策が有効な場合＞
- spring Security を導入すると、CSRF 対策への対応も必要となります。
  jQuery の場合と同様に、次のように data の内容を変更して、CSRF トークンを送信するようにすればリクエストをすることができます。
  - 修正前
  ```js
  let data = "note=" + encodeURIComponent(inputText);
  ```
  - 修正後
  ```js
  let csrfToken = document.getElementsByName("_csrf")[0].value; // csrfトークンを取得
  let data =
    "note=" +
    encodeURIComponent(inputText) +
    "&_csrf=" +
    encodeURIComponent(csrfToken); // リクエストbodyにcsrfトークンを追加
  ```
  JavaScript の Ajax については、次の記事で細かく書いていますのでご参考としていただければ幸いです。
  - Rails における Ajax の実装（JavaScript と jQuery のコード比較）
    https://qiita.com/t-yama-3/items/7148d25b17e70ceb48d6
  - JavaScript の多次元ハッシュ（連想配列）を一括で URI エンコードする
    https://qiita.com/t-yama-3/items/e4c16bcbd42fd7ec50f8
- Controller
  - コントローラーは、次のように書きます。

```java
package com.example.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping
public class TestController {
    @GetMapping
    public String index() {
        return "index";
    }

    @PostMapping("/test")
    @ResponseBody
    public String note(@RequestParam String note) {
        return note + note;
    }
}
```

1. リクエストデータを受け取る
   クライアントから送信されたリクエストパラメータを受け取るには、＠RequestParam アノテーションを使用します。
   メソッドの引数に、＠RequestParam String note と指定することで、キーが note のデータを取得することができます。

1. メソッドの戻り値をレスポンスデータにする
   ＠Controller で作成したコントローラーのメソッドの戻り値は、基本的に view の遷移先となります。
   この場合、メソッドに ＠ResponseBody アノテーションを付けることで、戻り値を HTTP レスポンスのコンテンツとすることができます。

- なお、＠RestController を使用してコントローラーを作成すると、メソッドの戻り値がそのままレスポンスのコンテンツとなります（後述「3-2. ＠RestController を使用する場合」に記載）。

- サンプルコードは以上です。 これで、Ajax を使用したデータのやり取りができます。

* 補足
  - 補足として、別のパターンによる実装にも触れておきます。

1. レスポンスデータを JSON で返す場合

1. レスポンスデータは JSON で返すことの方が多いと思いますので、そのパターンを書いておきます。

1. JavaScript ファイル

- JavaScript ファイルでは、レスポンスデータを JSON 形式で受け取れるように修正します。

3-1-1-1. jQuery で実装
/src/main/resources/static/js/note.js
$(function() {
  $("#note_form").on("submit", function(e) {
    e.preventDefault();
    $.ajax({
      url: $(this).attr("action"),
      type: "POST",
      data: {
        note: $("#note").val()
      },
      dataType: "json"  // レスポンスデータをjson形式と指定する
    })
    .done(function(data) {
      $(".notes").append(`<div>${data.note}</div>`); // JSON 形式のレスポンスから note を取得
$("#note").val("");
    })
    .fail(function() {
      alert("error!");
    })
  });
});
修正したのは、コメントを付した２行です。
dataType: "json" で、レスポンスデータをjson形式と指定しています。
${data.note} で、JSON 形式で受け取った data から、キーが note の値を取得しています。

なお、Spring Security を導入していると、CSRF トークンの送信も必要となります。
実装方法は、前述の「2-3-2. （参考）jQuery で実装（CSRF 対策あり）」を参照してください。

3-1-1-2. （参考）ネイティブ JavaScript で実装
ネイティブ javaScript で書くと次のとおりです。

/src/main/resources/static/js/note.js
window.addEventListener("load", function() {
document.getElementById("note_form").addEventListener("submit", function(e) {
e.preventDefault();
let inputText = document.getElementById("note").value;
let url = document.getElementById("note_form").getAttribute("action");
let data = "note=" + encodeURIComponent(inputText);
let httpRequest = new XMLHttpRequest();
httpRequest.open("POST", url, true);
httpRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
httpRequest.setRequestHeader("Accept", "application/json"); // リクエストヘッダーを追加（クライアントが理解できるコンテンツタイプをサーバに通知）
httpRequest.responseType = "json"; // レスポンスデータを json 形式と指定
httpRequest.send(data);
httpRequest.onreadystatechange = function() {
if (httpRequest.readyState === 4) {
if (httpRequest.status === 200) {
let divElm = document.createElement("div");
divElm.appendChild(document.createTextNode(httpRequest.response.note)); // JSON 形式のレスポンスから note を取得
document.getElementsByClassName("notes")[0].appendChild(divElm);
document.getElementById("note").value = "";
} else {
alert("error");
}
}
};
}, false);
});
修正したのは、コメントを付した部分です。

① レスポンスデータを json 形式と指定
レスポンスデータを json 形式と指定するには、次の２行が必要です.

Sample
httpRequest.setRequestHeader("Accept", "application/json");
httpRequest.responseType = "json";
最初の行では、クライアントが受け取るデータタイプを JSON と指定しています（参考記事）。
２行目は、受け取るデータの形式が JSON 形式であること指定しています（指定しなければ"note":"hogehoge" という形のままですが、指定すれば note: "hogehoge" という形のデータとして受け取れるので、その後の処理が簡単になります）。

② JSON 形式のデータから値を取得
次の１行で、受け取ったデータ httpRequest.response から、キーが note のデータを取り出して、HTML コードを生成しています。

Sample
divElm.appendChild(document.createTextNode(httpRequest.response.note));
なお、Spring Security を導入していると、CSRF トークンの送信も必要となります。
実装方法は、前述の「2-3-3. （参考）ネイティブ JavaScript で実装」を参照してください。

3-1-2. Controller
コントローラーでは、次のようにして、レスポンスデータを単純に JSON 形式にしています。
これにより、戻り値は "{"note":"テストテスト"}" というような文字列になります。

/src/main/java/com/example/demo/TestController.java
// 略
@Controller
@RequestMapping
public class TestController {
// 略
@PostMapping("/test")
@ResponseBody
public String note(@RequestParam String note) {
String json = "{\"note\":\"" + note + note + "\"}";
return json;
}
}
＜参考サイト＞
・JSON とは？データフォーマット（データ形式）について学ぼう！

3-2. ＠RestController を使用する場合
＠Controller に代えて ＠RestController を使用すると、メソッドの戻り値がそのままレスポンスのコンテンツになります。
Ajax のみのコントローラーを作成する場合は、こちらを使用した方がシンプルに記述できます。

/src/main/java/com/example/demo/TestController.java
// 略
@RestController
@RequestMapping
public class TestController {
@GetMapping
public ModelAndView index() {
ModelAndView mav = new ModelAndView();
mav.setViewName("index");
return mav;
}

    @PostMapping("/test")
    public String note(@RequestParam String note) {
        return note + note;
    }

}
＜参考サイト＞
・Spring Boot の RestController から HTML を生成する方法

さいごに
とりあえず、順序通りにコードを書けば動くものを記事として残しておきました。
まだ、色々と試している段階なので、CSRF トークンの送信方法については、もっと簡単な方法があるのかもしれません。より簡単な方法がわかりましたら追記するようにします。
