- 変数宣言はvar
- defはpythonライクな関数宣言で型をしていしない、一方fnはrustライクで返り値も含めて型宣言が必要
    - fnで定義しておけばコンパイル時にエラーを吐かせて確認できる
- .mojo .🔥ではトップレベルコード（関数・クラス外で実行されるコード）を記述できないので、fnかdefでmain関数を宣言する必要がある
- 関数宣言時に引数にownedをつけると関数内でのみ変更される。またreadならimmutable,mutならmutableに、つまり関数内外問わず変更できなかったり逆に外部に対しても変更を認めることになる
- struct宣言によってオブジェクトを定義できる。pythonライクにはclassである。
    - 現在はstruct内の静的宣言のみ許しており、”””今後dynamicなpython-styleも実装予定”””

### trait
- Rustにある多態性（ポリモーフィズム）やジェネリックプログラミングを可能にするための仕組み
    - trait宣言をすると構造体に必要なメソッドを事前に宣言し継承させる
    - ジェネリック関数とは、本来関数の定義には引数の型定義が必要でありそれによって期待される型ごとに同じロジックを組む必要があるが（IntのaddとFloat54のadd）そのような事前の型定義なしに一度に関数を定義できる
    - Javaでは abstract class [抽象クラス]として知られる

### parameter
- mojoでは fn fnuc[param](msg: String): のように宣言できる。このparamは他の言語の"引数"と異なりコンパイルに関するパラメタとなる
- コンパイル時に実行内容が確定するため計算量を削減しパフォーマンスが向上する

### Python統合
- from python import Python 
    - var np = Python.import_module("numpy")
    - var ar = np.arrange(15).reshape(3,5)
        - このようにpythonのライブラリをimportすることが可能

### functions
- defは制限のない関数定義。def greet(name: String) -> String:のように型定義もできる
    - 定義しなかったらreturnはデフォルトでobject
    - このobject型は実行時に推論されるので動的型付けが可能
    - 開発中であり全ての基礎型をサポートしているわけではない
- fnはdefの上位互換
    - 引数に型を指定する必要がある
    - 戻り値も型が必要。指定しなければNone
    - 引数はデフォルトでは不変参照(read-only)
    - mut宣言をすれば可変に
    - エラー発生の可能性があれば、raises宣言をしておく
- exp: Int = 2のようにデフォルトの定義ができる
    - mut宣言されている場合はデフォルト値を置けない
    - myfoo(exp=3,base=2)のように**kwargsを利用できる（変数指定して代入）
    - sum(*values: Int) -> Int:の*valuesはvariadic arguments(可変長引数)の宣言
    - homogeneous variadic(同種可変長引数) すべての引数の型が同じ可変長引数。
    - def sum(*nums: Int):    sum(1,2,3,4)
    - 現在レジスタ渡し可能な型(Intなど)とメモリのみの型(String)で多少違いがあり将来的に直したい
- Heterogeneous variadic arguments(異種可変長)
    - traitとparametersが必要 def count_many_things[*ArgTypes: Intable](*args: *ArgTypes):
    - @parameterデコレータはコンパイル時にコードを実行。これによりコンパイル時の評価となり最終的なバイナリサイズの削減をする
        - たとえばfor i in range(4){print("i")}のコードが展開されて{print("1"),print("2")...}と異なりコンパイルに関するパラメタとなる
            - これによりforによる可読性を得つつ実行時の速度も得られる
        - またグローバル変数として定義は
        - 実行時ではなく、コンパイル時に確定する処理に有利
- variadic keyword(**kwargs)もサポート
    - fn print_nicely(**kwargs:Int):と定義するとprint_nicely(a=7,b=8)で呼び出したときにkwargs.keys()にkeyの配列が、kwargs[key[]]にvalueが入る
    - fn read_var_kwargs(read **kwargs: Int)のようにkwargsを変数名で宣言することはサポートされていない
- only arguments
    - fn min(a:Int, b: Int, /) -> Int: と定義すればpositional-onlyとなり位置引数のみで代入できる
        - 引数名が呼び出し元にとって意味がない場合や後で引数名を変更できるようにしたいとき
- overloaded functions
    - fn add(x: Int,y: Int) -> Int,fn add(x: String, y: String) -> Stringのように複数の型に対して同じ名前の関数を書ける
    - しかしこの時、この関数呼び出しを行うなら struct MyString: fn __init__(out self, string: StringLiteral): passのような構造体で呼び出し時に引数をラップする必要があるfn call_foo(): foo(MyString(string))

### Variables(変数)
- value(値)またはobjectを保持する名前。
- 基本的にMojoの変数は可変。不変にするにはaliasを使うらしい
- var宣言してもしなくても定義できる
- implicit conversion暗黙的型変換もされる
    - たとえばFloat64型に99を代入するとprint時に99.0と出力される
    - name=String("sam")もuser_id=0もそれぞれStringとinteger型に暗黙的に定義される。そのため、nameにInt、user_idにStringは代入できない
    - var user_id: Int = "Sam"のようにexplicitly(明示的)にInt宣言したものでimplicitly(暗黙的)なStringの"Sam"を代入できない
    - explicitlyに宣言すれば、initializeせずにdeclareできる。 var value: Float64
- Type annotations 型アノテーション
    - var name: String = get_name()と書くとnameがStringであることを保証する
        - get_nameではどの型が返るかわからない
- @implicitデコレータを使用すると特定の型への暗黙的な型変換を可能にしたり制限したりできる
    - struct MyInt: var value: Int @implicit fn __init__(out self,value: Int): self.value =value (@implicitをつけない)fn __init__(out self, value: Float64): self.value = Int(value)
    - fn func(n: MyInt): で func(Int(42))→〇,func(MyInt(Float64(4.2)))→〇,func(Float64(4.2))→×
    - どちらにも@implicitをつければどちらも暗黙の型変換ができる。__init__内の処理はつまり入ってきた引数に自動で行う処理
- lexical scoping→変数の範囲がプログラムの構造によって決まること
- Float64型を表すには
    - Sign(符号ビット):1ビット、Exponent(指数部):11ビット、Mantissa(仮数部):52ビットで表される
- 0x--,0o--,0b--はそれぞれ16進数(0xff)-(255)、8進数(0o77)-(63)、2進数(0b0111)-(7)を表す
- String型はvar s = "A very long string which is "
        "broken into two literals for legibility."のように定義されていた場合勝手に連結される
- 複数行の文字列にはvar s = """
Multi-line string literals let you
enter long blocks of text, including
newlines."""のように三重引用符をつかう
- Listは動的にサイズが変更される配列
- var list: List[Int] = [2,3,5]のように初期化をすることができないが var list = List(2,3,5)として定義する事はできる
- var list = List(2,3,5)をprint(list)のようなこともできないが、var l = List[Int]()と初期化しておけば可能
- Register-passableはマシンレジスタに渡すことが可能なInt,Bool,SIMD等の型でIntやBoolを含むstructも可能
    - このとき@register_passableデコレータをつかう
- Memory-only型はregister-passableに該当しないものである。それらは大抵動的に割り当てられたポインタや参照をふくむ
- AnyTypeは型の型（全ての型を含む）
- AnyTrivialRegTypeはregister-passableの型の型
#### SIMD
- 
- 
### Operator(演算子)
- () Parenthesized expression 括弧付き表現
- x[index], x[index:index] Subscripting, slicing 添え字、スライス
- ** Exponentiation 累乗
- +x,-x,~x Positive,negative, bitwise NOT 正、負、bitにおけるNOT
- *,@,/,//,% Multiplication,matrix,division,floor division, remainder
- +,- Addition and subtraction
- << >> Shifts
- & Bitwise AND
- ^ Bitwise XOR
- | Bitwise OR
- in,not in, is, is not <,<=,>,>=,!=,==
- not x Boolean NOT
- x and y Boolean NOT
- x or y Boolean AND
- Boolean OR
- if-else Conditional expression
- := Assignment expression
- @valueデコレータ
    - @valueデコレータが実装されたものは、自動的にライフサイクルメソッドを作成する
        - __init__ __copyinit__ __moveinit__
- dunder methodsは__init__などの__がつくメソッド
- 
- value ownership
    - value semantic(値セマンティック)
    - reference semantic(参照セマンティック)
- value^ 転送シジル
    - メモリの所有権を異動することを明示的にする
    - fn my_func():
        - var message: String = "Hello"
        - teke_text(message^) #ここで所有権を関数内のローカル変数に移動したのでこれ以降は使えない
    - def add_to_list(owned name: String, mut list:List[String]):
        - list.append(name^) #ここで所有権を返している
    - def consume_string(owned name: String):
        - print(s) #ここで所有権を返していないので関数終了時に値が破棄され使えなくなる

### 構造体
- struct NoInstance:
    - var state: Int
    - 
    - @staticmethod
    - fn print_hello():
        - print("hello world!")
    のようにコンストラクタ無しで構造体を定義することはできるがインスタンス化はできない。静的メソッドのNoInstance.print_hello()だけ実行可
- Mojo型のインスタンスを作成するにはdunderメソッドの__init__コンストラクタメソッドが必要
    - fn __init__(out self, name: String, age: Int):
        - self.name = name
        - self.age = age
    - fn __init__(out self):
        - self.name = ""
        - self.age = 0  #どちらも引数が与えられなかった場合のオーバーロード
    - fn __init__(out self, name: String):
        - self = MyPet()
        - self.name = name #nameだけ与えられた場合
