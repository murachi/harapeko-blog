[author: murachi]
# C++0x の <unordered_map> を使用する

C++0x 規格の標準ライブラリとして正式採用された <unordered_map> を GCC 4.4.0 から利用するには、 g++ コマンドのオプションに **-std=gnu++0x** を加える必要がある。

```
$ g++ -o test -std=gnu++0x test.cpp
```

-std オプションには値 "c++0x" を指定することもできるが、 4.4.0 では <cwchar> 内で参照されている ::swprintf および ::vswprintf が未定義であるとするコンパイルエラーが発生する模様。
