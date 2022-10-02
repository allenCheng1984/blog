---
layout: post
title:  The Thirteen Commandments for Newbie C/C++ Programmers
date:   2022-10-1 06:00:50 +0800
categories: C
---

[from PTT C_and_CPP 版](https://www.ptt.cc/bbs/C_and_CPP/M.1465304337.A.9F2.html)

[友善閱讀(HackMD)](https://hackmd.io/@allencheng/rkfm2IHGo)


這裡做排版與筆記, 以 C 語言為主

---

## 01. 不可以使用尚未給予適當初值的變數

錯誤例子：
```c
int accumulate(int max)
{
    int sum;    /* 未給予初值的區域變數，其內容值是垃圾 */
    for (int num = 1; num <= max; num++) {  sum += num;  }
    return sum;
}
```

正確例子：
```c
int accumulate(int max)
{
    int sum = 0;    /* 正確的賦予適當的初值 */
    for (int num = 1; num <= max; num++) {  sum += num;  }
    return sum;
}
```

備註：

根據 C Standard，具有靜態儲存期(static storage duration)的變數，\
例如 全域變數(global variable)或帶有 static 修飾符者等，\
如果沒有顯式初始化的話，根據不同的資料型態予以進行以下初始化：

若變數為算術型別 (int , double , ...) 時，初始化為零或正零。\
若變數為指標型別 (int*, double*, ...) 時，初始化為 null 指標。\
若變數為複合型別 (struct, double _Complex, ...) 時，遞迴初始化所有成員。\
若變數為聯合型別 (union) 時，只有其中的第一個成員會被遞迴初始化。



---

## 02. 不能存取超過陣列既定範圍的空間

錯誤例子：
```c
int str[5];
for (int i = 0 ; i <= 5 ; i++) str[i] = i;
```

正確例子：
```c
int str[5];
for (int i = 0; i < 5; i++) str[i] = i;
```

說明：

宣告陣列時，所給的陣列元素個數值如果是 N, 那麼我們在後面\
透過 [索引值] 存取其元素時，所能使用的索引值範圍是從 0 到 N-1

C/C++ 為了執行效率，並不會自動檢查陣列索引值是否超過陣列邊界，\
我們要自己來確保不會越界。一旦越界，操作的不再是合法的空間，\
將導致無法預期的後果。

---

## 03. 不可以提取不知指向何方的指標

錯誤例子：
```c
char *pc1;         /* 未給予初值，不知指向何方 */
char *pc2 = NULL;  /* pc2 起始化為 null pointer */
*pc1 = 'a';        /* 將 'a' 寫到不知何方，錯誤 */
*pc2 = 'b';        /* 將 'b' 寫到「位址0」，錯誤 */
```

正確例子：
```c
char c;          /* c 的內容尚未起始化 */
char *pc1 = &c;  /* pc1 指向字元變數 c */
*pc1 = 'a';      /* c 的內容變為 'a' */

/* 動態分配 10 個 char(其值未定),並將第一個char的位址賦值給 pc2 */
char *pc2 = (char *) malloc(10);
pc2[0] = 'b';    /* 動態配置來的第 0 個字元，內容變為 'b' */
free(pc2);
```

說明：

指標變數必需先指向某個可以合法操作的空間，才能進行操作。\
( 使用者記得要檢查 malloc 回傳是否為 NULL，礙於篇幅本文假定使用上皆合法，也有正確歸還記憶體 )

錯誤例子：
```c
char *name;             /* name 尚未指向有效的空間 */
printf("Your name, please: ");
fgets(name,20,stdin);   /* 您確定要寫入的那塊空間合法嗎??? */
printf("Hello, %s\n", name);
```

正確例子：
```c
/* 如果編譯期就能決定字串的最大空間，那就不要宣告成 char* 改用 char[] */

char name[21];         /* 可讀入字串最長 20 個字元，保留一格空間放 '\0' */
printf("Your name, please: ");
fgets(name,20,stdin);
printf("Hello, %s\n", name);
```

正確例子(2)：

若是在執行時期才能決定字串的最大空間，C提供兩種作法：

**利用 malloc() 函式來動態分配空間，用malloc宣告的陣列會被存在heap**\
須注意：若是宣告較大陣列，要確認malloc的回傳值是否為NULL

```c
size_t length;
printf("請輸入字串的最大長度(含null字元): ");
scanf("%u", &length);

name = (char *)malloc(length);
if (name) {         // name != NULL
    printf("您輸入的是 %u\n", length);
} else {            // name == NULL
    puts("輸入值太大或系統已無足夠空間");
}
/* 最後記得 free() 掉 malloc() 所分配的空間 */
free(name);
name = NULL;
```

**C99開始可使用variable-length array (VLA)**
須注意：
- 因為VLA是被存放在stack裡，使用前要確認array size不能太大
- 不是每個compiler都支援VLA `(gcc和clang支援VLA，Visual C++不支援)`
- C++ Standard不支援 `(雖然有些compiler支援)`

```c
float read_and_process(int n)
{
    float vals[n];
    for (int i = 0; i < n; i++)
        vals[i] = read_val();
    return process(vals, n);
}
```



---

## 04. 不要試圖用 char* 去更改一個"字串常數

**試圖去更改字串常數(string literal)的結果會是undefined behavior。**

錯誤例子：
```c
char* pc = "john";  /* pc 現在指著一個字串常數 */
*pc = 'J';  /* undefined behaviour，結果無法預測 */
pc = "jane";        /* 合法，pc指到在別的位址的另一個字串常數 */
                    /* 但是"john"這個字串還是存在原來的地方不會消失 */
```

因為 `char* pc = "john"` 這個動作會新增一個內含元素為 `"john\0"` 的 `static char[5]`，然後pc會指向這個static char的位址(通常是唯讀)。

若是試圖存取這個 `static char[]` ，Standard並沒有定義結果為何。

`pc = "jane"` 這個動作會把 pc 指到另一個沒在用的位址然後新增一個內含元素為 `"jane\0" `的 `static char[5]`。\
可是之前那個字串 `"john\n"` 還是留在原地沒有消失。

通常編譯器的作法是把字串常數放在一塊 read only(.rdata) 的區域內，此區域大小是有限的，所以如果你重複把 pc 指給不同的字串常數，是有可能會出問題的。

正確例子：
```c
char pc[] = "john";  /* pc 現在是個合法的陣列，裡面住著字串 john */
                     /* 也就是 pc[0]='j', pc[1]='o', pc[2]='h',
                                         pc[3]='n', pc[4]='\0'  */
*pc = 'J';
pc[2] = 'H';
```

說明：

字串常數的內容應該要是"唯讀"的。您有使用權，但是沒有更改的權利。\
若您希望使用可以更改的字串，那您應該將其放在合法空間

錯誤例子：
```c
char *s1 = "Hello, ";
char *s2 = "world!";
/* strcat() 不會另行配置空間，只會將資料附加到 s1 所指唯讀字串的後面，
   造成寫入到程式無權碰觸的記憶體空間 */
strcat(s1, s2);
```

正確例子：
```c
/* s1 宣告成陣列，並保留足夠空間存放後續要附加的內容 */
char s1[20] = "Hello, ";
char *s2 = "world!";
/* 因為 strcat() 的返回值等於第一個參數值，所以 s3 就不需要了 */
strcat(s1, s2);
```

備註：

由於不加const容易造成混淆，\
建議不管是C還是C++一律用 const char* 定義字串常數。


---
## 05. 不能在函式中回傳一個指向區域性自動變數的指標

錯誤例子：
```c
char *getstr(char *name)
{
    char buf[30] = "hello, "; /* 將字串常數"hello, "的內容複製到buf陣列 */
    strcat(buf, name);
    return buf; // <== 有問題
}
```

說明：

區域性自動變數，將會在離開該區域時(本例中就是從getstr函式返回時)\
被消滅，因此呼叫端得到的指標所指的字串內容就失效了。

正確例子：
```c
void getstr(char buf[], int buflen, char const *name)
{
    char const s[] = "hello, ";
    strcpy(buf, s);
    strcat(buf, name);
}
```

正確例子：
```c
int* foo()
{
    int* pInteger = (int*) malloc( 10*sizeof(int) );
    return pInteger;
}

int main()
{
    int* pFromfoo = foo();
}
```

說明：

上例雖然回傳了函式中的指標，但由於指標內容所指的位址並非區域變數，而是用動態的方式抓取而得，換句話說這塊空間是長在 heap 而非 stack，又因 heap 空間並不會自動回收，因此這塊空間在離開函式後，依然有效。\
(但是這個例子可能會因為 programmer 的疏忽，忘記 free 而造成memory leak)



---
## 06. 不可以只做 malloc(), 而不做相應的 free()

但若不是用 `malloc()` 所得到的記憶體，則不可以 `free()`。已經 `free()`了所指記憶體的指標，在它指向另一塊有效的動態分配得來的空間之前，不可以再被 `free()`，也不可以提取(dereference)這個指標。

小技巧: 可在 free 之後將指標指到 `NULL`， free 不會對空指標作用。

例：
```c
int *p = malloc(sizeof(int));
free(p);
p = NULL;
free(p);  // free不會對空指標有作用
```


---
## 07. 在數值運算、賦值或比較中不可以隨意混用不同型別的數值

錯誤例子：
```c
unsigned int sum = 2000000000 + 2000000000;  /* 超出 int 存放範圍 */
unsigned int sum = (unsigned int) (2000000000 + 2000000000);
double f = 10 / 3;
```

正確例子：
```c
/* 全部都用 unsigned int, 注意數字後面的 u, 大寫 U 也成 */
unsigned int sum = 2000000000u + 2000000000u;

/* 或是用顯式的轉型 */
unsigned int sum = (unsigned int) 2000000000 + 2000000000;
double f = 10.0 / 3.0;
```

錯誤例子:
```c
unsigned int a = 0;
int b[10];
for(int i = 9; i >= a; i--) {  b[i] = 0;  }
```

說明：\
由於 int 與 unsigned 共同運算的時候，會轉換 int 為 unsigned，因此迴圈條件永遠滿足，與預期行為不符

--

錯誤例子：
```c
unsigned char a = 0x80;   /* no problem */
char b = 0x80;    /* implementation-defined result */
if( b == 0x80 ) {        /* 不一定恒真 */
    printf( "b ok\n" );
}
```

說明：\
語言並未規定 char 天生為 unsigned 或 signed，因此將 0x80 放入 char 型態的變數，將會視各家編譯器不同作法而有不同結果

--

錯誤例子(以下假設為在32bit機器上執行)：
```c
#include <math.h>
long a = -2147483648 ;  // 2147483648 = 2 的 31 次方
while ( labs(a)>0 ) {   // labs(-2147483648) <0 有可能發生
    ++a;
}
```

說明：\
如果你去看 C99/C11 Standard，你會發現 long 變數的最大/最小值為

```c
/* (被 define 在 limits.h) */
LONG_MIN      -2147483647  // compiler實作時最小值不可大於 -(2147483648-1)
LONG_MAX       2147483647  // compiler實作時最小值不可小於  (2147483648-1)
```

不過由於 32bit 能顯示的範圍就是 2**32 種，所以一般 16/32bit 作業系統會把 LONG_MIN 多減去1，也就是 int 的顯示範圍為 (-LONG_MAX - 1) ~ LONG_MAX。\
(64bit 的作業系統 long 多為 8 bytes，但是依舊符合 Standard 要求的最小範圍)

當程式跑到 `labs(-2147483648)>0` 時，由於 2147483648 大於 LONG_MAX，\
Standard 告訴我們，當 labs 的結果無法被 long 有限的範圍表示，編譯器會怎麼幹就看他高興 (undefined behavior)。\
(不只 long ，其他如 int 、 long long等以此類推)

補充資料：

- C11 Standard 5.2.4.2.1, 7.22.6.1
- https://www.fefe.de/intof.html


---
## 08. ++i/i++/--i/i--/f(&i)哪個先執行跟順序有關

`++i/i++` 和 `--i/i--` 的問題幾乎每個月都會出現，所以特別強調。

當一段程式碼中，某個變數的值用某種方式被改變一次以上，例如 `++x`/ `--x` / `x++` / `x--` / `function(&x)(能改變x的函式)`
- 如果 Standard 沒有特別去定義某段敘述中哪個部份必須被先執行，那結果會是 undefined behavior(結果未知)。
- 如果 Standard 有特別去定義執行順序，那結果就根據執行順序決定。


C/C++均正確的例子：
```c
if (--a || ++a) {}        // || 左邊先計算，如果左邊為 1 右邊就不會算
if (++i && f(&i)) {}      // && 左邊先計算，如果左邊為 0 右邊就不會算
a = (*p++) ? (*p++) : 0;  // 問號左邊先計算
int j = (++i, i++);       // 這裡的逗號為運算子，表示依序計算
```

C/C++均錯誤的例子：
```c
int j = ++i + i++; // undefined behavior，Standard 沒定義 + 號哪邊先執行
x = x++;           // undefined behavior, Standard 沒定義 = 號哪邊先執行
printf( "%d %d %d", I++, f(&I), I++ ); // undefined behavior, 原因同上
foo(i++, i++);     // undefined behavior，這裡的逗號是用來分隔引入參數的
                   // 分隔符(separator) 而非運算子， Standard 沒定義哪邊先執行
```

補充資料

- Undefined behavior and sequence points
  http://stackoverflow.com/questions/4176328/undefined-behavior-and-
  sequence-points)
- C11 Standard 6.5.13-17，Annex C
- Sequence poit
  https://en.wikipedia.org/wiki/Sequence_point


---
## 09. 慎用Macro

Macro 是個像鐵鎚一樣好用又危險的工具：\
用得好可以釘釘子，用不好可以把釘子打彎、敲到你手指或被抓去吃子彈。\
因為 macro 定義出的「偽函式」有以下缺點：

1. debug 會變得複雜。
2. 無法遞迴呼叫。
3. 無法用 `&` 加在 macro name 之前，取得函式位址。
4. 可能會導致奇怪的side effect或其他無法預測的問題。

所以，使用macro前，請先確認以上的缺點是否會影響你的程式運行。\
替代方案：`enum`(定義整數)，`const T`(定義常數)，`inline function`(定義函式)


以下就針對macro的缺點做說明：

- debug會變得複雜。
  - 編譯器不能對macro本身做語法檢查，只能檢查預處理(preprocess)後的結果。
- 無法遞迴呼叫。
  - 根據C standard 6.10.3.4，如果某macro的定義裡裏面含有跟此macro名稱同樣的的字串，該字串將不會被預處理。
    ```c
      所以：

      #define pr(n) ((n==1)? 1 : pr(n-1))
      cout<< pr(5) <<endl;

      預處理過後會變成：

      cout<< ((5==1)? 1 : pr(5 -1)) <<endl;  // pr沒有定義，編譯會出錯
    ```
- 無法用 & 加在 macro name 之前，取得函式位址。
  - 因為他不是函式，所以你也不可以把函式指標套用在macro上。
- 可能會導致奇怪的side effect或其他無法預測的問題。
  - 錯誤例子：
    ```c
      #include <stdio.h>
      #define SQUARE(x)    (x * x)
      int main()
      {
          printf("%d\n", SQUARE(10-5)); // 預處理後變成SQUARE(10-5*10-5)
          return 0;
      }
    ```

  - 正確例子：在 Macro 定義中, 務必為它的參數個別加上括號
    ```c
      #include <stdio.h>
      #define SQUARE(x)    ((x) * (x))
      int main()
      {
          printf("%d\n", SQUARE(10-5));
          return 0;
      }
    ```

    不過遇到以下有side effect的例子就算加了括號也沒用。

  - 錯誤例子：
    ```c
    #define MACRO(x)     (((x) * (x)) - ((x) * (x)))
    int main()
    {
        int x = 3;
        printf("%d\n", MACRO(++x)); // 有side effect
        return 0;
    }
    ```

補充資料：

- http://stackoverflow.com/questions/14041453/why-are-preprocessor-macros-evil-and-what-are-the-alternatives
- http://stackoverflow.com/questions/12447557/can-we-have-recursive-macros
- C11 Standard 6.10.3.4


---
## 10. 不要在 stack 設置過大的變數以避免堆疊溢位(stack overflow

由於編譯器會自行決定 stack 的上限，某些預設是數 KB 或數十KB，當變數所需的空間過大時，很容易造成 stack overflow，程式亦隨之當掉 (segmentation fault)。

可能造成堆疊溢位的原因包括遞迴太多次 (多為程式設計缺陷)，或是在 stack 設置過大的變數。

錯誤例子：
```c
int array[10000000];       // 在stack宣告過大陣列
```

正確例子：
```c
int *array = (int*) malloc( 10000000*sizeof(int) );
```
說明：

建議將使用空間較大的變數用malloc/new配置在 heap 上，由於此時 stack 上只需配置一個 int* 的空間指到在heap的該變數，可避免 stack overflow。

使用 heap 時，雖然整個 process 可用的空間是有限的，但採用動態抓取的方式，new 無法配置時會丟出 std::bad_alloc 例外，malloc 無法配置時會回傳 null(註2)，不會影響到正常使用下的程式功能

備註：

註1.\
使用 heap 時，整個 process 可用的空間一樣是有限的，若是需要頻繁地 malloc / free 或 new / delete 較大的空間，需注意避免造成記憶體破碎 (memory fragmentation)。

註2.\
由於 Linux 使用 overcommit 機制管理記憶體， malloc 即使在記憶體不足時仍然會回傳非 NULL 的 address ，同樣情形在 Windows / Mac OS 則會回傳 NULL

補充資料：
- https://zh.wikipedia.org/wiki/%E5%A0%86%E7%96%8A%E6%BA%A2%E4%BD%8D
- http://stackoverflow.com/questions/3770457/what-is-memory-fragmentation
- http://library.softwareverify.com/memory-fragmentation-your-worst-nightmare/

overcommit 跟 malloc:
- http://goo.gl/V9krbB
- http://goo.gl/5tCLQc


---
## 11. 使用浮點數精確度造成的誤差問題

根據 IEEE 754 的規範，又電腦中是用有限的二進位儲存數字，因此常有可能因為精確度而造成誤差，例如加減乘除，等號大小判斷，分配律等數學上常用到的操作，很有可能因此而出錯(不成立)

更詳細的說明可以參考冼鏡光老師所發表的一文 "使用浮點數最最基本的觀念"
http://blog.dcview.com/article.php?a=VmhQNVY%2BCzo%3D


---
## 12. 不要猜想二維陣列可以用 pointer to pointer 來傳遞

首先必須有個觀念，C 語言中陣列是無法直接拿來傳遞的!\
不過這時候會有人跳出來反駁:

```c
void pass1DArray( int array[] );

int a[10];
pass1DArray( a );   /* 可以合法編譯，而且執行結果正確!! */
```

事實上，編譯器會這麼看待

```c
void pass1DArray( int *array );

int a[10];
pass1DArray( &a[0] );
```

我們可以順便看出來，array 變數本身可以 decay 成記憶體起頭的位置，因此我們可以 `int *p = a;` 這種方式，拿指標去接陣列。\
也因為上述的例子，許多人以為那二維陣列是不是也可以改成 `int **`

錯誤例子:
```c
void pass2DArray( int **array );

int a[5][10];
pass2DArray( a );
/* 這時候編譯器就會報錯啦 */
/* expected ‘int **’ but argument is of type ‘int (*)[10]’*/
```

在一維陣列中，指標的移動操作，會剛好覆蓋到陣列的範圍。例如，宣告了一個 `a[10]`，那我可以把 a 當成指標來操作 `*a` 至 `*(a+9)`\
因此我們可以得到一個概念，在操作的時候，可以 decay 成指標來使用，也就是我可以把一個陣列當成一個指標來使用 (again, **陣列!=指標**)

但是多維陣列中，無法如此使用，事實上這也很直觀，試圖拿一個 `pointer to pointer to int` 來操作一個 int 二維陣列，這是不合理的！

儘管我們無法將二維陣列直接 decay 成兩個指標，但是我們可以換個角度想，二維陣列可以看成 **"外層大的一維陣列，每一維內層各又包含著一維陣列"**

如果想通了這一點，我們可以仿造之前的規則，把外層大的一維陣列 decay 成指標，該指標指向內層的一維陣列
```c
void pass2DArray( int (*array) [10] );  // array 是個指標，指向 int [10]

int a[5][10];
pass2DArray( a );
```

這時候就很好理解了，函數 `pass2DArray` 內的 `array[0]` 會代表什麼呢？\
答案是它代表著 `a[0]` 外層的那一維陣列，裡面包含著內層 `[0]~[9]`\
也因此 `array[0][2]` 就會對應到 `a[0][2]` ， `array[4][9]` 對應到 `a[4][9]`

結論就是，只有最外層的那一維陣列可以 decay 成指標，其他維陣列都要明確的指出陣列大小，這樣多維陣列的傳遞就不會有問題了

也因為剛剛的例子，我們可以清楚的知道在傳遞陣列時，實際行為是在傳遞指標，也因此如果我們想用 sizeof 來求得陣列元素個數，那是不可行的

錯誤例子:
```c
void print1DArraySize( int* arr ) {
    printf("%u", sizeof(arr)/sizeof(arr[0])); /* sizeof(arr) 只是 */
}                                             /* 一個指標的大小   */

受此限制，我們必須手動傳入大小
void print1DArraySize( int* arr, size_t arrSize );
```

---
## 13. 函式內 new 出來的空間記得要讓主程式的指標接住

對指標不熟悉的使用者會以為以下的程式碼是符合預期的
```c
void newArray(int* local, int size) {
    local = (int*) malloc( size * sizeof(int) );
}

int main() {
    int* ptr;
    newArray(ptr, 10);
}
```

接著就會找了很久的 bug，最後仍然搞不懂為什麼 ptr 沒有指向剛剛拿到的合法空間

![](https://hackmd.io/_uploads/HkPLy_Bfi.png)

也許有人會想問，指標不是傳址嗎？

精確來講，指標也是傳值，只不過該值是一個位址 (ex: `0xfefefefe`)

local 接到了 ptr 指向的那個位置，接著函式內 local 要到了新的位置\
但是 ptr 指向的位置還是沒變的，因此離開函式後就好像事什麼都沒發生\
( 嚴格說起來還發生了 memory leak )

以下是一種解決辦法
```c
int* createNewArray(int size) {
    return (int*) malloc( size * sizeof(int) );
}

int main() {
    int* ptr;
    ptr = createNewArray(10);
}
```

改成這樣亦可   ( 為何用 int** 就可以？想想他會傳什麼過去給 local )
```c
void createNewArray(int** local, int size) {
    *local = (int*) malloc( size * sizeof(int) );
}

int main() {
    int *ptr;
    createNewArray(&ptr, 10);
}
```


---

## 後記

從「古時候」流傳下來一篇文章

  "The Ten Commandments for C Programmers"(Annotated Edition)
  by Henry Spencer
  http://www.lysator.liu.se/c/ten-commandments.html

  一方面它不是針對 C 的初學者，一方面它特意模仿中古英文聖經的用語，寫得文謅謅。所以我現在另外寫了這篇，希望能涵蓋最重要的觀念以及初學甚至老手最易犯的錯誤。

作者：潘科元(Khoguan Phuann) (c)2005. 感謝 ptt.cc BBS 的 C_and_CPP 看板眾多網友提供寶貴意見及程式實例。

  nowar100 多次加以修改整理，擴充至 13 項，並且製作成動畫版。

  wtchen 應板友要求移除動畫並根據C/C++標準修改內容(Ver.2016)

  如發現 Bug 請推文回報，謝謝您
