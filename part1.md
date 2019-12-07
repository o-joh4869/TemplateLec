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

printArray関数がテンプレート関数と呼ばれるものである。  

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
実は、
```cpp
template<class T>
```
としても正常に動作する。  
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

## tipps : 強力な型の判別

今回のmax関数のように、複数の型から一番強力なものを知りたいときがある。  
本編では明示的に指定することでごまかしたが、使うときの面倒ごとはできれば避けたいものである。  
上位変換トレイツ(Promotion Traits)と呼ばれるもので、僕自身よくわかっていないので参考文献\[1](C++テンプレート完全ガイド)を参照するといいだろう。  

# クラステンプレート

関数同様、クラスもテンプレート化できる。  

```cpp
template<typename T>
class Hoge {
public:
    Hoge(const T &t);
    T t;
    void f(const T &t);
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
```

このクラスは指定された型のメンバを一つだけ持つクラスで、引数として受け取った値だけマイナス演算子を実行するメンバ関数fを持つ。  
このクラスを使うときは、
```cpp
Hoge<int> h(1); //インスタンスの生成
h.f(); //メンバ関数fの実行
```
とすればいい。  



メンバ関数fには、マイナス演算子が使用されている。  
通常の関数テンプレートは、指定した型にマイナス演算子が定義されていなかったらコンパイルエラーになってしまうが、クラステンプレートはメンバ関数が個々にインスタンス化されるため、メンバ関数fを使用しなかったらコンパイルエラーにならない。  
つまるところ、
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

ということだ。  

