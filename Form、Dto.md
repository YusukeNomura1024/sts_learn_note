### 日付の入力に関して
* 日付はすべて文字列として取得されるので、以下のようにアノテーションをつけてLocalDateTimeに変換する必要があります。
* htmlのinputのtype="datetime-local"に設定しておけばローカルの日時の文字列として受け取るだけなので、@DateTimeFormatを使ってフォーマットできる。
* 文字列として受け取った後に、サービスなどで修正をするのであれば、Stringでもよいですが、その場合、LocalDateTimeの日付に関するバリデーション（@Futureなど）が使えなくなります。
```
    @DateTimeFormat(pattern = "yyyy-MM-dd'T'HH:mm")
    private LocalDateTime deadline;
```
* @Future 入力時よりも過去の日付を入力しようとした場合、
```
@Future(message = "期限が過去に設定されています。")
private LocalDateTime deadline;
```

### Boolean型について
* 一般的にBoolean型のフィールド名はisXXXXとなる
* 新規か更新か判断するためのフィールドを作るテクニック
    * 以下のようにしておくことで、新規か更新が判定できるようにする

    ```
    public boolean isNewTask;
    ```

### 空入力に関して
* @InitBinderをコントローラーやアドバイスクラスに設定することで、コントローラ―に読み込まれる前に、空入力をnullに変換できます。
```
    @InitBinder
    public void initBinder(WebDataBinder dataBinder) {
        // 入力値の空文字をnullに変換
        dataBinder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
    }
```


