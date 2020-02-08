# テンプレート講座
少しの新しい文法を用いてやや特殊なテンプレートの使い方を紹介する。  

# ファイル分け
テンプレートを含むもののファイル分けは通常可能なcppファイルとヘッダーファイルにそれぞれ定義と宣言を書く方法が使えない。  
```cpp
/* f.hpp */
template<typename T>
void f(const T &t);
```
```cpp
/* f.cpp */
#include<iostream>
#include"f.hpp"
template<typename T>
void f(const T &t) {
    std::cout << t << std::endl;
}
```
```cpp
/* main.cpp */
#include"f.hpp"
int main() {
    int a = 10;
    f(a);
}
```
通常の方法でこうファイル分けすると、テンプレート関数fは作成されず、リンカエラーを吐く。  
とりあえずエラーを避けるにはすべてヘッダーファイルに記述して、
```cpp
/* f.hpp */
template<typename T>
void f(const T &t) {
    std::cout << t << std::endl;
}
```
とすればいい。  
大抵の場合はこの方法で解決することが多いし、これで特に問題がないならこうするべきだ。  
しかし、ヘッダーファイルにすべて書いてしまうと、内容が変更されたときインクルードしたcppファイルすべてがリコンパイルになってしまいコンパイル時間の増加に繋がる。  
これはできるだけ避けるべきことであり、頻繁に変更されうるものについては扱いを考えなければならない。  
そこで別の方法を紹介する。  
まずはその準備として定義と宣言について簡単にまとめた。  

# 定義と宣言
宣言とは、
```cpp
void f();
extern int a;
```
といった、コンパイラにそのものの存在を伝えるだけのもので実体を持たない。  
定義は、  
```cpp
void f() {
    //
    return;
}
int a = 0;
```
のように実体生成する手順をコンパイラに伝えるものである。  

プログラムのどこかに少なくとも一度必要で、全くないとエラーになってしまう。  
関数やクラスは内容が全く同じであることを前提とした上で複数定義があることも許されているが、基本は定義は一つのみである。  

関数の呼び出し時は関数の中身は必要なく、宣言のみがあればいいので
```cpp
/* f.hpp */
#pragma once
void f();
```
```cpp
/* f.cpp */
void f() {
    //
    return;
}
```
```cpp
/* main.cpp */
#include"f.hpp"
int main() {
    f();
    return 0;
}
```
といったファイル分けをすることが多い。  

コンパイル前にファイルの中身をそのまま張り付けるincludeの仕様を把握していれば
```cpp
/* main.cpp */
void f();
int main() {
    f();
    return 0;
}
```
といった記述もできるが、関数の引数や返り値は変わることがありうるのでヘッダーファイルを用いるのが安全だろう。  

クラスの宣言はやや特殊で
```cpp
//宣言
class Hoge;

//定義
class Hoge {
};
```

と書く。  
クラスの宣言のみでできることは、そのクラスのポインタ変数を作成することと、関数の引数や返り値の宣言に利用できる程度だ。(関数の実行はできない)  
つまり、実体が必要ない処理のみ行うことができる。  

# コンパイラとリンカ
コンパイルは各cppファイルごとに行われ、オブジェクトファイル(.o)が生成される。  
VSなどのIDEではこれをまとめて実行ファイル(.exe)を作る。  
実行ファイルを作るときに関数の中身と結びつける処理が発生する。  
これを行うものをコンパイラに対してリンカといい、そこで起きるエラーをリンカエラーという。  
リンカも含めてコンパイラという場合もあるが、ここでは明確に区別する。  
上の例でいえば、main.cppをコンパイルしたときは関数fの中身は知らないので、どこかにある定義と結びつける処理をリンカに投げる。  
f.cppのコンパイルも終わった後、リンカがmain.oの知らない関数fの中身が書かれたf.oの処理と結びつけて実行ファイルを作成する。(若干語弊があるがここでは気にしないことにする)  

この仕組みを利用してヘッダーファイルに定義を書かないテンプレートの用法を見ていく。  

# リンカエラーの原因
最初のソースコードに戻ろう。  
```cpp
/* f.hpp */
template<typename T>
void f(const T &t);
```
```cpp
/* f.cpp */
#include<iostream>
#include"f.hpp"
template<typename T>
void f(const T &t) {
    std::cout << t << std::endl;
}
```
```cpp
/* main.cpp */
#include"f.hpp"
int main() {
    int a = 10;
    f(a);
}
```
最初にもいった通り、このコードはリンカエラーが発生する。  

main.cppのコンパイルはうまくいく。  
どこかにint型でインスタンス化されたテンプレート関数fが存在すると信じてコンパイルを終了する。  

f.cppのコンパイルも一応はうまくいく。  
コンパイラは関数テンプレートの定義を見てコード上の問題がないことを確認して、そのcppファイル内でその関数が使用されてないためインスタンス化が行われずコンパイルが終了する。  

次にリンカの処理に移り、main.oの知らないint型の関数テンプレートfの定義を探すが見つからずリンカエラーを吐く。  

エラーの原因になっているのは関数テンプレートのインスタンス化が正しく行われていないことであり、cppファイルで分けてしまってはどんな型に対してもうまくいかない。  

一方で定義もヘッダーファイルに書いてしまえば、main.cppの方で正常にインスタンス化が行われ正常に動作する。  

# 明示的インスタンス化

この問題を解決するには明示的にインスタンス化する方法がある。  

```cpp
/* f.hpp */
template<typename T>
void f(const T &t);
```
```cpp
/* f.cpp */
#include<iostream>
#include"f.hpp"
template<typename T>
void f(const T &t) {
    std::cout << t << std::endl;
}

template void f<int>(const int &);
```
```cpp
/* main.cpp */
#include"f.hpp"
int main() {
    int a = 10;
    f(a);
}
```

f.cppの最後に
```cpp
template void f<int>(const int &);
```
が追加されただけである。  
この行により、使用の有無にかかわらずコンパイラはint型の関数テンプレートをインスタンス化してくれる。  

このほかに、明示的完全特殊化したときにも暗黙にインスタンス化されている。  

今のところ、こうするメリットは少ない。  
わざわざ使う度にf.cppにインスタンス化の処理が書かれているかを調べて無ければ追加しなければいけない。  
これではテンプレートを用いる利点が大きく減ってしまう。  
次のextern templateと合わせて使うことでこの機能がある程度有用に見えるだろう。  

# extern template

明示的インスタンス化が存在するので、明示的非インスタンス化も可能である。  
すなわち、テンプレート関数やテンプレートクラスを使用しているにもかかわらず、そのcppファイルではインスタンス化しないことができる。  

```cpp
/* f.hpp */
#include<iostream>
template<typename T>
void f(const T &t) {
    std::cout << t << std::endl;
}
```
```cpp
/* main.cpp */
#include"f.hpp"
//int型ではインスタンス化しない
extern template void f<int>(const int &);

int main() {
    int a = 10;
    f(a); //インスタンス化されない
}
```

こうすることで、main.cppのコンパイル時にint型のfはインスタンス化されない。  
当然このままでは定義が存在せずリンカエラーを起こすので
```cpp
/* f.cpp */
#include"f.hpp"
template void f<int>(const int &);
```
のように、どこかでインスタンス化が必要である。  

こうするメリットの一つとして、ファイルサイズを抑えることができるというものがある。  
例えば、多くのcppファイルでf.hppをインクルードしint型でインスタンス化を行ったとする。  
このとき、各cppファイルをコンパイルするときにそれぞれでインスタンス化されてしまう。  
そのため、内容が全く同じ関数が大量に作られ、実行ファイルが肥大化してしまう。  
あらかじめ多くの場面で使われることがわかっている場合には、ヘッダーファイルにexternでint型を指定しておき、f.cppで明示的インスタンス化を行うことで解決できる。  

Vector2やMatrixなどは、int,double,floatなどの基本型のもので多く使われることが予想されるためこの手法がよく使われる。  

## Tips : inline展開と非インスタンス化
非インスタンス化した場合、翻訳単位の外に定義を求めるため関数のインライン展開を阻害してしまう。  
今回のように簡単な関数についてはinline宣言をしてヘッダーファイルに書いてしまうのがベストだろう。  

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
std::string &&GetClassName<Hoge>() {
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
