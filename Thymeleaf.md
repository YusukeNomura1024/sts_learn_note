## ビュー
1. 渡された値を表示させる
    1. 一時的に表示させる場合
        1. コントローラーで@ModelAttribute というアノテーションが付いた引数は設定した変数名で呼び出せる
    1. 特定のリダイレクトのみ表示
        1. コントローラーでフラッシュアットリビュートなどで渡された値を@ModelAttributeで受け取ればcompleteという変数が渡された時のみ表示できるようになる。（渡されていなときにもエラーにならない）
            ```
            <h2 th:text="${complete}"></h2>
            ```
    1. オブジェクトを丸ごと受け取る
        ```
        th:object="${}" 
        ```  
    1. オブジェクトのフィールドを使用したいタグ、もしくは親のタグに付ける。
    1. th:object="${}"を記述すると、その要素か子要素でth:xxx="*{xx}"とすることで、objectのxxというフィールドを取り出せる
1. フォームに値を渡す
    1. th:value th:fieldなど値の状況により使い分ける。
    1. フォーム後の確認ページを実装する場合
        1. 入力後確認用ページでも、type="hidden"を使って、値を渡していく必要がある
        1. 確認表示用にth:text=で表示をさせて、それとは別に、input type="hidden" th:value="xxx"と記述する必要がある。
        1. ここでも開発ツールを使えば値を変えることができてしまうので、バリデーションを取り入れる。
1. フォームエリア
    1. 送る範囲をformタグで囲む
    1. method="post" actionはドメインが変わることもあるので、"#"としておく。
    1. th:actionで@{/survey/comfirm}としておくと@はドメインの意味となる。
    1. int型でデータを送る場合はtype="number"とした方がいい
    1. ラジオボタン
    ```
    　　<label for="satisfaction">満足度:</label>
        <input type="radio" name="satisfaction" value="1" th:checked="*{satisfaction == 1}">1
        <input type="radio" name="satisfaction" value="2" th:checked="*{satisfaction == 2}">2
        <input type="radio" name="satisfaction" value="3" th:checked="*{satisfaction == 3}">3
        <input type="radio" name="satisfaction" value="4" th:checked="*{satisfaction == 4}">4
        <input type="radio" name="satisfaction" value="5" th:checked="*{satisfaction == 5}">5<br>
    ```
    * 一致するフォーム名はすべて揃えて、戻ってきたときに、選択した値が自動で選ばれるように。satisfactionが1であれば1のチェックボックスのcheckedがtrueになりそれ以外はfalseになる
    * 初めて入力フォームに来たときは、空白なので、チェックはついていない
1. エラー表示
    ```
    th:if="${#fields.hasErrors('エラーを表示させたいフィールド名')}" th:errors="*{エラーを表示させたいフィールド名}"
    ```
    * とすることで、そのフィールドにあったエラーを表示できる。
    * #fieldsにはそのフォームに関する情報が入っており、hasErrorsと引数でエラーの有無を確認
    * th:ifとすることで、エラーがあればそのエラー文を表示できる。
        ```
        <div th:if="${#fields.hasErrors('age')}" th:errors="*{age}"></div>
        ```
1. 繰り返し表示
    1. th:eachを使う
        ```
        <tr th:each="inquiry : ${inquiryList}">
		<td th:text="${inquiry.id}">1</td>
		<td th:text="${inquiry.name}"></td>
		<td th:text="${inquiry.email}"></td>
		<td th:text="${inquiry.contents}"></td>
		<td th:text="${inquiry.created}"></td>
	    </tr>
        ```
        * コントローラーから渡されたinquiryListというList型のデータを、inquiryという変数に一つずつ取り出して、処理を繰り返すという意味になります。
        * このinquiryはentityのinquiryクラスですので、フィールドに設定されているので、inquiryに続けてドット＋フィールド名で値を取り出せます。

## 条件分岐
```
        <div th:unless="${#strings.isEmpty(complete)}" >
            <div th:text="${complete}" class="alert alert-success" role="alert">
                A simple success alert—check it out!
		    </div>
	    </div>
```
* この記述は、complete変数の中に値が格納されていなかったらという意味の記述になります。
    * th:unlessでtrueじゃないときに表示できるようにします。
    * つまり、completeが空じゃないときに表示するということになる。

