DAOパターン

コントローラーからサービスクラス（ビジネスロジック）へ検索用の値や、保存用のEntitiを渡す
↓
BL（ビジネスロジック）は対応するDAOクラスを見つけて依頼をかける
つまりコントローラーから受け取った値やデータをそのままDAOクラスに渡している
↓
JDBCドライバー
↓
DB
↓
DAO　は検索結果などをエンティティに詰めた状態（エンティティクラスとして）受け取る
BLへ表示データを渡したり、更新であれば、更新件数などを返します。
↓
BLはDAOから受け取ったentityをコントローラークラスへ返します。
もし更新が0件などおかしな挙動があれば、BLで例外処理をする―したりします。


DAOクラスの作り方
2つのクラスから構成される
interface と　implements

public interface inquiryDao{
	void insertInquiry(Inquiry inquiry);
	List<Inquiry> getAll();
}

interfaceであからかじめどういった実装をするのかを記述して置き
それをimplementsしたクラスに記述していく

implementsで作成するクラスには@Repositoryをつける
これによってデータベースを専門に扱うクラスであることがわかります。

1. パッケージでentityを作る
1. クラスはInquiry.javaなど
	1. entityパッケージはまずはフィールドの作成
	1. すべてprivateで記述
	1. フィールド名はテーブルのカラム名と一致させる
	1. テーブルに保存されるDateはTimeStamp型として保存されているので、Date系はDate型やLocalDateTimeなどにしておく
	1. デフォルトコンストラクターと、ゲッターセッターを用意（@Data で自動生成）
		* STSのソースからデフォルトコンストラクターを作成する場合は、フィールドはすべて外してから作成する
		* superが自動で生成されているが不要なので削除しておく

1. daoパッケージを作る
	1. InquiryDao.javaを作る（これをインターフェースとして作る）
	1. 必要なメソッドを入れていく（実装予定のメソッド）
		* ここに定義するメソッドはデータベースに直接作用するメソッドを作る
		* データベースに対して,select update delete insertなどの機能
		* void insertInquiry(Inquiry inquiry)やList<Inquiry> getAll();などとして定義
		
	1. 実装クラスを作っていく
		1. InquiryDaoImpl.javaなどDaoをimplementするクラスという意味
		1. 新規でクラスを作成するときに、インターフェースを追加することができるので、楽にimplementsされたクラスを作成できる（オーバーライドなども書かれている）
		1. クラスの前に@Repositoryを入れることで,データベースを操作するクラスであることをDIコンテナの方に伝える
		1. private final JdbcTemplate jdbcTemplateとすることで、このリポジトリ―クラスで、このjdbcTemplateに対していろいろと操作をすることで、データベースに変更を加えることができる。
		1. リポジトリ―クラスをインスタンス化しなくてもシングルトンクラスとしてDIコンテナーでいつでも呼び出せるように@Autowiredを使って、自動で生成されたこのInquiryDaoImplインスタンスにJdbcTemplateクラスを渡すことで、データベースを操作できることになります。
		```
			@Autowired
			InquriyDaoImpl(JdbcTemplate jdbcTemplate){
				this.jdbcTemplate = jdbcTemplate;
			}
		```
		1. オーバーライドするメソッドの内容を作成していく（jdbcTemplateに対して操作を加えることでテーブルに操作を加えることと同じ意味となる）
			* 更新する場合はjdbcTemplate.update("SQL文", 引数・・・);
			* idはオートインクルメントになるので指定禁止。自動で連番が振られるので、insert文のカラムには入れない
			```
			@Override
			public void insertInquiry(Inquiry inquiry) {
				jdbcTemplate.update("INSERT INTO inquiry(name, email, contents, created) VALUES(?, ?, ?, ?)",
				inquiry.getName(), inquiry.getEmail(), inquiry.getContents(), inquiry.getCreated());
			```
			* 上記のように第一引数のSQL文に対して、第2引数に？？？？に入れる値を指定する。
			* 引数を適当に指定していい理由は、このメソッドの受け取り側で引数を配列として受け取っているので、引数に指定はない。（第一引数をＳＱＬ文として引き取り、それ以降はＳＱＬ文に対して当てはめていく値として扱われる。
			* この書き方はプリペアードステートメントになっているので、sqlインジェクションを防ぐことができます。
			* すべての一覧を取得する場合はgetAll()
			* getAllはSQL文が変更されることはないので、変数にSQL文を文字列として格納する。
			* Stringをキーとして、何かしらのデータをもつマップ型のリストを作る
			* List<Map<String, Object>> resultList = jdbcTemplate.queryForList(sql);
			* jdbcTemplateにqueryForList(sql)をすることで受け取ることができる。一つのデータがマップ型になっており、それをList型で返す。
			* そしてその受け取ったMap型のListをビューで表示できるようにentityであるinquiryのフィールドにそれぞれ落とし込んで、最終的にinquiryのList型として返すようにします。（これによってinquiry.getName()のように簡単に扱えるようになります。）
			* そもそもオブジェクト指向的にはInquiryというデータを扱うという意味にする必要があります。
			* addでListに加えていくため、まずはList<Inquiry> list を初期化します。
			1. 繰り返し処理で初期化した空のinquiryにマップの中身をとりだしてsetしていきます。出来上がったinquiryをListに加えていきます。
			2. まず引数で受け取ったMap型のresultからidキーでidを取得（(int)result.get("id"))して、それをinquiry.setIdでセットします。※(int)でキャストしている理由は、Map型はString,　Objectという形なので、get("id")で取得した値はObject型のため、int型に戻しています。ほかにも(String)や(Timestamp)でキャストしています。
			* データベースではTimeDate型ですが、取り出すときはTimestamp型として引き出します。さらにentityではLocalDateTime型として設定していますので、.toLocalDateTime()というメソッドで型を修正して、entityに入れます。
			* import java.sql.Timestamp;とする必要がある
```
	@Override
	public List<Inquiry> getAll() {
		String sql = "SELECT id, name, email, contents, created FROM inquiry";
		List<Map<String, Object>> resultList = jdbcTemplate.queryForList(sql);
		List<Inquiry> list = new ArrayList<Inquiry>();
		for(Map<String, Object> result : resultList) {
			Inquiry inquiry = new Inquiry();
			inquiry.setId((int)result.get("id"));
			inquiry.setName((String)result.get("name"));
			inquiry.setEmail((String)result.get("email"));
			inquiry.setContents((String)result.get("contents"));
			inquiry.setCreated(((Timestamp)result.get("created")).toLocalDateTime());
			list.add(inquiry);
		}
		return list;
		
	}

```

		* Dao（リポジトリ―）はこの様にデータベースから値を引き出すだけでなく、entityに詰め込むという役割も担っています。、特定のentityオブジェクトとして扱えるようにしています。


