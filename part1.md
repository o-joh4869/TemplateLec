# テンプレート講座

ここでは基本的なテンプレートの文法を紹介する。  

# 関数テンプレート

まずは次の処理をみてほしい。  

``` cpp
#include<iostream>

void printArrayI(int *arr, size_t size) {
    for(size_t i = 0; i < size; i++) {
        std::cout << arr[i] << "\n";
    }
}
void printArrayF(float *arr, size_t size) {
    for(size_t i = 0; i < size; i++) {
        std::cout << arr[i] << "\n";
    }
}

int main() {
    int arr[] = {
        10,
        20,
        30
    };
    float arr2[] = {
        3.14f,
        2.71f
    };

    printArrayI(arr, 3);
    std::cout << "\n";
    printArrayF(arr2, 2);

    return 0;
}
```

適当な配列を用意し、表示するだけの処理だ。  

int型の配列用のprintArrayI関数とfloat型用のprintArrayF関数を用意した。  
しかしこれらの中身は同じで処理の内容もほぼ同等である。  
関数のオーバーロードを利用しても書く手間は大きくは変わらない。  

中身は全く同じことを書いているのにすべての型に対して定義を書くのは面倒だ。  
ここでテンプレートが登場する。  

```cpp
#include<iostream>

template<typename T>
void printArray(T *t, size_t size) {
    for(size_t i = 0; i < size; i++) {
        std::cout << t[i] << "\n";
    }
}

int main() {
    int arr[] = {
        10,
        20,
        30
    };
    float arr2[] = {
        3.14f,
        2.71f
    };

    printArray<int>(arr, sizeof(arr) / sizeof(arr[0]));
    std::cout << "\n";
    printArray<float>(arr2, sizeof(arr2) / sizeof(arr2[0]));
}
```

printArray関数が関数テンプレートと呼ばれるものである。  

関数の定義の前に
```cpp
template<typename T>
```
と書かれている。  
これによってprintArray関数の仮引数tの型をオープンなまま残すことができる。  
このTをテンプレートパラメータと呼ぶ。  
また、仮引数のことを呼び出しパラメータと呼ぶこともある。  

この関数を利用するときは、  
```cpp
printArray<int>(arr, 3);
```
のように関数名の後ろに<>で型を指定して、通常の関数同様引数を指定する。  
コンパイラは定義されていなかった型Tをintとして関数の定義を作成する。  
このように、関数が作成されることをインスタンス化という。  
※クラスのインスタンスとは異なる  
また、インスタンス化された関数を特殊化という(後に登場するクラスも同様)。  

今回のように引数からすべてのテンプレートパラメータが推測できるときは  
```cpp
printArray(arr, 3);
```
としても適切にコンパイルされる。  

## Tipps : typename

テンプレートパラメータの宣言のときにtypenameキーワードを用いた。  
typenameキーワードは歴史が浅く、当初はclassをキーワードとして用いた。  
現在でも、
```cpp
template<class T>
```
というコードは正常に動作する。  
classとするとint型などの基本型の代入の際に誤解を招く。  
そのため、typenameキーワードが後から導入された。  

# 複数のテンプレートパラメータ

テンプレートパラメータを複数にすることができる。  

```cpp
template<typename T, typename U>
T max(const T &t, const U &u) {
    return t > u ? t : u;
}

void main() {
    int a = 10;
    float b = 20.5f;

    std::cout << max(a, b) << std::endl;
}
```

これは引数に渡されたもののうち、大きい方を返すものだ。  
関数の引数を増やすときのように,で区切って
```cpp
template<typename T, typename U>
```
とすればいい。  
三つ以上についても同様だ。  
すべてのテンプレートパラメータをまとめてテンプレートパラメータリストと呼ぶ。  

上のコードでは、int型とfloat型がTとUに対応して処理され20が出力される。  

このコードには問題があることに気が付いただろうか。  
返り値が先に指定した型にキャストされてしまう点だ。  
10と20.5fを比べたとき、20.5fを返すべきだがint型にキャストされてしまい20が返される。  
使うときにより強力な型を先に渡すというルールを設けてもいいが、ルールを無視しても動作してしまうためバグが発生する恐れがある。  

これを解決するためにテンプレートパラメータを増やして解決できる。  

```cpp
template<typename RT, typename T, typename U>
RT max(const T &t, const U &u) {
    return t > u ? t : u;
}

int main() {
    int a = 10;
    float b = 20.5f;
    double c = max<double>(a, b);
}
```

返り値の型用のテンプレートパラメータを増やした。  
返り値の型を指定しなければコンパイルエラーになるため意図しない型変換を回避できる。  

一方、max関数のテンプレートパラメータは3つなのに対し、型の指定はdoubleのみである。  
実用上二つのパラメータの最大を取りたい時、毎回引数に渡す変数の型まで明示的に指定するのは面倒だ。  
そのため、残りの二つを引数から推論させた。  
今回のように一部のテンプレートパラメータのみを指定したい時は、推論させる部分をパラメータリストの後ろに持っていく必要がある。  

C++14以降では、autoを使って  
```cpp
template<typename T, typename U>
auto max(const T &t, const U &u) {
    return t > u ? t : u;
}
```
とできる。  

# クラステンプレート

関数同様、クラスもテンプレート化できる。  

```cpp
template<typename T>
class Hoge {
public:
    Hoge(const T &t);
    T t;
    void f(const T &t);
    void g(const Hoge<T> &h);
};

template<tyoename T>
Hoge::Hoge(const T &t) : 
    t(t)
{}

template<typename T>
void Hoge<T>::f(const T &t) {
    this->t - t;
    return;
}

template<typename T>
void g(const Hoge<T> &h) {
    t = h.t;
}
```

まず、クラスの定義の前に
```cpp
template<typename T>
```
と書く。  

各メンバは、Tを任意の型として使うことができる。  
例えば、その型の実体をメンバに持たせるときは
```cpp
T t;
```
とする。  
各メンバ関数の宣言も同様である。  

メンバ関数の宣言と定義を分ける場合は定義の書き方に少し変更がある。  
すべてのメンバ関数の定義の前に
```cpp
template<typename T>
```
と書かなければならない。  
パラメータリストはクラスの定義と一致させる必要がある。  
コンストラクタやデストラクタ等も同様である。  

また、コンストラクタやデストラクタの名前以外で自分自身のクラスの型を用いる場合は型指定も含めて
```cpp
Hoge<T>
```
としなければならない。(Hoge::gの引数を参照)  

そしてこのクラスを使うときは、
```cpp
Hoge<int> h(1); //インスタンスの生成
h.f(); //メンバ関数fの実行
```
とする。  

メンバ関数fには、マイナス演算子が使用されている。  
関数テンプレートは、指定した型にマイナス演算子が定義されていなかったらコンパイルエラーになってしまう。  
クラステンプレートは各メンバ関数が使用された時にその関数のみをインスタンス化する。  
つまり、マイナス演算子が定義されていない型をテンプレート引数に渡したクラスHogeを作成しても、メンバ関数fを使用しなければ正常に動作する。  

```cpp
class Fuga {
};

int main() {
    //コンパイルエラーにならない
    Hoge<Fuga> h;

    //コメントを外すとコンパイルエラー
    //h.f();
}
```

## Tipps : テンプレート引数にクラステンプレートを渡す

vectorのテンプレート引数にint型のvectorを渡す場合を考える。  

```cpp
std::vector<std::vector<int>> obj;
```

比較的新しいバージョンのC++ではこれで問題なくコンパイルできる。  
しかし古いコンパイラではintの後の>>がシフト演算子にみなされてコンパイルエラーになってしまう。  
そのときは
```cpp
std::vector< std::vector<int> > obj;
```
のように適宜スペースを入れる必要がある。  

# 特殊メンバ関数のテンプレート

メンバ関数も関数テンプレート同様にテンプレート化することができる。  
大方通常の関数と同様だが、コンストラクタなどの特殊メンバ関数は事情が異なる。  

## コンストラクタテンプレート
コンストラクタをテンプレート化した場合、テンプレート引数の明示的指定ができないため引数から推論できるもののみ使用可能である。  
暗黙に型変換が定義された型同士のコピーを許すときなどによく使われる。  
```cpp
template<typename T>
class Hoge {
public:
    template<typename U>
    Hoge(const U &_a) : 
        a(_a)
    {}
    
    template<typename U>
    Hoge(const Hoge<U> &o) : 
        a(o.a)
    {}

private:
    T a;
};
```

## 演算子オーバーロード
演算子もテンプレート化できる。  
コンストラクタ同様、引数から推論する以外にテンプレートパラメータを指定できない。  
```cpp
template<typename T>
class Hoge {
public:
    Hoge(const T &t) : 
        a(t)
    {}

    template<typename U>
    Hoge operator+(const U &u) {
        return Hoge(a + u.a);
    }
private:
    T a;
};
```

## デストラクタ
デストラクタは引数がないのでテンプレート化できない。  
(そもそもデストラクタテンプレートとは……？？って感じですしおすし)  

# ノンタイプテンプレートパラメータ

テンプレート引数には型以外に値を渡すこともできる。  
```cpp
#include<iostream>

template<typename T, int size>
T *makeArray() {
	return new T[size];
}

int main() {

	int *p = makeArray<int, 5>();

	delete[] p;

	return 0;
}
```

関数やクラスのテンプレートリストに、関数の引数を増やすときのように
```cpp
template<int size>
```
とすることで、関数内でint型のsizeを使うことができる。  

使うときはコンパイル時に値が決まっている必要があるため、constやconstexprで修飾された変数やリテラル値などしか扱うことができない。  
(さらに言えばconstで修飾されたものも使いようによっては危険なので、リテラル値以外を用いるときはconstexprにしておくと安全)  

コンパイル時にリテラル値に置き換えたようにできるため、静的な固定長配列を作成することができる。  

double型やfloat型といった実数型はノンタイプテンプレートパラメータとして使用することができない。  
整数型以外で利用できるものもあるのでので調べてみるといいかもしれない。  

# 明示的特殊化

関数テンプレートやクラステンプレートで、特定の型においてのみ別の挙動をさせることができる。  
```cpp
//通常の関数テンプレート
template<typename T, typename U>
auto max(const T &t, const U &u) {
    return t > u ? t : u;
}

//明示的(完全)特殊化
template<>
bool max<bool, bool>(bool t, bool u) {
    return t || u;
}

//明示的部分特殊化
template<typename T, typename U>
auto *max(const T *t, const U *u) {
    return (*t) > (*u) ? t : u;
}
```

明示的(完全)特殊化は、
```cpp
template<typename T>
```
としていたところを
```cpp
template<>
```
として、その後に記述する関数名やクラス名に
```cpp
bool f<bool>() {
}
```
としてパラメータリストを指定し、その定義を書く。  

ソースコードではmax関数をboolで明示的(完全)特殊化した。  
bool型二つの比較はtrueがあればtrue、無ければfalseを返せばよい。  
したがって、or演算子(||)を用いた定義をした。  

今回の場合、引数からテンプレートパラメータをすべて推論できるので
```cpp
template<>
bool max(bool t, bool u) {
}
```
としてもいい。  

部分的特殊化は、部分的にテンプレートパラメータを残して定義する。  
```cpp
template<typename T, typename U>
T *max(const T *t, const U *u) {
}
```
これは両方の引数がポインタ型だった場合に呼ばれる。  

また、クラステンプレートのメンバ関数も特殊化することができる。  
クラステンプレートはメンバ関数が別々にインスタンス化されるので、一部メンバ関数のみを特殊化することもできる。  
```cpp
template<typename T>
class Hoge {
public:
    void f();
    void g();
};

template<typename T>
void Hoge<T>::f() {
}

template<>
void Hoge<bool>::f() {
}

template<typename T>
void Hoge<T>::g() {
}
```

## Tipps : ファイル分けと特殊化
テンプレートのファイル分けは面倒なことが絡むため後に詳しく紹介する。  
今はすべての明示的特殊化も含めて1つのヘッダーファイルにすべて書くことを勧める。  

# テンプレートテンプレートパラメータ

テンプレート引数にテンプレートクラスが指定されることがわかっているとき、テンプレートパラメータの指定の仕方を変更して引数に渡せる型を制限することができる。  
```cpp
template<typename T, template<typename ELEM> class CONT>
class Hoge {

};

template<typename T>
class Fuga {

};

int main() {
    Hoge<int, Fuga> h;
}
```

クラステンプレートHogeのパラメータリストの二番目のものが、テンプレートテンプレートパラメータである。  

```cpp
template<template<typename ELEM> class CONT>
```
と書く。  

引数に渡したクラステンプレートはCONTに対応して、渡したクラスのパラメータリストがELEMに対応する。  

CONTのパラメータリストであるELEMはHogeクラス内で利用できるものではなく、テンプレートパラメータの数や種類が分かれば十分なので  
```cpp
template<template<typename> class CONT>
```
ともできる。  

テンプレートテンプレートパラメータに指定するパラメータリストは、引数に渡すテンプレートクラスのパラメータリストと、デフォルトテンプレート引数で補完される部分も含めて完全に一致する必要がある。  
つまり、テンプレートテンプレートパラメータの引数にvectorなどを渡したい時は、
```cpp
template<typename T, template< typename ELEM, typename ALLOC = std::allocator<ELEM> > class CONT = std::vector>
class Hoge {

};
```
としてアロケータ部分も書く必要がある。  

これも前述したようにテンプレートパラメータを省略して  
```cpp
template<typename T, template<typename, typename> class CONT = std::vector>
```
とすることができる。  
しかし、このクラスの中でCONTを利用する場合、毎回CONTの第二引数に```std::allocator<T>```と書かなければいけないのは実用上面倒である。
デフォルト引数が指定されているクラステンプレートをテンプレートテンプレートパラメータで想定するときはデフォルト引数まで書くことを勧める。  

# 可変引数テンプレート
引数を可変長にしたい場面がしばしばある。  
そこで、指定されたクラスについてnewしてそのポインタを返すmake関数を例に紹介する。  
```cpp
template<typename T, typename... Args>
T *make(Args... args) {
	return new T(args...);
}
```

可変テンプレートパラメータは
```cpp
template<typename... Args>
```
や
```cpp
template<class... Args>
```
と書く。  

非可変テンプレートパラメータがある場合は、可変テンプレートパラメータが最後になるようにしなければならない。  
```cpp
template<typename T, typename... Args>
```

複数(0個を含む)のパラメータを持つArgsやargsをパラメータパックという。  
このパラメータパックを展開するには
```
args...
```
とする。  

上の関数では展開した値をそのままTのコンストラクタに渡している。  

基本的にパラメータパックをそのまま用いることはなく、関数に渡してしまうことが多い。  

一つ一つの値を用いて何かをしたい場合、別の関数テンプレートを用いて再帰的に解決できる。  
```cpp
//可変引数の最大値を返す関数
template<typename T, typename... Args>
const auto &max(const T &t, const Args&... args) {
	const auto &u = max(args...);
	return t > u ? t : u;
}

//二つの引数の大きい方を返す関数
template<typename T, typename U>
const auto &max(const T &t, const U &u) {
	return t > u ? t : u;
}

//可変部分が0個だった場合
template<typename T>
const auto &max(const T &t) {
	return t;
}
```

# 演習問題
以下の条件でqueue的な固定長配列クラスを作成し挙動を確認せよ。  
・要素の管理はC配列でおこなう。(std::arrayの方が好ましい)  
・要素番号を引数としてもらってその要素への参照を返すメンバ関数atを作成する。(operator[]でもいい)  
・先頭に要素を追加し、同時に末尾の要素を削除するメンバ関数pushを作成する。  

    ヒント  
    見かけ上の先頭の要素と実際の配列に入っている要素の先頭は異なる。  
    見かけ上の先頭の要素が配列の何番目にあるかを変数に格納する。  
    atではそこからの番号の要素を参照、pushでその番号を一つずらす。  

# 発展問題  
演習問題で作ったクラスを以下の要件で拡張せよ。  
1. at関数やoperator[]で配列外参照したときにout_of_rangeをthrowする。  
2. 要素を管理するクラスを指定できるようにして、デフォルトをstd::vectorにする。  
その際、テンプレートテンプレートパラメータを用いること。  
挙動の確認にはstd::vectorとstd::dequeを用いるとよい。  
3. 動的に要素数が指定できるようにコンストラクタを変更する。  
4. std::listが二つ目のパラメータに指定されたときも同様に動作するように部分特殊化する。  
5. この方法では静的に要素数が決定する場合の特殊化を十分に実現できないため、動的静的の双方で要素数を指定できる方法を実装する。  
6. 静的な場合の要素数が2個だったときについて特殊化し、見かけの先頭要素の配列番号をbool型で管理する。 　