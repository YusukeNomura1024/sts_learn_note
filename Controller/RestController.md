* @RestController
* 戻り値
    * 戻り値でテキストコンテンツを返す
        * @RestControllerを付けたコントローラのメソッドでは@ResponseBodyを付けなくても戻り値がレスポンスのコンテンツになる。
        ```
        @RequestMapping("/text1")
        public String text1() {
            return "text content";
        }
        ```

    * HttpServletResponseでテキストコンテンツを返す
        * @Controllerの場合と同じくHttpServletResponseで直接コンテンツを書き込める。
        ```
        @RequestMapping("/text2")
        public void text2(HttpServletResponse res) throws IOException {
            res.getWriter().write("text content");
        }
        ```

    * コンテンツタイプを指定する
        * コンテンツタイプは@RequestMappingのproducesで指定する
        ```
        @RequestMapping(value = "/xml1", produces = "application/xml")
        public String xml1() {
            return "<a><b>content</b></a>";
        }
        ```
        * org.springframework.http.MediaTypeクラスにコンテンツタイプの定数が定義してあるのでこれを使うといい。
        ```
        @RequestMapping(value = "/xml2", produces = MediaType.APPLICATION_XML_VALUE)
        public String xml2() {
            return "<a><b>content</b></a>";
        }
        ```
    * HTTPステータスやレスポンスヘッダを指定する
        * HTTPステータスやコンテンツタイプ以外のレスポンスヘッダも指定したい時は、戻り値をResponseEntityにする。
        ```
        @RequestMapping("/responseEntity")
        public ResponseEntity<String> responseEntity() {
            HttpHeaders headers = new HttpHeaders();
            headers.add("header1", "heaer1-value");
            HttpStatus status = HttpStatus.NOT_FOUND;
            return new ResponseEntity<>("text content", headers, status);
        }
        ```
    * JSONを返す
        * メソッドの戻り値を任意のクラスにすれば、SpringMVCがJacksonを使用してうまいことやってくれる。
        ```
        @RequestMapping("/json")
        public List<User> json() {
            return Arrays.asList(new User("紀伊", "太郎"), new User("紀伊", "花子"));
        }
        ```
        ```
        // 返り値
        [
            {
                "lastName": "紀伊",
                "firstName": "太郎"
            },
            {
                "lastName": "紀伊",
                "firstName": "花子"
            }
        ]
        ```
    * ファイルのダウンロード
        * よく見かけるサンプルではHttpServletResponseに直接書き込んでるものが多い。
        ```
            @RequestMapping("/file1")
            public void file1(HttpServletResponse res) throws IOException {
                File file = new File("pom.xml");
                res.setContentLength((int) file.length());
                res.setContentType(MediaType.APPLICATION_XML_VALUE);
                FileCopyUtils.copy(new FileInputStream(file), res.getOutputStream());
            }
        ```
        * 戻り値をorg.springframework.core.io.Resourceインターフェースにするともっと簡単に実装できる。
        * 戻り値がResourceだとContent-Lengthは自動的に設定してくれる。
        ```
        @RequestMapping(value = "/file2", produces = MediaType.APPLICATION_XML_VALUE)
        public Resource file2() {
            return new FileSystemResource("pom.xml");
        }

        // マークダウンファイルの場合
        @RequestMapping(value = "/file2", produces = MediaType.TEXT_MARKDOWN_VALUE)
        public Resource file2() {
            return new FileSystemResource("README.md");
        }
        ```
        * org.springframework.core.io.Resourceの実装クラスでよく使いそうなものをピックアップしておく。
            * FileSystemResource
            * ClassPathResource
            * ServletContextResource
            * ByteArrayResource
            * InputStreamResource


