@PostMapping("/complete")
	public String complete(@Validated InquiryForm inquiryForm,
			BindingResult result,
			Model model,
			RedirectAttributes redirectAttributes) {


コントローラー
* @Controllerをクラスの上に付ける
	* 戻り値をStringにする
	* Viewの遷移先を指定する一番単純なやり方。
	```
		@RequestMapping("/method1")
    	public String method1() {
        		return "/index.html";
    	}
	```

　@RequestMapping("パス")をクラスの上につける

* リクエストに反応させる
	* @GetMapping("/form")とすることで、メソッドGETの時に、クラスに付けられたRequestMappingのパスの下の階層を指定
	* @PostMapping("/form"）とすることで、メソッドがPOSTの時に起動する
* 値を渡す
	* ページのタイトルを状況によって変えたい場合
	* model.addAttribute("title", "渡したい文字列")というようにtitleという変数に文字列を書くのして渡す
　　
　　メソッドの引数にForm系のクラスを渡すすることで、ビューにフォームのデータが渡される
　
* 値を受け取る
	* 戻り値をModelAndViewにする
		* ModevAndViewを使用するとViewに渡したい情報を一緒に返すことができる。
		```
		@RequestMapping("/method2")
		public ModelAndView method2() {
			ModelAndView modelAndView = new ModelAndView();
			modelAndView.addObject(new User("紀伊", "太郎"));
			modelAndView.setViewName("/index.html");
			return modelAndView;
		}　
		```
	
	* コントローラーの別のメソッドからリダイレクトされた時など
	* model.addAttributeなどで変数が作成されていた時は、コントローラーで受け取れる
　　@ModelAttribute("complete") String complete
    　このように、completeという変数に格納されているデータをここでは、String型のcompleteという変数に入れなおして渡しています。

	* リクエストパラメーターを受け取る
		* 引数に@RequestParam("taskId") int id,のように追加する
		* @RequestParamは通常String型で渡ってくるが、上記のように記述することで、int型に変換される。
		

* リターン
	* フォワード
		* 戻り値で指定する遷移先に"forward:"を付けると他のコントローラメソッドに遷移する。
		* ちなみに、フォワード：URLは変わりません。パラメータも継承されます。
			* URLはそのままで、ページの表示が変わりますので、直接アクセスしないコントローラーメソッドを呼び出すことに使います。
		* 同一サーバー内でユーザーが要求するページを別のページに強制的に変更するというもので、遷移先のページはサーバー内のため、データも同じサーバー内だからそのまま送ることができる。
		```
		@RequestMapping("/forward1")
		public String forward1() {
			return "forward:method1";
		}
		```

	* リダイレクト
		* リダイレクトしたい場合は"redirect:"を付ける。
		* ちなみに、リダイレクト：URLは変わります。パラメータも継承されません。
			* postのようにデータを受け取って処理をする必要がないコントローラーのメソッドへ移動するときに便利です。
			* 処理だけして画面の表示などは次のコントローラーに任せる感じですね。
			* RedirectAttributes redirectAttributeという引数に値を追加して送らないとデータを渡せません。
		* ユーザーが指定したURLを強制的に変更するというもので、遷移先のページは同じサーバーとは限らない。
		* 異なるサーバーへデータを送るわけにはいかないので、リダイレクトはデータを送れない
		```
		    @RequestMapping("/redirect1")
			public String redirect1(@RequestParam(defaultValue = "NOT INDEX PARAM") String msg、RedirectAttributes redirectAttribute) {
				redirectAttribute.addAttribute("msg", msg);
				return "redirect:method1";
			}

			// 外部URLを指定する事もできる。
			return "redirect:https://www.google.co.jp/";
		```
	* コントローラで直接コンテンツを返す
		* メソッドに@ResponseBodyを付けると戻り値で直接レスポンスのコンテンツを返す事ができる。
		```
		@RequestMapping("/text1")
		@ResponseBody
		public String text1() {
			return "text content";
		}
		```

		* 戻り値をvoidにしてHttpServletResponseを引数で受け取るようにすれば、直接レスポンスを書き込む事もできる。

		```
		@RequestMapping("/text2")
		public void text2(HttpServletResponse res) throws IOException {
			res.getWriter().write("text content");
		}
		```

　バリデーション
　　@Validatedアノテーションを受け取ったFormクラスに設定する
　　BindingResult型の引数を追加することで、Validatedの検証をする
　　バリデーションエラー時は現在のページにとどまるようにする。
　　　if(result.hasErrors())でresultにえらがあれば検知できる
　　　現在のページにとどまるときも、現在のページを再び表示するためのmodelを渡す
    フォームが送信されるときだけでなく、確認用ページでhiddenとしているときもバリデーションは必要
　　

* 保存
	* 二重保存を防ぐためにRedirectAttributesクラスを引数に加える　フラッシュスコープの活用
	* RedirectAttributes redirectAttributes
	* フラッシュのため、modelにaddAttしても、保存されないので、そのために、redirectAttributesにデータを渡すことになる
	* redirectAttributes.addFlashAttribute("", "");という記述になる
	* 正確に言うとセッションにデータが書くのされている形となる
	* セッションは一度読み込まれた時点でデータは破棄されるので、2重登録を防げる
		* redirect:を使って、フォーム送信したら、リダイレクトすることで回避できる
	```
	    @PostMapping("/update")
		public String update(
			@Valid @ModelAttribute TaskForm taskForm,
			BindingResult result,
			@RequestParam("taskId") int taskId,
			Model model,
			RedirectAttributes redirectAttributes) {
	```
	上記のように、引数に、RedirectAttributesを入れる



　ページの遷移
　　return "パス"文字列を返すことで、そのパスにあったテンプレートを表示させる
　　
　　リダイレクトさせる場合
　　　バリデーションエラー時、2重登録回避などの時につかう
　　　return "redirect:/xxxxx/"とする
　　　　このredirect:とすることで、テンプレートファイル名を指すのではなく、URL名を指すことになります
	* リダイレクトの時はコントローラ―を実行させ、リクエストの時はビューファイルを呼び出します。
	

フォーム
　値を一つずつやり取りするのではなく、インプットエリアに入力されたデータを一括でやり取りする
　それぞれインプットエリアなどに対応したフィールド名を持っているので名前注意
　<input id="name" name="name" type="text" th:value="*{name}">
  　このinputタグのnameプロパティの名前と、formクラスのフィールド名が一致するものが結びつけられるので、この名前が違えば値を渡せないので注意する。

　複数のデータを一括で処理する場合は
　　Listのフィールドを持つformを作ってそのそのフォームの中に複数入れたいフォームを格納する
　

コントローラーアドバイス
@ControllerAdviceをつけたクラスはコントローラー実行時に処理される
　@InitBinder
　　をつけることで、初期設定ができる
　　　nullだった場合はから文字として処理してくださいという指示もできる
