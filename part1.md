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

    printArrayI(arr);
    std::cout << "\n";
    printArrayF(arr2);

    return 0;
}
```

適当な配列を用意し、表示するだけの処理だ。  

このときint型の配列用のprintArrayI関数とfloat型用のprintArrayF関数を用意した。  
しかしこれらの中身は同じで処理の内容もほぼ同等である。  
関数のオーバーロードを利用しても書く手間は大きくは変わらない。  

中身は全く同じことを書いているのにすべて定義しなおすのはやや面倒だ。  
ここでテンプレートが登場する。  

```cpp
#include<iostream>

template<typename T>
void pringArray(T *t, size_t size) {
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

関数の前に
```cpp
template<typename T>
```
と書くことでTという識別子をあいまいな型として扱うことができる。  
このTをテンプレートパラメータと呼ぶ。  

この関数を利用するときは、  
```cpp
printArray<int>(arr);
```
のように関数名の後ろに<>で型を指定して、通常の関数同様引数を指定する。  
こうすることで、コンパイラがあいまいな型Tをintに補完して関数を作成される。  
このように、関数が作成されることをインスタンス化という。  
クラスのインスタンスとは違うので注意すること。  

今回のように引数から型が推測できるときには  
```cpp
printArray(arr);
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
classとすると、int型などの基本型の代入の際に誤解を招く。  
そのため、typenameキーワードが後から導入された。  

# 複数のテンプレートパラメータ

テンプレートパラメータは複数にすることもできる。  

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

上のコードでは、int型とfloat型がTとUに対応して処理され、20が出力される。  

このコードには問題があることに気が付いただろうか。  
返り値が先に指定した型にキャストされてしまうのだ。  
今回でいえば、20.5を返してほしいのにint型にキャストされてしまい20が返される。  
使うときにより強力な型を先に渡すというルールを設けてもいいが、ルールを無視しても動作してしまうためバグが発生する恐れがある。  

これを解決するためにテンプレートパラメータを増やして使うたびに指定することにする。  

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

テンプレートパラメータが3つなのに対し、型の指定はdoubleのみである。  
これは、残りの二つをコンパイラに推論させる。  
今回は指定したいものは返り値の型のみなので、残りの二つをパラメータリストの後ろにした。  

実際に使う上では、異なる型のものをくらべること自体ナンセンスなので、一つの型で
```cpp
template<typename T>
const T &max(const T &t1, const T &t2) {
    return t1 > t2 ? t1 : t2;
}
```
として、指定したいときだけ指定してそれ以外は推論させるのがいいだろう。  

## tipps : 強力な型の判別

今回のmax関数のように、複数の型から一番強力なものを知りたいときがある。  
本編では明示的に指定することでごまかしたが、使うときの面倒ごとはできれば避けたいものである。  
上位変換トレイツ(Promotion Traits)と呼ばれるもので無理やり解決できるが、僕自身よくわかっていないので参考文献\[1](C++テンプレート完全ガイド)を参照するといいだろう。  

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

各メンバは、このあいまいな型Tを使って記述することができる。  
その型の実体をメンバ変数として持たせる場合は、通常通り
```cpp
T t;
```
とすればいい。  
各メンバ関数の宣言も同様である。  

しかし、メンバ関数の宣言と定義を分ける場合は、定義の書き方には少し変更がある。  
すべてのメンバ関数の定義の前に
```cpp
template<typename T>
```
と書かなければならない。  
当然、パラメータリストはクラスの定義と一致させる必要がある。  
コンストラクタやデストラクタ等も同様である。  

また、コンストラクタやデストラクタの名前以外で自分自身のクラスの型を用いる場合は、型指定も含めて
```cpp
Hoge<T>
```
としなければならない。(Hoge::gの引数を参照)  

そしてこのクラスを使うときは、
```cpp
Hoge<int> h(1); //インスタンスの生成
h.f(); //メンバ関数fの実行
```
とすればいい。  

メンバ関数fには、マイナス演算子が使用されている。  
通常の関数テンプレートは、指定した型にマイナス演算子が定義されていなかったらコンパイルエラーになってしまうが、クラステンプレートはメンバ関数が個々にインスタンス化されるため、メンバ関数fを使用しなかったらコンパイルエラーにならない。  
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
std::vector< std::vector<int> >
```
のように適宜スペースを入れる必要がある。  

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

使うときはコンパイル時に決まった定数である必要があるため、constやconstexprで修飾されたものしか扱うことができない。  
(さらに言えばconstで修飾されたものも使いようによっては危険なので、リテラル値以外を用いるときはconstexprにしておくと安全)  

コンパイル時にリテラル値に置き換えたようにできるため、静的な固定長配列を作成することができる。  

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

パラメータリストの二番目のものが、テンプレートテンプレートパラメータである。  

```cpp
template<template<typename ELEM> class CONT>
```
のように書く。  

引数に渡したテンプレートクラスはCONTに対応して、渡したクラスのパラメータリストがELEMに対応する。  

このELEMは便宜上書いただけの物であり、Hogeクラス内で利用できるものではないので、
```cpp
template<template<typename> class CONT>
```
として、型パラメータの数だけ指定すれば十分である。  

こうすることで、第二引数に渡せるクラスはテンプレートリストが型パラメータ一つのみのものに制限される。  

このとき、テンプレートテンプレートパラメータに指定するパラメータリストは、引数に渡すテンプレートクラスのパラメータリストと、デフォルトテンプレート引数で補完される部分も含めて完全に一致する必要がある。  
例えばテンプレートテンプレートパラメータの引数にvectorなどを渡したい時は、
```cpp
template<typename T, template< typename ELEM, typename ALLOC = std::allocator<ELEM> > class CONT = std::vector>
class Hoge {

};
```

のようにする必要がある。  

これも当然
```cpp
template<typename T, template<typename, typename> class CONT = std::vector>
```
とすることができる。  
しかし、このクラスの中でCONTのインスタンスを生成する場合などに、毎回CONTの第二引数にstd::allocator<T>と書かなければいけないのは実用上面倒なので、デフォルト引数が指定されているものに関しては書くことを勧める。  

# 演習問題

1. テンプレートパラメータを一つもち、デフォルトコンストラクタでnewしてそのポインタを返す関数makeを作り挙動を確認せよ。  
受け取ったポインタをdeleteするのを忘れないこと。  
2. 以下の条件でqueue的な固定長配列クラスを作成し挙動を確認せよ。  
・要素の管理はvectorでおこなう。(std::arrayの方が好ましい)  
・要素番号を引数としてもらってその要素への参照を返すメンバ関数atを作成する。(operator[]でもいい)  
・先頭の要素を削除して同時に末尾に要素を追加するメンバ関数updateを作成する。  

    ヒント  
    見かけ上の先頭の要素と実際のvector等の配列に入っている要素の先頭は異なる。  
    見かけ上の先頭の要素が配列の何番目にあるかを変数に格納する。  
    atではそこからの番号の要素を参照、updateでその番号を一つずらす。  

発展問題  
    2で作ったクラスを以下の要件で拡張し挙動を確認せよ。  
    ・at関数やoperator[]で配列外参照したときにout_of_rangeをthrowする。  
    ・要素を管理するクラスを指定できるようにして、デフォルトをstd::vectorにする。  
    その際、テンプレートテンプレートパラメータを用いること。  
    挙動の確認にはstd::vectorとstd::dequeを用いるとよい。  
