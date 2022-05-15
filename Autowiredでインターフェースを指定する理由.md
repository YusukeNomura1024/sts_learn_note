## @Autowiredで実装クラスではなくインターフェースを指定する理由

### デフォルトコンストラクターを使って以下のようにinterfaceであるinquiryDaoを使えるようにすることがありますが、どうして実装クラスを直接指定してはだめなのでしょうか？

```
    public final InquiryDao dao;

    @Autowired
    public InquiryServiceimpl(InquiryDao dao) {
        this.dao = dao;
    
    }
```
* 実装クラスはInquiryDaoImpl implements InquiryDaoとしています。
* InquiryDaoは実装クラスのスーパークラスでもあるので、InquiryDaoImplという実装クラスが出来上がっていなくても、Serviceクラス自体を動かす単体テストは実行可能になります。
* 


## そもそもなぜ@Autowiredを使う必要があるのか？
* まず実装クラスというのが、@Component/@Controller/@Service/@Repositoryといったアノテーションが付けられたクラスのことですね。このアノテーションはクラス単位に付与されて、アノテーションが付与されたクラスは、SpringBoot起動時に、フレームワークによってnewされて、コンテナに格納されます。
* @Autowiredを使うと、Springフレームワークが上記のように自動でインスタンスを生成したものを、変数に格納してくれます。
* 具体的な使い方としては、メソッドの前に@Autowiredをつけることで、引数にDIコンテナで管理しているクラスがあれば、自動でnewしてくれて引数に渡してくれます。この機能を利用して、デフォルトコンストラクターに@Autowiredをつけることで、このクラスがDIでインスタンス化されたときに、インスタンス化時に自動でべつのDIでインスタンス化していたクラスを挿入してくれます。
* userServiceインスタンスがDIに存在しており、userControllerというクラス内で、
```
    public final UserService userService;
    @Autowired
    public UserController(UserService userService){
        this.userService = userService;
    }
```
* という記述があれば、このUserControllerがDIでインスタンス化されるときに、引数として自動でDIで管理しているUserServiceが引数として渡される。
* 本来であれば、このコントローラは引数なしでDIでインスタンス化されるので、@Autowiredを使わない場合、コントローラ内で public final userService userService;というフィール定数を設定して、引数なしのデフォルトコンストラクター内でUserServiceをnewしてthis.userServiceに代入する必要がある。以下のように
```
    public final Userservice userService;

    public UserController(){
        UserService userService = new UserService();
        this.userService = userService;
    }
```
* Autowiredを使わなかった場合、インスタンス時にUserServiceを渡せないので、クラス内で、new UserServiceをしないといけなくなる。
* Autowiredを使わなかった場合、mainクラスであるすべてのクラスを使う側の記述内で実装したクラスをインスタンス化する必要があるが、そのインスタンス化をDIコンテナ側で自動で行ってくれるので、mainクラスに記述する必要がなくなる。
