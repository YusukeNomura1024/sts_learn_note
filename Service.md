## Serviceクラスについて
1. Daoクラス（リポジトリ―）に対して命令をかける
1. serviceというパッケージに追加していく
2. インターフェースを作成する
    * InquiryService.java
    * 実装予定のメソッドを作成
    * 受け取ったentityをデータベースに保存するための指示をリポジトリ―に出すためのメソッド void save(Entity entity)
    * リスト一覧を取得するための、メソッド　List<Inquiry> getAll();
    ```
    void save(Inquiry inquiry);
    List<Inquiry> getAll();
    ```
1. 実装クラスの作成
1. InquiryServiceImpl.java（作成時にインターフェースの追加もしておくと楽に作成できる）
1. クラスの前に@ServiceをつけることでサービスクラスとしてDIにシングルトンとして登録されます
1. 作成しているDaoクラスを初期化してフィールドに加えます。
    ```
    private final InquiryDao dao;
    ```
    * ここではインターフェースであるInquiryDaoを呼び出すことが重要です。
1. デフォルトコンストラクターに@Autowiredをつけます。
    ```
	@Autowired
	public InquiryServiceImpl(InquiryDao dao) {
		this.dao = dao;
	}
    ```
    * 実際に渡される引数はインターフェースではなくて、実装クラスになります。
    ```
    @Override
	public void save(Inquiry inquiry) {
		dao.insertInquiry(inquiry);
	}
    ```
    * saveメソッドの実装は単純にリポジトリ―のinsertInquiryを呼び出すだけです。
1. コントローラークラスの修正
    1. サービスクラスを読み込めるようにするためにコントローラーでサービスクラスの初期化を行う。
        * 型の名前はインターフェース名を指定する。ちなみにインターフェース名を指定することで、実装クラスの名前が変わってもここの記述を変更する必要がないからである。
        ```
        private final InquiryService inquiryService;
        ```
    1. デフォルトコンストラクターを使ってinquiryServiceを使えるようにします。

        ```
        @Autowired
 	    public InquiryController(InquiryService inquiryService) {
 		    this.inquiryService = inquiryService;
 	    
        }
        ```

    1. 実際にServiceで作成したメソッドを使う必要があるコントローラーのメソッドに記述する
    1. フォームのデータをデータベースに保存するようなcompleteメソッドであれば、メソッド内で、フォームの値をエンティティに入れなおす作業が必要となる。
        * 実際にはもっと楽に実装する方法があるが、今回は丁寧にやります。
        * daoにデータを入れなおすときと同様に、getでformの値を取得して、setでentityに入れている。
        * Created（作成日)に関しては、フォームからではなく、javaのライブラリで現在の日時を取得して入れます。
        * Createdのようにフォームでは入力をしないが、値を渡したい場合はにはここで処理を記述する。
        * idは自動連番なので設定する必要はないです。
    1. Serviceクラスで作成したsaveメソッドを使って、値を入れなおしたinquiryを引数として渡す。
        * 追加した処理
        ```
        Inquiry inquiry = new Inquiry();
		inquiry.setName(inquiryForm.getName());
		inquiry.setEmail(inquiryForm.getEmail());
		inquiry.setContents(inquiryForm.getContents());
		inquiry.setCreated(LocalDateTime.now());
		
		inquiryService.save(inquiry);
        ```
    1. 一覧データを取得した場合は、GetMappingをつけたメソッド内でgetAllでリストを取得して、modelに格納してビューに渡せるようにします。
        ```
        List<Inquiry> list = inquiryService.getAll();





