# テンプレート講座

ここでは基本的なテンプレートの文法を紹介する。  

# テンプレート関数

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
void pringArray(T t, size_t size) {
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

    printArray<int>(arr);
    std::cout << "\n";
    printArray<float>(arr2);
}
```

printArray関数がテンプレート関数と呼ばれるものである。  
関数の前に
```cpp
template<typename T>
```
と書くことでTという識別子をあいまいな型として扱うことができる。  
