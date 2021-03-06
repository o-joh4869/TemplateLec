# テンプレート講座
新しい文法を導入しつつ関数テンプレートのファイル分けについて紹介する。  

# ファイル分け
関数テンプレートのファイル分けは通常可能なcppファイルとヘッダーファイルにそれぞれ定義と宣言を書く方法が使えない。  
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
通常の関数のファイル分けと同様にテンプレート関数fをファイル分けをすると、インスタンス化されずにリンカエラーを吐く。  
代わりにすべてヘッダーファイルに記述して、  
```cpp
/* f.hpp */
template<typename T>
void f(const T &t) {
    std::cout << t << std::endl;
}
```
とすれば適切に動作する。  
大抵の場合はこの方法で解決することが多く、これで特に問題がないならこうするべきだ。  
しかしヘッダーファイルにすべて書いてしまうと、内容が変更されたときインクルードしたcppファイルすべてがリコンパイルされコンパイル時間の増加に繋がる。  
(多くのファイルにインクルードされるリスクの大きさをインクルードコストということもある)  
これはできるだけ避けるべきことであり、頻繁に変更されうるものについては扱いを考えなければならない。  

そこで別の方法を紹介する。  
まずはその準備として定義と宣言について簡単にまとめた。  

# 定義と宣言
宣言は、  
```cpp
void f();
extern int a;
```
といった、コンパイラに変数や関数などの存在を伝えるだけのもので実体を持たないものである。  

定義は、  
```cpp
void f() {
    //
    return;
}
int a = 0;
```
のように実体生成する手順をコンパイラに伝えるものである。  

定義はプログラムのどこかに少なくとも一度必要で、全くないとエラーになってしまう。  
関数やクラスは定義が全く同じであることを前提とした上で翻訳単位をまたいで複数書くことが許されているが、基本的に定義は一つのみである。  

関数の呼び出し時は関数の定義は必要なく、宣言のみがあればいいので
```cpp
/* f.hpp */
#pragma once
void f();
```
```cpp
/* f.cpp */
void f() {
    //定義
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
といった記述もできるが、関数の引数や返り値は変わりうるのでヘッダーファイルを用いるのが安全だろう。  

# コンパイラとリンカ
コンパイルは翻訳単位(通常cppファイル1つが1単位)ごとに行われ、オブジェクトファイル(.o)が生成される。  
VSなどのIDEではこれをまとめて実行ファイル(.exe)を作る。  
実行ファイルを作るときに関数の中身と結びつける処理が発生する。  
これを行うものをコンパイラに対してリンカといい、そこで起きるエラーをリンカエラーという。  
非テンプレート関数の例でいうと、main.cppをコンパイルしたときは関数fの中身は知らないので、どこかにあるfの定義と結びつける処理をリンカに投げる。  
f.cppのコンパイルも終わった後、リンカがmain.oの知らない関数fの中身が書かれたf.oの処理と結びつけて実行ファイルを作成する。  
(若干語弊があるがここでは気にしないことにする)  

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
どこかにint型で特殊化されたテンプレート関数fが存在すると信じてコンパイルを終了する。  
この時点で関数```f<int>```の定義は存在しない。  

f.cppのコンパイルも一応はうまくいく。  
コンパイラは関数テンプレートの定義を見てコード上の問題(;がないなど)がないことを確認する。  
確認が終わったら一度関数テンプレートから離れ、別の箇所のコンパイルを進める。  
しかし、cppファイル内でこの関数テンプレートが使用されていないためインスタンス化が行われずコンパイルが終了する。  

リンカの処理に移るが、main.oの知らない```f<int>```の定義を探すが見つからずリンカエラーを吐く。  

エラーの原因は関数テンプレートのインスタンス化が正しく行われていないことであり、cppファイルで分けてしまってはどんな型に対してもうまくいかない。  

一方で定義もヘッダーファイルに書いてしまえば、main.cppのコンパイルでインスタンス化が行われ正常に動作する。  

# 明示的インスタンス化

インスタンス化はその関数テンプレートが使われたタイミングで行われてきたが、それ以外のタイミングでも明示的にインスタンス化を行うことができる。  

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

リンカエラーになった最初のソースコードのf.cppの最後の行に
```cpp
template void f<int>(const int &);
```
が追加されただけである。  
この行により、関数テンプレートfの使用の有無にかかわらずコンパイラは```f<int>```をインスタンス化する。  

こうするメリットとして実行ファイルのサイズを小さくできる点が挙げられる。  
ヘッダーファイル直書きの方法だと、多くの翻訳単位で```f<int>```を利用したときそれぞれの翻訳単位でインスタンス化されてしまい実行ファイルのサイズが膨大になってしまう。  
しかし一か所のみでインスタンス化することによってファイルサイズを小さくできる。  

デメリットとしては、使う度にf.cppに戻って使いたい型についてインスタンス化の処理が書かれているかを調べて無ければ追加する必要があり、煩雑である点だ。  
ホワイトリスト方式で使用する型をインスタンス化するため、想定される型が少なければ現実的だが柔軟性のないコードになってしまう。  
この点は次のextern templateを用いることで解決できる。  

# extern template

明示的インスタンス化が存在するので、明示的非インスタンス化も可能である。  
すなわち、関数テンプレートやクラステンプレートを使用し定義が見えているにも関わらず、その翻訳単位内ではインスタンス化しないことができる。  

```cpp
/* f.hpp */
#include<iostream>
template<typename T>
void f(const T &t) {
    std::cout << t << std::endl;
}

//int型でインスタンス化しない
extern template void f<int>(const int &);
```
```cpp
/* f.cpp */
#include"f.hpp"
//明示的インスタンス化
template void f<int>(const int &);
```
```cpp
/* main.cpp */
#include"f.hpp"

int main() {
    int i = 10;
    double d = 20.5;
    f(i); //インスタンス化されない
    f(d); //インスタンス化される
}
```

extern templateが翻訳単位中に現れたとき、その翻訳単位中では指定された型において明示的インスタンス化されない限りインスタンス化されない。  
上のソースコードでは、main.cppのコンパイル時には```f<int>```はインスタンス化されず、f.cppのコンパイル時にインスタンス化される。  

柔軟性を確保した上でファイルサイズを削減することができた。  
しかし結局定義がヘッダーファイルに書かれてしまっているためインクルードコストの増大に対応できない。  
ヘッダーファイル直書きの方法の改良版といった形になるだろう。  

## Tips : inline展開と非インスタンス化
非インスタンス化した場合、翻訳単位の外に定義を求めるため関数のインライン展開を阻害してしまう。  
今回のように簡単な関数についてはinline宣言をしてヘッダーファイルに書いてしまうのがベストだろう。  

# 演習問題
part1で作ったqueue的配列クラスについて以下の要件で拡張せよ
1. すべてヘッダーファイルに書く形でファイル分けする。  
2. int, std::arrayの場合についてextern templateを用いて非インスタンス化し、cppファイルを作成して明示的インスタンス化する。  