* 値をラップして、その値がnullかもしれないことを表現するクラス。
* メソッドの戻り値として、使用する
* nullの場合の処理を強制し、NullPointerExceptionの発生を防ぐことができます。

* java 11 Optionalで検索して調べよう
```
public class Test {

	public static void main(String[] args) {
		String str = null;
		System.out.println(str.length());
	}

}
```
上記のような記述の場合、NullPointerExceptionが発生してしまいエラーが起こる

* .isPresent()というメソッドがOptionalには使用できる（中に入っていますか？という確認のメソッド）
* nullかもしれないstrを処理する前に、Optionalクラスでラッピングする
* このOptionalクラスに対して、.isPresent()を使って、nullでないことがわかれば .get()でOptionalに入っている値を取り出して、処理をさせることで、実行時エラーは起きない
```
	public static void main(String[] args) {
		String str = null;
		
		Optional<String> strOpt = Optional.ofNullable(str);
		if(strOpt.isPresent()) {
			String message = strOpt.get();
			System.out.println(message.length());
		}
	}
```
* 省略して書きたい場合
    ```
            strOpt.ifPresent(v -> System.out.println(v.length()));
    ```
    * ラムダ式で、ifPresentの引数内で、trueだった場合の処理を記述できます。
    * isPresent()の中のｖはstrOpt.get()で取得できる値を格納することができます。

* .map()メソッドも使える
	* Optionalに対してラップされている値に直接変数などの処理を行えませんが、mapを使えばラップされたまま、中身に対して処理を加えることがで、中身の型も変えることができます。
	* strOpt.map(s -> s.length())という記述にすると、OptionalにマップされていたString型の値に対して、lengthを使って、int型に書き換えることができます。
	* いろいろラップされたオブジェクトに対して処理を行った後に、最後の最後でOptionalに対して.isPresent()を使って条件分岐させることができます。