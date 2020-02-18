# クラスの宣言と定義
クラスの宣言はやや特殊で
```cpp
//宣言
class Hoge;

//定義
class Hoge {
};
```

と書く。  
クラスの宣言をしておくとそのクラスのポインタ型と参照型は利用できるが、実体を持つことはできない。  
```cpp
class Hoge;

int main() {
    //OK
    Hoge *p = nullptr;

    //↓コメントアウトを外すとコンパイルエラー
    //Hoge o;

    return 0;
}

class Hoge {

};
```

# extern templateの不穏な性質

extern templateは聞いた限りまともそうな顔をしているが、少し不穏な性質も持っている。  
次のコードを見てほしい。  

```cpp
/* make.hpp */
template<typename T>
T *make() {
    return new T();
}
```
```cpp
/* Hoge.hpp */
class Hoge {
public:
    Hoge();
private:
    int a;
};
```
```cpp
/* Hoge.cpp */
#include"make.hpp"
Hoge::Hoge() : 
    a(0)
{
}

template Hoge *make<Hoge>();
```
```cpp
/* main.cpp */
#include<iostream>
#include"make.hpp"

class Hoge;
extern template Hoge *make<Hoge>();

int main() {
    Hoge *p = make<Hoge>();
    std::cout << (void*)p << std::endl;
}
```

このコードは正常に動作し、nullptrでないポインタが得られる。  
newしたままdeleteされてないので不適切なコードだが、注目すべきはmain.cppの中身だ。  
main.cpp内ではHogeクラスに関する情報は名前しかない。  
そのため通常Hogeに関するnewは呼び出せない。  
しかし、extern templateを使用するとmain.cppのコンパイルではインスタンス化されないので、通常の関数の宣言のみが存在している状態になりコンパイルが通ってしまう。  
Hoge.cppの中で明示的インスタンス化をしているのでリンクも成功する。  
結果、正常に動作してしまう。  

名前以外の情報がない状態からインスタンスが生成できてしまうのは何とも言えない不気味さがある。  

# 静的map

クラスの宣言のみでもテンプレート引数に渡すことができる性質を利用して静的なmapのようなものを実装することができる。  

```cpp
/* GetClassName.hpp */
#include<string>
template<typename T>
std::string GetClassName();
```
```cpp
/* Hoge.hpp */
#include<string>
class Hoge{
}
```
```cpp
/* Hoge.cpp */
#include"Hoge.hpp"
#include"GetClassName.hpp"

////////////////////////////////
//  Hogeのメンバ関数の定義など  //
////////////////////////////////

//明示的完全特殊化
//Hogeというクラスをキーとしてstring型のものを得る関数
template<>
std::string GetClassName<Hoge>() {
    return "Hoge";
}
```
```cpp
/* main.cpp */
#include<iostream>
#include"GetClassName.hpp"

class Hoge;

int main() {
    //Hogeというクラス名をキーにしてstring型のmap先を取得
    std::cout << GetClassName<Hoge>() << std::endl;

    return 0;
}
```
