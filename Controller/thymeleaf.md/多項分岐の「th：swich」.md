* th:swichを使う
    ```
    <○○ th:swich="条件式">
        <△△ th:case="条件式">・・・</△△>
        <△△ th:case="条件式">・・・</△△>
    </○○>
    ```
    * th:swichは指定された条件式の値をチェックし、その内側にあるth:caseから同じ値のものを探す
    * 一致するタグだけを出力する
    ```html
    <div th:swich="${check}">
        <p th:case="0" th:text="${month}"></p>
        <p th:case="1" th:text="${month}"></p>
        <p th:case="2" th:text="${month}"></p>
        <p th:case="3" th:text="${month}"></p>
    </div>
        