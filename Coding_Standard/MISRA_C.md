
# MISRA C

## Introduction

* MISRA : Motor Industry Software Reliability Association (汽車產業軟體可靠性協會)
* MISRA C : 指導你怎樣避免寫出有問題的 C 程式
* MISRA C:1998 / MISRA C:2004 / MISRA C:2012
* [Green Hills support](https://www.ghs.com/products/misrac.html)

## Environment

### Rule 1.1 (req) (by ijliao)

* All code shall conform to ISO/IEC 9899:1990 “Programming languages — C”, amended and corrected by ISO/IEC 9899/COR1:1995, ISO/IEC 9899/AMD1:1995, and ISO/IEC 9899/COR2:1996.
* 中文說明 : 程式碼需符合 C 語言標準
  * [ISO/IEC 9899](https://www.iso.org/standard/17782.html) : 標準本文
  * [ISO/IEC 9899/COR1](https://www.iso.org/standard/24271.html) : 勘誤 1
  * [ISO/IEC 9899/AMD1](https://www.iso.org/standard/23909.html) : 修訂 1
  * [ISO/IEC 9899/COR2](https://www.iso.org/standard/27110.html) : 勘誤 2

### Rule 1.2 (req) (by U.Chen)

* No reliance shall be placed on undefined or unspecified behaviour.
* 中文說明 : 不得依賴[未定義或未指定的行為](https://en.cppreference.com/w/c/language/behavior)
* 避免依賴於那些未被規則所規範的未定義與未指定行為 如果另一條規則明確涵蓋了特定行為，則僅在需要時才需要偏離該特定規則。

### Rule 1.3 (req) (by Jackal)

* Multiple compilers and/or languages shall only be used if there is a common defined interface standard for object code to which the languages/compilers/assemblers conform.
* 中文說明 : 僅當存在針對語言/編譯器/彙編器所遵循的目標代碼的通用定義接口標準時，才可使用多種編譯器和/或語言。
* 如果要使用C以外的語言來實現模塊或使用其他C編譯器進行編譯，則必須確保該模塊將與其他模塊正確集成。C語言行為的某些方面取決於編譯器，因此，對於所使用的編譯器，必須理解這些方面。以下是需要了解到的問題:stack使用情況、參數傳遞以及數據值的存儲方式（長度，對齊方式，鋸齒，重疊等）

### Rule 1.4 (req) (by Weiren)

* The compiler/linker shall be checked to ensure that 31 character significance and case sensitivity are supported for external identifiers.
* 中文說明 : 應當檢查編譯器/鏈接器，以確保外部標識符支持31個字符有效和區分大小寫。
* [未定義7; 實施5、6]
* ISO標準要求外部標識符的前6個字符必須不同。但是，由於大多數編譯器/鏈接器至少允許31個字符有效（對於內部標識符而言），因此遵守此平凡且無用的限制被認為是不必要的限制。
* 必須檢查編譯器/鏈接器是否符合上述標準。如果編譯器/鏈接器不能夠滿足此限制，則使用編譯器的限制。

### Rule 1.5 (adv) (by Mars)

* Floating-point implementations should comply with a defined floating-point standard.
* 中文說明 : 浮點數的應用需遵守浮點數定義標準
* 浮點運算會帶來許多問題，其中有一些問題可以透過公認的標準來克服，ANSI/IEEE Std 754 [21] 就是一個合適的標準，規則 6.3 關於浮點數類型的定義提供了一個正在用的例子，如：

    ```c
    /* IEEE 754 single precision floating point */
    typedef float float32_t;
    ```

## Language extensions

### Rule 2.1 (req) (by Noah)

* Assembly language shall be encapsulated and isolated in either (a) assembler functions, (b) C functions or (c) macros.
* 中文說明：組語必需是以(a) assembler functions、(b)C functions或是(c) macros的形式被獨立包起來使用
* 範例：
  * 不合規定的寫法：

    ```c
    void fn ( void )
    {
        DoSomething ( );
        asm ( "NOP" ); // Noncompliant, asm mixed with C/C++ statements
        DoSomething ( );
    }
    ```

  * 合規定的寫法：

      (a) assembler function:

      ```asm
          .global
          _Delay:
              nop
          jmp
      ```

      ```c
          void fn ( void )
              {
                  DoSomething ( );
                  Delay ( ); // Compliant, Assembler is encapsulated
                  DoSomething ( );
              }

      ```

      (b) C function:

      ```c
          void Delay ( void )
          {
              asm ( "NOP" ); // Compliant, asm not mixed with C/C++ statements
          }

          void fn ( void )
          {
              DoSomething ( );
              Delay ( ); // Compliant, Assembler is encapsulated
              DoSomething ( );
          }
      ```

      (c) Macro:

      ```c
          #define Delay asm ( "NOP" );

          void fn ( void )
          {
              DoSomething ( );
              Delay;
              DoSomething ( );
          }
      ```

### Rule 2.2 (req) (by Ray)

* Source code shall only use `/*... */` style comments.
* 中文說明：源代碼只能使用 `/* … */` 樣式註釋。
* 在C90中不允許"//"的註釋樣式 (雖然有許多編譯器以插件的方式支援這樣的寫法)。在編譯程式定向 (preprocessor directives ex: #define) 的觀點看來， `//` 在不同情況下也會有不同效果。此外，混用 `/* ....*/` 和 `//` 並不統一。
* 最重要的是，這已經不僅僅是風格問題了，因為在c99前的編譯器對於 `//` 的定義與處理可能都不太相同。
* 範例：
  * 不合規定的寫法：

    ```c
    // C99 style comments and C++ style comments
    ```

  * 合規定的寫法：

    ```c
    /* Original C style comment
    Can extend across multiple lines
    */
    ```

### Rule 2.3 (req) (by Liou)

* The character sequence `/*` shall not be used within a comment.
* 中文說明 :字符序列 `/*` 不得在註釋中使用。
* 範例:

    ```c
    /* some comment, end comment marker accidentally omitted
    <<New Page>>
    Perform_Critical_Safety_Function (X);
    /* this comment is not compliant */
    ```

以上述例子為例，由於不小心省略了末端的注釋符號，`Perform_Critical_Safety_Function`將不會被執行

### Rule 2.4 (adv) (by U.Chen)

* Sections of code should not be “commented out”.
* 中文說明 : 程式碼區段不應該被"註釋掉"
* 當程式碼不需要被編譯執行時，應該用條件編譯來完成"注釋"的目的 ( 如：#if 或 #ifdef 加上註釋 )，用 `/*` 跟 `*/` 使程式碼不執行是危險的，因為C語言的編譯器不支援這樣的方式進行巢狀註釋(nested comments)，任何程式碼中的原註釋內容可能會影響執行結果。

## Documentation

### Rule 3.1 (req) (by Jackal)

* All usage of implementation-defined behaviour shall be documented.
* 中文說明 : 實作定義行為的所有用法都應記錄下來。
* 任何依賴於未被其他規則明確處理的實作定義行為都應記錄在案，例如，記錄在編譯器的文檔中。
* 如果另一條規則明確涵蓋了特定行為，則僅在需要時才需要偏離該特定規則。 有關這些問題的完整列表，請參見ISO / IEC 9899：1990附錄G [2]。

### Rule 3.2 (req) (by Weiren)

* The character set and the corresponding encoding shall be documented.
* 中文說明 : 字符集和相應的編碼應該文檔化。
* 例如，ISO 10646 [22]定義了字符集映射到數字值的國際標準。出於可移植性的考慮，字符常數和字串只能包含映射到已經文檔化的子集中的字符。源代碼以一個或多個字符集編寫。程序可彈性在第二個或多個字符集中執行。所有的源代碼和執行字符集都應當被記錄下來。

### Rule 3.3 (adv) (by Mars)

* The implementation of integer division in the chosen compiler should be determined, documented and taken into account.
* 中文說明：在選擇的編譯器時，整數除法的實現應該要被確認、記錄以及考慮到。
* 當兩個帶符號的整數做除法時，ISO相容的編譯器有潛在的可能產出兩個不同結果，
    1. 編譯器可能會四捨五入(進位)並帶來負餘數或 (e.g. -5/3 = -1 remainder -2)
    2. 也可能會四捨五入(捨去)並帶來正餘數 (e.g. -5/3 = -2 remainder +1)
* 重要的是，這些編譯的結果須確認後並記錄下來轉告程序員，尤其是第二種情況(比較少見)。

### Rule 3.4 (req) (by Noah)

* All uses of the #pragma directive shall be documented and explained.
* 中文說明：所有 pragma 的使用都需有文件來說明他的含意

### Rule 3.5 (req) (by Ray)

* If it is being relied upon, the implementation defined behaviour and packing of bitfields shall be documented.
* 中文說明：當使用實作定義的行為(implementation defined behavior)和位域打包(packing of bitfields)時，請記得都要記錄下來。
* 位域用於兩個主要用途：
    1. 以較大的數據類型（`union` 結合併集）訪問單個位或一組位。 不允許這種使用（請參見規則18.4）。
    2. 允許打包旗標或其他短長度數據以節省存儲空間。
* 在這份文件中，位域的使用僅在打包短長度數據時被接受。由於 `structure` 中的元素僅可透過其名字來溝通，開發者不應該去考慮位域在 `sturcture` 中的儲存方式。
* 建議特別宣告 `structure` 來保留位域段集，並且不要在同一 `structure` 內包含任何其他數據。
* 如果編譯器有一個開關來強制位字段遵循特定的佈局，那麼這可能會有助於這種證明。
* 範例：
  * 合規定的寫法：

    ```c
    struct message /* Struct is for bit-fields only */
    {
        signed int little: 4; /* Note: use of basic types is required */
        unsigned int x_set: 1;
        unsigned int y_set: 1;
    } message_chunk;
    ```

* 使用位域時，請注意潛在的陷阱和實作定義行為（即非便攜式）。 尤其應該注意以下幾點：
  * 存儲單元中位域的對齊方式是實作定義的，即它們是從單元的高端還是低端開始儲存（通常是一個字節）。
  * 位域是否可以與存儲單元邊界重疊也是實作定義的範疇(例如：如果順序儲存一個6位字段和一個4位字段，那麼4位字段是全部從新的字節開始，還是其中2位佔據一個字節的剩餘2位，而其他2位開始於下個字節)。

### Rule 3.6 (req) (by Liou)

* All libraries used in production code shall be written to comply with the provisions of this document, and shall have been subject to appropriate validation.
* 中文說明：生產代碼中使用的所有函數庫均應符合本文件的規定，並且經過適當的驗證。

## Character sets

### Rule 4.1 (req) (by U.Chen)

* Only those escape sequences that are defined in the ISO C standard shall be used.
* 中文說明 : 只能使用 ISO C標準中定義的跳脫序列
* 僅允許使用ISO / IEC 9899：1990 [3-6]第6.1.3.4節中的“簡單跳脫序列”和 `\0`。
禁止所有“十六進制跳脫序列”。
規則7.1也禁止使用 `\0` 以外的“八進制跳脫序列”。
* 簡單跳脫序列
  * `\a` 警報聲
  * `\b` Backspace
  * `\f` 換頁
  * `\n` 換行
  * `\r` 回車
  * `\t` 水平Tab
  * `\v` 垂直Tab
  * `\\` 反斜槓
  * `\'` 單引號
  * `\"` 雙引號
  * `\?` 問號
  * `\0` Null

### Rule 4.2 (req) (by Jackal)

* Trigraphs shall not be used.
* 中文說明：不應使用三字符組。
* 三字符組由兩個問號加上一個特殊的第三字元表示而成 ( `??-` 表示 `~`   `??)` 表示 `]` )，在某些情況，這可能會意外的造成混淆 (見範例)
* 範例:

    ```c
    /* For example the string */
    "(Date should be in the form ??-??-??)"
    /* would not behave as expected, actually being interpreted by the compiler as */
    "(Date should be in the form ~~]"
    ```

## Identifiers

### Rule 5.1 (req) (by Weiren)

* Identifiers (internal and external) shall not rely on the significance of more than 31 characters.
* 中文說明：標識符（內部和外部）不得依賴於超過31個有效字符。
* ISO標準要求內部標識符的前31個字符必須不同，以確保代碼的可移植性。即使編譯器支持，也不應超過此限制。
* 這條規則應適用於所有名稱空間。
* 宏名稱 (macro names) 也包括在內，且限制為31個字符適用於替換前後。
* ISO標準要求外部標識符的前6個字符必須不同，以確保最佳的可移植性。但是，此限制相當平凡並且被認為是不必要的。該規則的目的是意圖放寬ISO要求達到與現代環境相對應的程度，並應確認支持實施31個字符/大小寫的有效性。
* 使用標識符名稱要注意的一个相關問題是發生在名稱之間只有一個字符或少數字符不同的情況，尤其是在名稱較長的情況。當名稱之間的區別是容易誤讀的字符，問題就比較明顯。比如 1(數字1)和(L的小寫)、0 和 O、2 和 Z、5 和 S，或者 n 和 h。
* 建議名稱的區別要顯而易見
* 在這個問題上的具體指導方針可以放在風格指南（參見4.2.2節）。

### Rule 5.2 (req) (by Noah)

* Identifiers in an inner scope shall not use the same name as an identifier in an outer scope, and therefore hide that identifier.
* 中文說明：在內作用範圍 (inner scope) 裡的標識符 (identifier) 不應與外部作用範圍 (outer scope) 的標識符有相同的名字，否則內作用範圍的標識符將覆蓋掉外作用範圍的。

* 內作用範圍 (inner scope) 和外作用範圍 (outer scope) 的定義如下所述，一個標識符 (identifier) 的作用範圍如果是整個檔案的話(file scope)，那就可以叫做是最外作用範圍 (outermost scope) ，如果一個標識符作用範圍是在一個區塊內的話(block scope)，就叫做是相對內作用範圍 (more inner scope) 。

* 此規定意在防範內作用範圍標識符定義覆蓋掉外部作用範圍標識符定義的情況發生。但凡第二次的標識符定義不會隱藏第一次的標識符定義，就不算違反此項規定。
* 範例：
  * 不合規定的寫法：

    ```c
    int16_t i;
    {
        int16_t i; /* This is a different variable */
                    /* This is not compliant */
        i = 3;     /* It could be confusing as to which i this refers */
    }
    ```

### Rule 5.3 (req) (by Mars)

* A typedef name shall be a unique identifier.
* 中文說明：`typedef` 的命名應是獨特的。
* 無論如何，一個 `typedef` 的名字都不應被重複使用為另一個類型定義的名字或用作其他目的。
* 範例:

    ```c
    {
      typedef unsigned char uint8_t;
    }

    /* Not compliant – redefinition */
    {
      typedef unsigned char uint8_t;
    }

    /* Not compliant – reuse of uint8_t */
    {
      unsigned char uint8_t;
    }
    ```

* 除了把 `typedef` 放在標頭檔 (header file) ，被多個源碼呼叫的情況外，在程式中的任何地方 (即使是完全相同的宣告) ，都不應該重複使用 `typedef` 的命名。

### Rule 5.4 (req) (by Ray)

* A tag name shall be a unique identifier.
* 中文說明：`tag` 名稱應是特別的標識符。
* `tag` 名稱不得再用於定義其他 `tag` 或是任何其他目的。
* ISO / IEC 9899：1990 [2]沒有定義在不同形式的類型說明符(`struct` 或 `union`)下，聚合聲明 (aggregate declaration) 使用一個一樣 `tag` 時的行為。
* `tag` 的所有用途都應該在 `struct` 類型說明符中，或者都在 `union` 類型說明符中。
* 範例：

    ```c
    struct stag { uint16_t a; uint16_t b; };
    struct stag a1 = { 0, 0 }; /* 合規定 - 以上兼容 */
    union stag a2 = { 0, 0 }; /* 不合規定 - 與之前聲明不兼容 */
    void foo(void)
    {
        struct stag { uint16_t a; }; /* Not compliant - tag stag redefined */
    }
    ```

* 即使定義完全相同，也不得在源代碼文件中的任何地方重複相同的 `tag` 定義。
* 如果在標頭檔 (header file) 中進行 `tag` 定義，並且該標頭檔包含在多個源文件中，則不會違反此規則。

### Rule 5.5 (adv) (by Liou)

* No object or function identifier with static storage duration should be reused.

* 中文說明：具有靜態存儲期 (static storage duration) 的 `object` 或 `function` 標識符不能被重複使用。

* 無論範圍 (scope) ，具有靜態存儲期 (static storage duration) 的變量或者函數 (包含具有外部連結的 `object` 與 `function` 以及以 `static` 宣告的 `object` 與 `function`)，都不應在任何源文件中被重複使用。即便編譯器能夠理解，使用者仍可能有機會誤用。此情形的一個例子便是一個擁有內部連結 (internal linkage) 的標識符與在另一個檔案擁有相同名稱但外部連結 (external linkage) 的標識符。

### Rule 5.6 (adv) (by U.Chen)

* No identifier in one name space should have the same spelling as an identifier in another name space, with the exception of structure member and union member names.
* 中文說明：一個命名空間 (name space) 中不應存在與另外一个命名空間中的標識符拼寫一樣的標識符 (identifier) ，除了 `structure` 和 `union` 中的成員名字。
* ISO C 定義了許多不同的命名空間 (詳見 ISO/IEC 9899: 1990 6.1.2.3[2]) 。技術上，在彼此獨立的命名空間中使用相同的名字以代表完全不同的項目是可能的，然而由於會引起混淆，通常不贊成這種做法，因此即使是在獨立的命名空間中名字也不能重複使用。
* 範例:
  * 不建議的用法，其中 value 在不经意中代替了 record.value：

    ```c
    struct { int16_t key ; int16_t value ; } record ;
    int16_t value; /* Rule violation – 2nd use of value */
    record.key = 1;
    value = 0; /* should have been record.value */
    ```

  * 相比之下，下面的例子没有違背此規則，因為兩個成員名字不會引起混淆：

    ```c
    struct device_q { struct device_q *next ; /* ... */ }
    devices[N_DEVICES] ;
    struct task_q { struct task_q *next ; /* … */ }
    tasks[N_TASKS];
    device[0].next = &devices[1];
    tasks[0].next = &tasks[1];
    ```

### Rule 5.7 (adv) (by Jackal)

* No identifier name should be reused.
* 中文說明：不得重複使用標識符名稱。
* 無論範圍如何，都不應在系統中的任何文件中重複使用標識符。本規則納入了規則5.2、5.3、5.4、5.5和5.6的規定。

    ```c
    struct air_speed
    {
        uint16_t speed;         /* knots */
    } * x;
    struct gnd_speed
    {
        uint16_t speed;         /* mph                                         */
                                /* Not Compliant - speed is in different units */
    } * y;
    x->speed = y->speed;
    ```

## Types

### Rule 6.1 (req) (by Weiren)

* The plain char type shall be used only for the storage and use of character values.
* 中文說明：單純的 `char` 類型應僅用於字符值的存儲和使用。

### Rule 6.2 (req) (by Noah)

* signed and unsigned char type shall be used only for the storage and use of numeric values.
* 中文說明：`signed char` 和 `unsigned char`只能拿來做數值資料存儲使用。

* char types有三種， `char` 、 `signed char` 和  `unsigned char` 。 `char` 只能拿來做字符或字串存儲使用，因為不同的編譯器可能會把 `char` 當做 `signed char` 或   `unsinged char`，所以不建議拿來儲存數值，要儲存數值的話需使用 `signed char` 或 `unsigned char` 。

### Rule 6.3 (adv) (by Mars)

* typedefs that indicate size and signedness should be used in place of the basic numerical types.
* 中文說明：應使用標示了大小和符號的 `typedef` 來取代基本的數值類型。
* 除了使用特定長度的 `typedef` 外，不應使用基礎數值的類型包含　`char` 、 `int` 、 `short` 、 `long` 、 `float` 、 `double` 的 `signed` 與 `unsigned` 變化。規則 6.3 幫助了我們區分儲存的大小，但因為整型提升 (integral promotion) 的不對稱行為，這些定義無法保證可攜性 ( 6.10 章節 )，了解整數大小的實作方式仍然是一件重要的事情。
* 程序員需要注意到 `typedef` 在這些定義下的實作方式。
* 舉例來說：下列是符合32bits的整數晶片並建議使用的範例，下述的 ISO (POSIX) `typedefs` 已被使用於這份文件的所有基本數值與字符型態：

    ```c
    typedef char char_t;
    typedef signed char int8_t;
    typedef signed short int16_t;
    typedef signed int int32_t;
    typedef signed long int64_t;
    typedef unsigned char uint8_t;
    typedef unsigned short uint16_t;
    typedef unsigned int uint32_t;
    typedef unsigned long uint64_t;
    typedef float float32_t;
    typedef double float64_t;
    typedef long double float128_t;
    ```

* 在位域的規範 (specification) 方面， `typedefs` 並不被視為是必要的。

### Rule 6.4 (req) (by Ray)

* Bit fields shall only be defined to be of type unsigned int or signed int.
* 中文說明：位域只應被定義為 `unsigned int`或 `signed int` 類型。
* 某些類型不太適合在位域中使用，因為它們的行為是實作定義(implementation defined)。
* 範例：
  * 不合規定的寫法：

    ```c
    int b:3; /* 取值的範圍可能是 0到7 或 -4到3。 */
    ```

  * 合規定的寫法：

    ```c
    unsigned int b:3;
    ```

### Rule 6.5 (req) (by Liou)

* Bit fields of signed type shall be at least 2 bits long.

* 中文說明：1 bit 長度的有符號位域是無用的。

## Constants

### Rule 7.1 (req) (by U.Chen)

* Octal constants (other than zero) and octal escape sequences shall not be used.
* 中文說明：不得使用八進制常數（零除外）和八進制跳脫序列。
* 任何以 `0` （零）開頭的整數常量都被視為八進制。 因此，存在編寫固定長度常量的危險。
* 八進制跳脫序列可能會出現問題，因為無意間引入了十進制數字會終止八進制跳脫並引入另一個字符。
* 範例：
  * 以下用於三位數總線消息的數組初始化將無法按預期方式進行：

    ```c
    code[1] = 109;   /* equivalent to decimal 109 */
    code[2] = 100;   /* equivalent to decimal 100 */
    code[3] = 052;   /* equivalent to decimal 42  */
    code[4] = 071;   /* equivalent to decimal 57  */
    ```

  * 第一個表達式的值是實作定義的，因為字符常量由兩個字符“\10”和“9”組成。第二個字符常量表達式包含單個字符“\100”。如果基本執行字符集中未表示字符64，則其值將由實作定義決定。

    ```c
    code[5] = '\109';   /* implementation-defined, two character constant */
    code[6] = '\100';   /* set to 64, or implementation-defined           */
    ```

* 最好不要使用八進制常量或跳脫序列，並且要檢查是否出現任何情況。整數常數零（寫為一個數字）嚴格來說是一個八進制常數，但這是該規則的允許例外。另外，`\0` 是唯一允許的八進制跳脫序列。

## Declarations and definitions

### Rule 8.1 (req) (by Jackal)

* Functions shall have prototype declarations and the prototype shall be visible at both the function definition and call.
* 中文說明：函數應具有原型聲明，並且原型在函數定義和調用處均需可見。

```c
 Example Code
 #include<stdio.h>
 main() {
     function(50);
 }

 void function(int x) {
     printf("The value of x is: %d", x);
 }

 Output
 The value of x is: 50
 This shows the output, but it is showing some warning like below:

 [Warning] conflicting types for 'function'

 [Note] previous implicit declaration of 'function' was here
 Now using function prototypes, it is executing without any problem.
 #include<stdio.h>
 void function(int); //prototype

 main() {
     function(50);
 }

 void function(int x) {
     printf("The value of x is: %d", x);
 }
 Output
 The value of x is: 50
```

### Rule 8.2 (req) (by Weiren)

* Whenever an object or function is declared or defined, its type shall be explicitly stated.
* 中文說明：每當宣告或定義一個物件或函數時，都應該明確表示其類型。
* 範例:
  * 不合規定的寫法：

    ```c
    extern x; /* Non-compliant – implicit int type */
    const y ; /* Non-compliant – implicit int type */
    static foo (void) ; /* Non-compliant – implicit type */
    ```

  * 合規定的寫法：

    ```c
    extern int16_t x ; /* Compliant – explicit type */
    const int16_t y ; /* Compliant – explicit type */
    static int16_t foo (void) ; /* Compliant – explicit type */
    ```

### Rule 8.3 (req) (by Mars)

* For each function parameter the type given in the declaration and definition shall be identical, and the return types shall also be identical.
* 中文說明：在宣告與定義中的每一個函數參數類型應該要相同，函數回傳的類型也要相同。
* 參數的型態跟回傳值在原型與定義上必須吻合，這要求不只是基本的型態，還包含了 `typedef` 名稱跟類型限定詞(const, volatile...)。

### Rule 8.4 (req) (by Noah)

* If objects or functions are declared more than once their types shall be compatible.
* 中文說明：如果一個物件或函數被重覆宣告，他們的形態必需是相容的。**但實際應用上應避免重覆宣告發生**。

### Rule 8.5 (req) (by Ray)

* There shall be no definitions of objects or functions in a header file.

* 中文說明：標頭檔中不應有物件或函數的定義。

* 標頭檔應用於聲明 `objects` 、 `functions` 、 `typedef` 與 `macros`。

* 標頭檔不得包含或產生佔用存儲空間的物件或函數定義，這清楚表明只有 C 文件包含可執行源代碼，而標頭檔僅包含聲明。

### Rule 8.6 (req) (by Liou)

* Functions shall be declared at file scope.

* 中文說明：函數應在文件範圍 (file scope) 內聲明。

* 範例：

  ```c
  class A {
  };

  void fun() {
    void nestedFun();  // Noncompliant; declares a function in block scope

    A a();      // Noncompliant; declares a function at block scope, not an object
  }
  ```

### Rule 8.7 (req) (by U.Chen)

* Objects shall be defined at block scope if they are only accessed from within a single function.
* 中文說明：如果僅從單個函數中訪問該 `object` ，則該 `object` 應在區塊範圍內定義。
* 在可能的情況下， 物件的範圍應限於函數 。文件範圍僅在物件需要具有內部或外部鏈接的情況下使用。在文件範圍內聲明物件的地方，適用規則8.10。除非必要，否則避免將標識符設置為全局是一種良好的做法。是否在最外層或最內層聲明物件在很大程度上取決於樣式。

### Rule 8.8 (req) (by Jackal)

* An external object or function shall be declared in one and only one file.
* 中文說明：外部物件或函數應在一個文件中聲明，並且只能在一個文件中聲明。

* 範例:

  ```c
  extern int16_t a;
  在featureX.h中，然後定義：

  #include <featureX.h>
  int16_t a = 0;
  一個項目中可能有一個標頭檔，也可能有很多標頭檔，但是每個外部對像或函數只能在一個標頭檔中聲明。
  ```

### Rule 8.9 (req)  (by Weiren)

* An identifier with external linkage shall have exactly one external definition.
* 中文說明：一個具有外部連結的標識符，應該具有精確的外部定義。
* 如果使用存在多個定義的標識符（在不同的檔案），或者使用根本不存在任何定義的標識符，這屬於 Undefined 行為。
* 不同文件中的多個定義是不允許的，即使它們定義是相同的。如果它們定義是不同的，或者將標識符初始化為不同的值，這顯然是很嚴重的。

### Rule 8.10 (req) (by Mars)

* All declarations and definitions of objects or functions at file scope shall have internal linkage unless external linkage is required.
* 中文說明：除了有外部連結外，所有在文件的範圍內的物件與函數的宣告與定義都應有內部連結。
* 如果變數僅使用在同一個文件內的函數，就使用 `static` 。同樣道理，如果函數僅在同一個文件內被呼叫，就使用 `static`。使用 `static` 儲存標示將確保僅在宣告的文件可以被識別，且可以避免與在其他文件或函數庫中相同的標識符 (identifiter) 相混淆。

### Rule 8.11 (req) (by Noah)

* The static storage class specifier shall be used in definitions and declarations of objects and functions that have internal linkage.
* 中文說明：在宣告任何內部連結的物件或函數時，應加上 `static` 這個關鍵字。

### Rule 8.12 (req) (by Ray)

* When an array is declared with external linkage, its size shall be stated explicitly or defined implicitly by initialisation.
* 中文說明：當使用外部鏈接聲明陣列時，其大小應明確聲明或通過初始化隱式定義。
* 範例：

  ```C
    int array1[10] ;                /* Compliant */
    extern int array2[] ;           /* Not compliant */
    int array2[] = { 0, 10, 15 };   /* Compliant */
  ```

* 儘管可以聲明不完整類型的陣列並訪問其元素，但是當可以顯式確定數組的大小時，這樣做更安全。

## Initialisation

### Rule 9.1 (req) (by Liou)

* All automatic variables shall have been assigned a value before being used.
* 中文說明：所有屬於自動儲存期的變量應在使用前進行初始化，以避免由於垃圾值引起的意外行為。

* 範例：
  * 不合規定的寫法：

    ```c
    int function(int flag, int b) {
      int a;
      if (flag) {
        a = b;
      }
      return a; // Noncompliant - "a" has not been initialized in all paths
    }
    ```

  * 合規定的寫法：

    ```c
    int function(int flag, int b) {
      int a = 0;
      if (flag) {
        a = b;
      }
      return a;
    }
    ```

### Rule 9.2 (req) (by U.Chen)

* Braces shall be used to indicate and match the structure in the non-zero initialisation of arrays and structures.
* 中文說明：應該使用大括號以指示和匹配陣列和 `structure` 的非零初始化構造。
* ISO C 要求 陣列、 `structure` 和 `union` 的初始化列表要以一對大括號括起来（盡管不這樣做的行為
是未定義的）。本規則更進一步地要求，使用附加的大括號来指示嵌套的 `structure`。
* 範例：
    二维陣列初始化的有效（在 ISO C 中）形式，但第一個與本規則相違背：

    ```c
    int16_t y[3][2] = { 1, 2, 3, 4, 5, 6 }; /* not compliant */
    int16_t y[3][2] = { { 1, 2 }, { 3, 4 }, { 5, 6 } } ; /* compliant */
    ```

* 此規則適用於
  * `structure`
  * `structure`、陣列及其他類型的嵌套组合中。
* 還要注意的是，陣列或 `structure` 的元素可以通過只初始化其首元素的方式初始化（為 `0` 或 `NULL`）。如果選擇了這樣的初始化方法，那麼首元素應该被初始化為 `0`（或 `NULL`），此時不需要使用嵌套的大括號。

### Rule 9.3 (req) (by Jackal)

* In an enumerator list, the `=` construct shall not be used to explicitly initialise members other than the first, unless all items are explicitly initialised.
* 中文說明：在枚舉數列表中，除非所有項目均已明確初始化，否則不得使用 `=` 來明確初始化除第一個成員以外的成員。

```c
enum colour { red=3, blue, green, yellow=5 };        /* non compliant */
   /* green and yellow represent the same value - this is duplication */
enum colour { red=3, blue=4, green=5, yellow=5 };        /* compliant */
   /* green and yellow represent the same value - this is duplication */

```

## Arithmetic type conversions

### Implicit type conversions

### Rule 10.1 (req) (by Weiren)

* The value of an expression of integer type shall not be implicitly converted to a different underlying type if:
    1. it is not a conversion to a wider integer type of the same signedness, or
    2. the expression is complex, or
    3. the expression is not constant and is a function argument, or
    4. the expression is not constant and is a return expression
* 中文說明：在下列情況下，整數類型的表達式的不得隱式轉換為其他基礎類型：
    1. 轉換不是向更寬的相同符號整數類型的轉換，或
    2. 表達式是複雜表達式，或
    3. 表達式不是常數，而是函數參數，或
    4. 表達式不是常數，而是一個回傳表達式

    ```C
        /* 1.*/
        u8a = u16a;  /*不合規，不是向更寬整數類型不得隱式轉換*/
        s8b = u8a;   /*不合規，不同符號的整數不得隱式轉換*/

        /* 2.*/
        s8a + u8a;  /*不合規，複雜表達式不得隱式轉換*/

        /* 3.*/
        void foo1(uint8_t x);
        foo1(s8a);  /*不合規，函數參數不得隱式轉換*/

        /* 4.*/
        int16_t t1(void)
        {
            return (u16a); /*不合規，回傳表示式不得隱式轉換*/
        }
    ```

### Rule 10.2 (req) (Mars)

* The value of an expression of floating type shall not be implicitly converted to a different type if:
    1. it is not a conversion to a wider floating type, or
    2. the expression is complex, or
    3. the expression is a function argument, or
    4. the expression is a return expression Notice
* 中文說明：下列狀況中，浮點數的表達不應有隱式轉換 ( 不同類型的數據混在一起，編譯器會自動換型態 ) 到其他型態的情況。
    1. 轉換不是向更大的浮點數型態轉換，或
    2. 表達式是複雜的，或
    3. 表達式是函數參數 ，或
    4. 表達式有回傳通知

### Explicit conversions (casts)明確類型轉換

### Rule 10.3 (req) (by Noah)

* The value of a complex expression of integer type shall only be cast to a type of the same signedness that is no wider than the underlying type of the expression.
* 中文說明：對任何複雜運算(複雜運算定義如MISRA-C 47頁所述)的結果做類型轉換時，只能轉換成同符號而且不能比預期運算結果還寬的類型。
* 範例：
  * 不合規定的寫法：

    ```c
    (uint32_t)(u16a + u16b)
    ```

  * 合規定的寫法：

    ```c
    (uint16_t)(u16a + u16b)
    (uint32_t)u16a + u16b
    ```

    此限制主要的目的是想確保寫程式的人自己清楚預期的運算結果，而不是依賴Integer promotion(定義於C99 55頁)的結果。
     Integer promotion:If an int can represent all values of the original type, the value is converted to an int; otherwise, it is converted to an unsigned int.
     像以下範例，如果依賴Integer promotion的話，在不同compiler就可能會有不同結果。

 ```c
    uint16_t u16a = 40000; // unsigned short / unsigned int?
    uint16_t u16b = 30000; // unsigned short / unsigned int?
    uint32_t u32x; // unsigned int / unsigned long?
    u32x = u16a + u16b; // u32x = 70000 or 4464?
 ```

* 20210621新增範例：
 YAMAHA BEE機種F-TRIP顯示不良問題，TRIPFUELKM為uint16變數，乘以1000後會造成overflow。

 ```c
 uint32_t DisplayBuffer;
 uint16_t TRIPFUELKM;

 DisplayBuffer = ((uint32_t)(TRIPFUELKM * 1000U)/ 1609); // Noncompliant;

 DisplayBuffer = (uint32_t)TRIPFUELKM * 1000U/ 1609; // Compliant;
 ```

### Rule 10.4 (req) (by Ray)

* The value of a complex expression of floating type shall only be cast to a floating type that is narrower or of the same size.
* 中文說明：浮點型複雜表達式的值只能轉換為更窄或相同大小的浮點型。
* 如果要在任何復雜表達式上使用強制類型轉換，則可以應用的強制類型將受到嚴格限制。
* 複雜表達式的轉換經常是混淆的來源，範例：

 ```C
 s8a + s8b
 ~u16a
 u16a >> 2
 foo (2) + u8a
 *ppc + 1
 ++u8a
 ```

* Rule 10.3與10.4範例：

 ```C
 ... (float32_t)(f64a + f64b)           /* compliant */
 ... (float64_t)(f32a + f32b)           /* not compliant */
 ... (float64_t)f32a                    /* compliant */
 ... (float64_t)(s32a / s32b)           /* not compliant */
 ... (float64_t)(s32a > s32b)           /* not compliant */
 ... (float64_t)s32a / (float32_t)s32b  /* compliant */
 ... (uint32_t)(u16a + u16b)            /* not compliant */
 ... (uint32_t)u16a + u16b              /* compliant */
 ... (uint32_t)u16a + (uint32_t)u16b    /* compliant */
 ... (int16_t)(s32a - 12345)            /* compliant */
 ... (uint8_t)(u16a * u16b)             /* compliant */
 ... (uint16_t)(u8a * u8b)              /* not compliant */
 ... (int16_t)(s32a * s32b)             /* compliant */
 ... (int32_t)(s16a * s16b)             /* not compliant */
 ... (uint16_t)(f64a + f64b)            /* not compliant */
 ... (float32_t)(u16a + u16b)           /* not compliant */
 ... (float64_t)foo1(u16a + u16b)       /* compliant */
 ... (int32_t)buf16a[u16a + u16b]       /* compliant */
 ```

### Rule 10.5 (req) (by Liou)

* If the bitwise operators ~ and << are applied to an operand of underlying type unsigned char or unsigned short, the result shall be immediately cast to the underlying type of the operand.
* 中文說明：如果位運算符 ~ 和 << 應用在基本類型為 unsigned char 或 unsigned short 的操作數，結果應該立即強制轉換為操作數的基本類型。
* 範例：

```C
uint8_t port = 0x5aU;
uint8_t result_8;
uint16_t result_16;
uint16_t mode;
result_8 = (~port) >> 4;
// Noncompliant;'~port' is 0xFFA5 on a 16-bit machine but 0xFFFFFFA5 on a 32-bit machine. Result is 0xFA for both, but 0x0A may have been expected.
```

這樣的危險可以通過如下所示的強制轉換來避免：

```C
result_8 = ((uint8_t)(~port)) >> 4 ; /*compliant */
result_16 = ((uint16_t)(~(uint16_t)port)) >> 4 /* *compliant */
```

### Rule 10.6 (req) (by U.Chen)

* A “U” suffix shall be applied to all constants of unsigned type.
* 中文說明：所有的unsigned類型都應該有後綴"U"
* 整數常量的類型是造成混淆的潛在原因，因為它取決於多種因素的複雜組合，其中包括
  * 常數的大小
  * 整數類型的實現大小
  * 存在任何後綴
  * 表示值的數字基數（即十進制，八進製或十六進制）
* 例如，整數常量“40000”在32位元的環境中為int類型，而在16位元的環境中為long類型。 值0x8000在16位元的環境中類型為unsigned int，但在32位元的環境中類型為（signed）int。
* 請注意以下幾點：
  * 任何帶有“U”後綴的值都是無符號類型
  * 小於 $2^{31}$的無後綴十進制值是帶符號類型
* 但是：
  * 大於或等於 $2^{15}$的不帶後綴的十六進制值可能是帶符號的或無符號的類型
  * 大於或等於 $2^{31}$的無後綴十進制值可能是帶符號或無符號類型
* 常數的符號應該明確。符號的一致性是建構良好形式的表達式的重要原則。如果一個常数是unsigned類型，為其加上“U”後綴將有助于避免混淆。當用在較大數值上時，後缀也许是多餘的；然而後缀的存在對程式的清晰性是種有價值的幫助。

### Rule 11.1 (req) (by jackal)

* Conversions shall not be performed between a pointer to a function and any type other than an integral type.
* 中文說明：不得在指向函數的指針與除整數類型以外的任何類型之間進行轉換
* 將函數指針轉換為其他類型的指針會導致未定義的行為。這意味著可以將函數指針轉換為整數類型或從整數類型轉換。不允許進行涉及函數指針的其他轉換。

    ```c

 int f（int a）
 {
      float（*p）（float）=（float（*）（float））&f; //不符合規定
 }
    ```

### Rule 11.2(req) (by Weiren)

* Conversions shall not be performed between a pointer to object and any type other than an integral type, another pointer to object type or a pointer to void.
* 中文說明：不得在指向對象的指標和任何其他類型之間進行轉換，除了整數類型、另一個指向對象的指標、或指向void的指標。

* 此類轉換是未定義行為。
* 此規則意味著可以將指向對象的指標轉換為：
    1. 一個整數類型
    2. 另一個指向對象的指標
    3. 指向void的指標

### Rule 11.3 (adv) (by Mars)

* A cast should not be performed between a pointer type and an integral type.
* 中文說明：指標與整數型態之間，不應該執行轉換。
* 當指針轉換到整數時，整數的記憶體大小是由實作來決定。盡可能避免在指針和整數類型之間進行強制轉換，但也許在記憶體位置對應暫存器或是在其他硬體的具體功能是無法避免的。

    ```c
    char *p ="abcd";
    int a = (int)p;
    func((char *)a);
    ```

### Rule 11.4 (adv) (by Noah)

* A cast should not be performed between a pointer to object type and a different pointer to object type.
* 中文說明：不同指標類型之間不應該做型別轉換，因為如果新的指標類型是需要強制對齊的話，轉換可能會失效。
* 範例：
  * 不合規定的寫法：

    ```c
    uint8_t * p1; /* for example, p1 = 0x03(奇數記憶體位址) */
    uint32_t * p2;
    p2 = (uint32_t *)p1; /* Incompatible alignment */
    ```

### Rule 11.5 (req) (by Ray)

* A cast shall not be performed that removes any const or volatile qualification from the type addressed by a pointer.
* 中文說明：如果指標所指向的類型帶有const或volatile限定符，那麼刪除限定符的強制轉換是不允許的。
* 通過強制轉換刪除類型限定符的任何嘗試都是違反了類型限定的原理。
* 範例：

 ```C
 uint16_t              x;
 uint16_t * const      cpi = &x;    /* const pointer */
 uint16_t * const      * pcpi;      /* pointer to const pointer */
 const uint16_t *      * ppci;      /* pointer to pointer to const */
 uint16_t *            * ppi;
 const uint16_t        * pci;       /* pointer to const */
 volatile uint16_t     * pvi;       /* pointer to volatile */
 uint16_t * pi;
 ...
 pi = cpi;                          /* Compliant - no conversion
                                                   no cast required */
 pi = (uint16_t *)pci;              /* Not compliant */
 pi = (uint16_t *)pvi;              /* Not compliant */
 ppi = (uint16_t * *)pcpi;          /* Not compliant */
 ppi = (uint16_t * *)ppci;          /* Not compliant */
 ```

## Expressions

### Rule 12.1(adv) (by Liou)

* Limited dependence should be placed on C’s operator precedence rules in expressions.

* 中文說明：有限的依存關係應放在運算符優先級上。運算符優先級的規則很複雜，可能導致錯誤。因此，在複雜的陳述中應使用括號來進行說明。但是，這並不意味著應該在每次操作前後都加括號。
* 範例：
  Compliant Solution

```C
x = a + b;
x = a * -1;
x = a + b + c;
x = f ( a + b, c );

x = ( a == b ) ? a : ( a - b );
x = ( a + b ) - ( c + d );
x = ( a * 3 ) + c + d;

if ( (a = f(b,c)) == true) { ... }
(x - b) ? a : c; // Compliant
(s << 5) == 1; // Compliant
```

### Rule 12.2 (req) (by U.Chen)

* The value of an expression shall be the same under any order of evaluation that the standard permits.
* 中文說明：在標准允許的任何運算次序下，表達式的值應相同。
* 除了少數運算符（特别的函數調用運算符 ( )、&&、| |、? : 和 , ）之外，子表達式所依據的運算次序是未指定的並會隨時更改。這意味着不能信任子表達式的運算次序，特别不能信任可能會發生副作用的運算次序。在表達式運算中的某些點上，如果能保證所有先前的副作用都已经發生，那麼這些點稱為“序列點”。序列點和副作用的描述見 ISO 9899:1990 [2]的 5.1.2.3 節、6.3 節和 6.6 節。
* 注意，運算次序的問题不能使用括號來解决，因為這不是優先級的問题。
* 下面的條款告诉我们對運算次序的依賴是如何發生的，並由此幫助我们採納本規則。
  * 自增或自减運算符\
    作為可能出問題的示例，請考慮

      ```c
      x = b[i] + i++;
      ```

    根據 b[i] 的運算是先于還是後於 i++ 的運算，表達式會產生不同的结果。把增值運算做為單獨的語句，可以避免這個问题。那麼：

      ```c
      x = b[i] + i;
      i++;
      ```

  * 函數參數\
    函數參數的運算次序是未指定的。

      ```c
      x = func ( i++, i);
      ```

    根據函數的兩個參數的運算次序不同，表達式會给出不同的结果。
  * 函數指针\
    如果函數是通過函數指針調用的，那麼函數標識符 (identifier )和函數參數運算次序是不可信任的。

      ```c
      p->task_start_fn (p++);
      ```

  * 函數調用\
    函數在被調用時可以具有附加的作用（如，修改某些全局數據）。可以通過在使用函數的表達式之前調用函數並為值採用臨時變量的方法避免對運算次序的依賴。例如

      ```c
      x = f (a) + g (a);
      ```

    可以寫成

      ```c
      x = f (a);
      x += g (a);
      ```

    作為可能出問題的示例，考慮下面的表達式，它從堆棧中取出兩個值，從第一個值中减去第二个值，再把结果放回棧中：

      ```c
      push ( pop () – pop () );
      ```

    根據哪一個 pop () 函數先進行计算（因为 pop()具有副作用）會產生不同的結果。
  * 嵌套的賦值语句\
    表達式中嵌套的賦值可以產生附加的副作用。不给這種能導致對運算次序的依賴提供任何機會的最好做法是，不要在表達式中嵌套賦值语句。\
    例如，下面的做法是不贊成的：

      ```c
      x = y = y = z / 3;
      x = y = y++;
      ```

  * volatile 訪問\
    類型限定符 volatile 是 C 提供的，用來表示那些其值可以獨立於程序的運行而自由更改的對象（例如输入寄存器）。對带有 volatile 限定類型的對象的訪問可能改變它的值。C 編譯器不會優化對 volatile 的讀取，而且，據 C 程序所關心的，對 volatile 的讀取具有副作用（改變 volatile 的值）。\
    做為表達式的一部分通常需要訪問 volatile 數據，這意味着對運算次序的依賴。建議對 volatile 的訪問儘可能地放在簡單的賦值语句中，如：

      ```c
      volatile uint16_t v;
      /* … */
      x = v;
      ```

    本規則討論了帶有副作用的運算次序問題。要注意子表達式的運算次數同樣會帶來問題，本規則沒有提及。這是函數調用的問題，其中函數是以巨集實現的。例如，考慮下面的函數巨集及其調用：

      ```c
      #define MAX (a, b) ( ( (a) > (b) ) ? (a) : (b) )
      /* … */
      z = MAX (i++, j);
      ```

    當 a > b 時，該定義計算了兩次第一個參數而在 a <= b 時只計算了一次。這樣，巨集調用根據 i 和 j 的值，對 i 增加了一次或兩次。\
    應該說明的是，比如那些由浮點的四捨五入引起的量級依賴的作用也沒有在這裡涉及。盡管可能發生副作用的運算次序是未定義的，運算结果在另一方面是良好定義的並被表達式的結構所控制。在下面的例子中，f1 和 f2 是浮點變量；F3、F4 和 F5代表浮點類型的表達式。

      ```c
      f1 = F3 + (F4 + F5);
      f2 = (F3 + F4 ) + F5;
      ```

    加法運算的次序由括號的位置决定，至少表面如此。即，首先 F4 的值加上 F5 然后加上F3，给出 f1 的值。假定 F3、F4 和 F5 沒有副作用，那麼它們的值獨立於它們被計算的次序。\
    然而，賦给 f1 和 f2 的值不能保證是相同的，因為浮點的四舍五入后緊接加法的運算將依賴於被加的值。

### Rule 12.3(req) (by Jackal)

* The sizeof operator shall not be used on expressions that contain side effects.

* 中文說明：sizeof運算符不得用於包含副作用的表達式。

    ```C

 int32_t i;
 int32_t j;
 j = sizeof(i = 1234);
              /*j is set to the sizeof the type of i which is an int*/
              /*i is not set to 1234.*/
    ```

### Rule 12.4(req) (by weiren)

* The right-hand operand of a logical && or || operator shall not contain side effects.
* 中文說明：邏輯運算符 && 或 || 的右側操作數不得含有副作用。
* 在C代碼中，有些情況下表達式的某些部分可能不會求值。如果這些子表達式包含副作用，這些副作用不一定會發生，這取決於其他的子表達式的值。
* 可能導致此問題的運算符是 && ， || ，和 ?:。對於前兩個（邏輯運算符），對右側操作數的求值取決於左側操作數的值。
* 對於 ?: 運算符，將對第二個或第三個操作數求值，但不會對兩者都求值。
* 如果程式設計師依賴於副作用，在兩種邏輯運算符其中之一的右操作數作條件求值很容易發生問題。
* ?: 運算符是在兩個子表達式之間進行選擇，因此不太可能發生錯誤。
* 範例：

    ```C
    if ( ishigh && ( x == i++ ) )  /* 不合規 */
    if ( ishigh && ( x == f(x) ) ) /* 僅在已經知道f（x）無副作用的情況下才合規定 */
    ```

* 在ISO / IEC 9899：1990 [2]的5.1.2.3節中將導致副作用的操作描述為訪問 volatile 的對像、修改對像、修改文件，或者是調用任何會執行這些操作的函數，函數所執行的這些操作可以導致函數所運行的環境狀態發生改變。

### Rule 12.5(req) (by Mars)

* The operands of a logical && or || shall be primary-expressions.
* 中文說明：邏輯運算符號 && 跟 || 應是主要運算式。
* 主要運算式 ( Primary expressions ) 定義在 ISO/IEC 9899:1990 [2] 的 6.3.1 節。本質上，他們可以是單一的標識符、常數或括起來的表達式。這個規則主要是用在，當不只一個標識符或常數被運算，就必須要括起來。括號在這種情況下，對於可讀性以及確保程式的預期行為來說非常重要。如果表達式只有使用邏輯 && 或 || 其中一種，就不需要使用括號。
* 示例：

    ```C
    if ( ( x == 0 ) && ishigh ) /* make x == 0 primary */
    if ( x || y || z ) /* exception allowed, if x, y and z are Boolean */
    if ( x || ( y && z ) ) /* make y && z primary */
    if ( x && ( !y ) ) /* make !y primary */
    if ( ( is_odd (y) ) && x ) /* make call primary */
    ```

* 如果表達式只有使用邏輯 && 或 || 的情況：

    ```C
    if ( ( x > c1) && (y > c2) && (z > c3) ) /* Compliant */
    if ( ( x > c1) && (y > c2) || (z > c3) ) /* not compliant */
    if ( ( x > c1) && ((y > c2) || (z > c3)) ) /* Compliant extra () used */
    ```

* 註記: Rule 12.5 對 Rule 12.1 來說是特例。

### Rule 12.6(adv) (by Noah)

* The operands of logical operators (&&, || and !) should be effectively Boolean.
  Expressions that are effectively Boolean should not be used as operands to operators other than
  (&&, ||, !,=, ==, != and ?:).

* 中文說明：一個邏輯運算式必需是有效的布林運算式(MISRA C 114頁)，
  不可和(&&, ||, !,=, ==, != and ?:)以外的運算元組合成一個運算式。

* 範例：

  * 不合規定的寫法：

    ```c
    a = ((b == c) + 5);
    ```

  * 合規定的寫法：

    ```c
    a = ((b == c) && d);
    ```

### Rule 12.7(req) (by Ray)

* Bitwise operators shall not be applied to operands whose underlying type is signed.
* 中文說明：位運算符不能用於基本類型(underlying type)是有符號的。
* 位運算符（〜，<<，<< =，>>，>> =，&，&=，^，^ =，|和|=）通常對有符號整數沒有意義。 例如，如果右移將符號位移至數字中，或者左移將數字位移至符號位，則可能會出現問題。

### Rule 12.8(req) (by Liou)

* The right-hand operand of a shift operator shall lie between zero and one less than the width in bits of the underlying type of the left-hand operand.

* 中文說明：移位運算符的右手操作數應該位於零和某數之間，這個數要小於左手操作數的基本類型的位寬。

* 範例：

  ```c
  u8a = (uint8_t) (u8a << 7); /* compliant */
   u8a = (uint8_t) (u8a << 9); /* not compliant */
   u16a = (uint16_t) ( (uint16_t) u8a << 9 ); /* compliant */
  ```

### Rule 12.9 (req) (by U.Chen)

* The unary minus operator shall not be applied to an expression whose underlying type is unsigned.
* 中文說明:一元減運算符不能用在基本類型無符號的表達式上。
* 把一元減運算符用在基本類型為 unsigned int 或 unsigned long 的表達式上時，會分别產生類型為 unsigned int 或 unsigned long 的结果，這是無意義的操作。把一元減運算符用在無符號短整型的操作數上，根據整數提升的作用它可以產生有意義的有符號结果，但這不是好的方法。

### Rule 12.10(req) (by Jackal)

* The comma operator shall not be used.
* 中文說明:不得使用逗號運算符。
* 逗號運算符的使用通常不利於代碼的可讀性，並且可以通過其他方式實現相同的效果。

    ```c

 i = a += 2, a + b;   // Noncompliant. What's the value of i ?

 a[1, 2] = 3;   // Noncompliant: 1 is ignored. This is not an access to a multidimensional array.

 x = a[i++, j = i + 1, j*2]; // Noncompliant. What index is used for a?
    ```

### Rule 12.11(adv) (by Weiren)

* Evaluation of constant unsigned integer expressions should not lead to wrap-around.
* 中文說明：無符號整數常數表達式的計算不應導致回繞(wrap-around)。
* 因為無符號整數表達式不會有真的溢位，但是會以 mod 的方式回繞，所以編譯器不會檢測到任何實際上是"溢位" 的無符號整數常數運算式。
* 儘管在運行時(run-time)可能有充分的理由依賴於無符號整數類型提供的 mod 計算方式，但在編譯時(compile-time)使用它來作常量表達式計算的原因卻不太明顯。因此，任何環繞的無符號整 數常數運算式實例都可能表示程式設計錯誤。
* 該規則同樣適用於翻譯過程的所有階段。
* 編譯器選擇在編譯時求值的常數運算式應確保結果與通過對目標求值獲得的結果相同，除了那些出現在條件預處理指令中的指令。
* 對於這樣的指令，可以使用常見的算術運算法則（見 ISO 9899:1990 [2]中 6.4 節），但是 int 和 unsigned int 類型的行為就好像它們分別是long和unsigned long一樣。
* 例如，在 int 類型為 16 位、long 類型為 32 位的機器上：

    ```c
        #define START 0x8000
        #define END 0xFFFF
        #define LEN 0x8000

        #if ((START + LEN) > END)
        #error Buffer Overrun
        /* OK because START and LEN are unsigned long */
        #endif

        #if (((END - START) - LEN) < 0)
        #error Buffer Overrun
        /* Not OK: subtraction result wraps around to 0xFFFFFFFF */
        #endif

        /* contrast the above START + LEN with the following */
        if ((START + LEN) > END)
        {
            error ("Buffer overrun");
            /* Not OK: START + LEN wraps around to 0x0000 due to unsigned int arithmetic */
        }
    ```

### Rule 12.12(req) (by Mars)

*The underlying bit representations of floating-point values shall not be used.

* 中文說明：浮點數不應使用的基本位元表示。
* 不應進行任何直接依賴於存儲方式的浮點操作，因為浮點數的存儲方式在不同的編譯器有可能不一樣。
* 應使用內建的運算元或函數進行浮點操作，因為這兩個方式的存儲細節都隱藏了存儲細節。

### Rule 12.13(adv) (by Noah)

* The increment (++) and decrement (--) operators should not be mixed with other operators in an expression.
* 中文說明：++和--不應該和其他運算元同時在一條運算式中使用，因為這樣程式可讀性比較差，也可能出現不是原本所預期的運算結果。
* 範例：
  * 不合規定的寫法：

    ```c
    u8a = ++u8b + u8c--;
    ```

  * 合規定的寫法：

    ```c
    ++u8b;
    u8a = u8b + u8c;
    u8c--;
    ```

## Rule 13.1 (req) (by Ray)

* Assignment operators shall not be used in expressions that yield a Boolean value.
* 中文說明：賦值運算符不得用於產生布爾值的表達式中。
* 在任何被認為具有布爾值的表達式中都不允許賦值。這排除了在布爾值表達式的操作數中同時使用簡單賦值運算符和復合賦值運算符。 但是，這並不排除將布爾值分配給變量。
* 如果布爾值表達式的操作數中需要賦值，則必須在這些操作數之外分別執行賦值。 這有助於避免混淆“ = ”和“ == ”，並有助於靜態檢測錯誤。

* 範例：

 ```C
 x = y;
 if ( x != 0 )
 {
  foo();
 }

 不能寫成：
 if ( ( x = y ) != 0 ) /* Boolean by context */
 {
  foo();
 }

 甚至更糟：
 if ( x = y )
 {
  foo();
 }
 ```

### Rule 13.2(adv) (by Liou)

* Tests of a value against zero should be made explicit, unless the
  operand is effectively Boolean.

* 中文說明：非布林值對零的測試應該是明確的

* 範例：

  此規則是為了清楚起見，並明確了整數和邏輯值之間的區別。

  ```c
  if ( x != 0 )  /* Correct way of testing x is non-zero                */
  if ( y )       /* Not compliant, unless y is effectively Boolean data */
                    (e.g. a flag)
  ```

### Rule 13.3 (req) (by U.Chen)

* Floating-point expressions shall not be tested for equality or inequality.
* 中文說明：浮點數不能做相等或不等的檢測。
* 這是浮點類型的固有特性，等值比較通常不會計算為 true，即使期望如此。而且，這種比較行為不能在執行前做出預測，它會隨著編譯器而不同。
* 範例
  下面程式碼的測試结果就是不可預期的：

    ```C
    float32_t x, y;
    /* some calculations in here */
    if ( x == y) /* not compliant */
        { /* … */ }
    if (x == 0.0f) /* not compliant */
    ```

  間接的檢測同樣是有問題的，在本規則内也是禁止的。

    ```c
    if ( ( x <= y ) && ( x >= y ) )
        { /* … */ }
    ```

* 為了獲得確定的浮點比较，建議寫一個實現比較運算的庫。這個庫應該考慮浮點的粒度以及參與比較的數的量級。見規則 13.4 和規則 20.3。

### Rule 13.4 (req) (by Jackal)

* The controlling expression of a for statement shall not contain
any objects of floating type.

* 中文說明：for語句的控製表達式不得包含任何浮點型對象。
* 控製表達式可以包括循環計數器，該循環計數器的值經過測試以確定循環的終止。

浮點變量不得用於此目的。

舍入和截斷錯誤可能會在循環的迭代過程中傳播，從而導致循環變量出現嚴重不正確，並且在執行測試時可能會產生意外結果。

例如，執行循環的次數可能因一種實現方式而異，並且可能是不可預測的。 另見規則13.3。

   ```c
 Noncompliant Code Example
 for (float counter = 0.0f; counter < 1.0f; counter += 0.001f) {
   ...
 }

 Compliant Solution
 for (int counter = 0; counter < 1000; ++counter) {
   ...
 }
   ```

### Rule 13.5(req) (by Weiren)

* The three expressions of a for statement shall be concerned only with loop control.
* 中文說明：for 語句的三個表達式只與循環控制有關。
* for 語句的三個表達式只能用於以下目的：
  * 第一個表達式:初始化循環計數器(下列範例中的 i)。
  * 第二個表達式:應包括測試循環計數器(i)，以及可選的其他循環控制變數（旗標）。
  * 第三個表達式:循環計數器(i)的遞增或遞減。

    ```c
        flag = 1;
        for ( i = 0; (i < 5) && (flag == 1); i++ )
        {
            /* ... */
        }
    ```

### Rule 13.6(req) (by Mars)

* Numeric variables being used within a for loop for iteration counting shall not be modified in the body of the loop.
* 中文說明：for迴圈用來疊代計數的變數數值不應該在本體內修改。
* 迴圈中用來疊代的變數數值不應該在本體內修改，但可以在循環中修改其他代表邏輯值的控制變數。比如說，一個表示已經完成的旗標。
* 範例：

    ```c
    flag = 1;
    for ( i = 0; (i < 5) && (flag == 1); i++ ){
        /* ... */
        flag = 0;  /* Compliant - allows early termination of loop */
        i = i + 3; /* Not compliant - altering the loop counter */
    }
    ```

### Rule 13.7(req) (by Noah)

* Boolean operations whose results are invariant shall not be permitted.
* 中文說明：一個布林運算式的結果如果是永遠為True或False的話，是不被允許的，因為有很高的可能性是寫錯了。
* 範例：
  * 不合規定的寫法：

    ```c
    if (u16a <= 0xffff) /* Not compliant - always true */

    if ((s8a < 10) && (s8a > 20)) /* Not compliant - always false */
    ```

### Rule 14.1 (req) (by Ray)

* There shall be no unreachable code.
* 中文說明：不應有無法訪問的代碼。
* 這條規則指的是在任何情況下都無法到達的代碼，並且可以在編譯時識別出這樣的代碼。
* 可以訪問但可能永遠不會執行的代碼從規則中排除（例如防禦性編程代碼）。
* 如果從相關入口點到該代碼之間不存在控制流路徑，則部分代碼將無法訪問。
* 範例，無法訪問無條件控制轉移後的未標記代碼：

 ```C
 switch (event)
 {
  case E_wakeup:
   do_wakeup ();
   break;          /* unconditional control transfer */
   do_more ();     /* Not compliant – unreachable code */
   /* … */
  default:
   /* … */
   break;
 }
 ```

* 如果沒有可以調用的方法，則整個函數將無法訪問。
* 預處理器指令排除的代碼在預處理後不存在，因此不受此規則的約束。

### Rule 14.2 (req) (by Liou)

* All non-null statements shall either:
  (a) have at least one side-effect however executed, or
  (b) cause control flow to change.

* 中文說明：

  所有非空語句（non-null statement）應該：

  a) 不管怎樣執行都至少有一個副作用（side-effect），或者

  b) 可以引起控制流的轉移

* 範例：

  ```c
  /* assume uint16_t x;
     and uint16_t i; */
  ...
  x >= 3u;   /* not compliant: x is compared to 3,
                and the answer is discarded */
  ```

Note that “null statement” and “side effect” are defined in ISO/IEC 9899:1990 [2] sections 6.6.3
and 5.1.2.3 respectively.

### Rule 14.3 (req) (by U.Chen)

* Before preprocessing, a null statement shall only occur on a line by itself; it may be followed by a comment provided that the first character following the null statement is a white-space character.
* 中文說明：在預處理之前，空語句只能出現在一行上；其後可以跟有註釋，前提是緊跟空語句的第一個字符是空格。
* 通常不會故意包含空語句，但是在使用它們的地方，它们應該出現在它們本身的行上。空語句前面可以有空格以保持縮進的格式。如果一條註釋跟在空語句的后面，那麼至少要有一個空格把空語句和註釋分隔開来。需要這樣起分隔作用的空格是因為它给讀者提供了重要的視覺信息。遵循本規則使得靜態檢查工具能夠為與其他文本出現在一行上的空語句提出警告，因為這樣的情形通常表示编程錯誤。
* 範例：

    ```c
    while ( ( port & 0x80 ) == 0 )
    {
        ; /* wait for pin – Compliant */
        /* wait for pin */ ; /* Not compliant, comment before ; */
        ;/* wait for pin – Not compliant, no white-space char after ; */
    }
    ```

### Rule 14.4 (req) (by Jackal)

* The goto statement shall not be used.

* 中文說明：不應使用 goto 語句。
* goto 是一個非結構化的控制流語句。 它使代碼的可讀性和可維護性降低。 應該使用結構化的控制流語句，例如 if、for、while、continue 或 break。

### Rule 14.5(req) (by Weiren)

* The continue statement shall not be used.
* 中文說明：不應該使用 continue 語句。
* 範例：
  * 不合規定的寫法：

    ```C
        int i;
        for (i = 0; i < 10; i++)
        {
            if (i == 5)
            {
                continue;
            }
            printf("i = %d\n", i);
        }
    ```

  * 合規定的寫法：

    ```C
        int i;
        for (i = 0; i < 10; i++)
        {
            if (i != 5)
            {
                printf("i = %d\n", i);
            }
        }
    ```

### Rule 14.6 (req) (by Mars)

* For any iteration statement there shall be at most one break statement used for loop termination.
* 中文說明：對於任何疊代的語句，最多只能有一個 break 去結束循環。
* 這些規則都是為了讓程式有更好的結構。只允許一個 break 的語句在迴圈中，是因為可以有雙重結果的產生或程式碼的優化。

### Rule 14.7 (req) (by Noah)

* A function shall have a single point of exit at the end of the function(IEC 61508 Part3).
* 中文說明：一個function結束的時候應該只有單一個離開點，這是由IEC 61508 Part3來的規範，是一個比較好的programming style。

### Rule 14.8 (req) (by Ray)

* The statement forming the body of a switch, while, do … while or for statement shall be a compound statement.
* 中文說明：構成 switch、while、do...while 或 for 語句主體的語句應是複合語句。
* 構成 switch 語句或 while、do...while 或 for 循環主體的語句應是複合語句（括在大括號內），即使該複合語句包含單個語句。
* 範例：

 ```C
 for (i = 0; i < N_ELEMENTS; ++i)
 {
  buffer[i] = 0;   /* Even a single statement must be in braces */
 }

 while ( new_data_available )
  process_data ();  /* Incorrectly not enclosed in braces */
  service_watchdog (); /* Added later but, despite the appearance (from the indent) it is actually not part of the body of the while statement, and is executed only after the loop has terminated */
 ```

### Rule 14.9(req) (by Liou)

* An if (expression) construct shall be followed by a compound statement. The else keyword shall be followed by either a compound statement, or another if statement.

* 中文說明：if（表達式）結構應該跟隨有復合語句。 else 關鍵字應該跟隨有復合語句或者另外的 if 語句。

* 範例：

  ```c
  if ( test1 )
  {
   x = 1;              /* Even a single statement must be in braces     */
  }
  else if ( test2 )    /* No need for braces in else if                 */
  {
   x = 0;              /* Single statement must be in braces            */
  }
  else
      x = 3;           /* This was (incorrectly) not enclosed in braces */
      y = 2;
  /* This line was added later but, despite the appearance (from the indent), it is actually not part of the else, and is executed unconditionally   */
  ```

### Rule 14.10 (req) (by U.Chen)

* All if … else if constructs shall be terminated with an else clause.
* 中文說明：所有 if ... else if 結構都應該以 else 子句終止。
* 當一個 if 語句後跟一個或多個 else if 語句時，此規則適用； 最後的 else if 後面應該跟一個 else 語句。 如果是簡單的 if 語句，則 else不需要包括聲明。最後一個 else 語句的要求是防禦性編程。 else 語句要麼採取適當的行動，要麼包含關於為什麼不採取行動的適當註解。 這與在 switch 語句 (規則15.3) 中具有最終默認子句的要求一致。
* 範例：
  * 簡單的 if 語句：

    ```c
        if ( x > 0)
        {
            log_error (3) ;
            x = 0 ;
        }             /* else not needed */
    ```

  * 包含了 if ... else if 的語句：

    ```c
        if ( x < 0 )
        {
            log_error(3);
            x = 0;
        }
        else if ( y < 0 )
        {
            x = 3;
        }
        else                 /* this else clause is required, even if the     */
        {                    /* programmer expects this will never be reached */
           /* no change in value of x */
        }
    ```

### Rule 15.0(req) (by Jackal)

* The MISRA C switch syntax shall be used.
* 中文說明：應使用 MISRA C 開關語法。
* C++ 中switch語句的語法很弱，允許複雜的、非結構化的行為。前面的文本描述了 MISRA C++ 定義的switch語句的語法。這和相關的規則對switch語句強加了一個簡單且一致的結構。

 ```c
 switch ( x )
 {
  case 0:
     ...
     break;   // break is required here

  case 1:     // empty clause, break not required

  case 2:
     break;   // break is required here

  default:    // default clause is required
     break;   // break is required here, in case a future
              // modification turns this into a case clause
 }
 ```

### Rule 15.1(req) (by Weiren)

* A switch label shall only be used when the most closely-enclosing compound statement is the body of a switch statement.
* 中文說明：switch label 只能在最封閉的複合語句是switch語句的主體時使用。
* 範例：
  * 不合規定的寫法：

    ```C
        void f (void)
        {
            switch (0)
            case 0: S;
        }
    ```

  * 合規定的寫法：

    ```C
        void f (void)
        {
            switch (1)
            {
                case 0: S;
            }
        }
    ```

### Rule 15.2(req) (by Mars)

* An unconditional break statement shall terminate every non-empty switch clause.
* 中文說明：每一個有內容的 switch 語法，都應該加上一個無條件的 break 來結束。
* 每個 switch 語法的最後一個語句應該是 break，或者如果 switch 語法是複合式的語句，在複合式語句的最後也應該要是 break。

### Rule 15.3 (req) (by Noah)

* The final clause of a switch statement shall be the default clause.
* 中文說明：為了防禦性編程，switch語法的最後都應該以default case來做結束。

### Rule 15.4 (req) (by Ray)

* A switch expression shall not represent a value that is effectively Boolean.
* 中文說明：switch 表達式不應表示一個有效的布爾值。
* 範例：

 ```C
 switch (x == 0) /* not compliant - effectively Boolean */
 {
  ...
 ```

### Rule 15.5(req) (by Liou)

* Every switch statement shall have at least one case clause.

* 中文說明：每個switch語句應至少包含一個case子句。

* 範例：

  ```c
  switch (x)
  {
     uint8_t var;
          /* not compliant - decl before 1st case     */
  case 0:
     a = b;
     break;
          /* break is required here                   */
  case 1:
          /* empty clause, break not required         */
  case 2:
     a = c;
          /* executed if x is 1 or 2                  */
     if ( a == b )
     {
        case 3:
         /* Not compliant - case is not allowed here */
     }
     break;
          /* break is required here                   */
  case 4:
     a = b;
          /* Not compliant - non empty drop through   */
  case 5:
     a = c;
     break;
  default:
          /* default clause is required               */
     errorflag = 1;
          /* should be non-empty if possible          */
     break;
          /* break is required here, in case a future                modification turns this into a case clause */
  }
  ```

### Rule 16.1 (req) (by U.Chen)

* Functions shall not be defined with a variable number of arguments.
* 中文說明：函數定義不得帶有可變數量的參數
* 本特性存在許多潛在的問題。使用者不應該編寫使用可變數量參數的附加函數。這排除了stdarg.h、va_arg、va_start 和 va_end 的使用。

### Rule 16.2(req) (by Jackal)

* Functions shall not call themselves, either directly or indirectly.
* 中文說明： 函數不得直接或間接調用自己。
* 這意味著不能在安全相關係統中使用遞歸函數調用。
* 遞歸帶有超出可用堆棧空間的危險，這可能是一個嚴重的錯誤。
* 除非遞歸受到非常嚴格的控制，否則無法在執行之前確定最壞情況下的堆棧使用情況。
* 範例：
  * 不合規定的寫法：

    ```C
        int pow(int num, int exponent) {
      if (exponent > 1) {
       num = num * pow(num, exponent-1);  // Noncompliant; direct recursion
      }
      return num;

 }
    ```

* 合規定的寫法：

    ```C
        int pow(int num, int exponent) {
      int val = num;
      while (exponent > 0) {
      val *= num;
      --exponent;
      }
      return val;

 }
    ```

### Rule 16.3(req) (by Weiren)

* Identifiers shall be given for all of the parameters in a function
prototype declaration.
* 中文說明： 應該為函數原型聲明中的所有參數提供標識符。
* 出於兼容性、清晰性和可維護性的原因，應該為函數聲明中的所有參數指定名稱。
* 範例：
  * 不合規定的寫法：

    ```C
        void divide (int, int);
    ```

  * 合規定的寫法：

    ```C
        void divide (int numerator, int denominator);
    ```

### Rule 16.4 (req) (by Mars)

* The identifiers used in the declaration and definition of a function shall be identical.
* 中文說明：函數中，標識符用於宣告與定義，都應該一致。
* 範例：

    ```C
    int myfunc(void){       /* declaration */
        int data=0;         /* definition */
        return data;
    }
    ```

### Rule 16.5 (req) (by Noah)

* Functions with no parameters shall be declared and defined with the parameter list void.
* 中文說明：如果一個function沒有return任何data，或是不需傳入任何參數，需宣告為void。
* 範例：

 ```C
 void myfunc (void);
 ```

### Rule 16.6 (req) (by Ray)

* The number of arguments passed to a function shall match the number of parameters.
* 中文說明：傳遞給函數的參數數量應與聲明的參數數量相匹配。
* 使用函數原型可以完全避免這個問題——參見規則8.1。 保留此規則是因為編譯器可能不會標記此約束錯誤。

### Rule 16.7(adv) (by Liou)

* A pointer parameter in a function prototype should be declared as pointer to const if the pointer is not used to modify the addressed object.

* 中文說明：如果不使用該函數原型中的指針參數來修改所尋址的對象，則該指針參數應聲明為const指針。

* 範例：

  ```c
  void myfunc( int16_t * param1, const int16_t * param2, int16_t * param3)
  /* param1: Addresses an object which is modified - no const
     param2: Addresses an object which is not modified - const required
     param3: Addresses an object which is not modified - const missing */
  {
     *param1 = *param2 + *param3;
     return;
  }
  /* data at address param3 has not been changed,
     but this is not const therefore not compliant */
  ```

### Rule 16.8 (req) (by U.Chen)

* All exit paths from a function with non-void return type shall have an explicit return statement with an expression.
* 中文說明： 帶有 non-void 返回類型的函數其所有退出路徑都應具有顯式的帶表達式的 return 語句。
* 表達式给出了函數的返回值。如果 return 語句不帶表達式，將導致未定義的行為（而且編譯器不會給出錯誤）。

### Rule 16.9 (req) (by Jackal)

* A function identifier shall only be used with either a preceding &, or with a parenthesised parameter list, which may be empty.
* 中文說明： 函數標識符只能與前面的 & 一起使用，或帶有括號的參數列表，該列表可能為空。
* 範例：

 ```C
 Noncompliant Code Example :
 int func(void) {
   // ...
 }

 void f2(int a, int b) {
    // ...
    if (func) {  // Noncompliant - tests that the memory address of func() is non-null
      //...
    }
    // ...
 }
 Compliant Solution
 void f2(int a, int b) {
    // ...
    if (func()) {  // tests that the return value of func() > 0
      //...
    }
    // ...
 }
 ```

### Rule 16.10(req) (by Weiren)

* If a function returns error information, then that error
information shall be tested.
* 中文說明：如果函數返回錯誤信息，則應該對錯誤信息做測試。
* 一個函數（無論它是標準庫的一部分、第三方庫還是用戶定義的函數）可以提供一些指示錯誤發生的方法，這可能是通過一個錯誤標誌、一些特殊的返回值或一些其他方式。每當函數提供這種機制時，調用程序應在函數返回後立即檢查錯誤指示。但是，請注意，與在函數完成後嘗試檢測錯誤相比，檢查函數的輸入值被認為是一種更可靠的錯誤預防方法（參見Rule 20.3）。另請注意，使用 errno（從函數返回錯誤信息）是笨拙的，應該小心使用（見Rule 20.5）。

## Pointers and arrays

### Rule 17.1 (req) (by Mars)

* Pointer arithmetic shall only be applied to pointers that address an array or array element.
* 中文說明：指標的運算應僅應用於陣列或陣列的成員中。
* 對於並非指向陣列或陣列成員中的指標做整數的加減運算(包含增、減值)會導致未定義的行為，

### Rule 17.2 (req) (by Noah)

* Pointer subtraction shall only be applied to pointers that address elements of the same array.
* 中文說明：指標相減只能應用在當這些指標是指到相同陣列時。
* 範例：

 ```C
 int array_1[10],array_2[10],p3;
 int* p1,*p2;

 p1 = &array_1[5];
 p2 = &array_1[3];
 p3 = p1 - p2;   /* compliant - p1和p2指向相同陣列 */

 p1 = &array_1[5];
 p2 = &array_2[3];
 p3 = p1 - p2;   /* not compliant - p1和p2指向不同陣列 */
 ```

### Rule 17.3 (req) (by Ray)

* &gt;, >=, <, <= shall not be applied to pointer types except where they point to the same array.
* 中文說明：>、>=、<、<= 不應應用於指針類型，除非它們指向相同的數組。
* 如果兩個指針不指向同一個對象，則嘗試在指針之間進行比較將產生未定義的行為。
* 注意：允許對數組末尾之外的下一個元素進行尋址，但不允許訪問該元素。

### Rule 17.4(req) (by Liou)

* Array indexing shall be the only allowed form of pointer arithmetic.

* 中文說明：數組索引應是唯一允許的指針算術形式。

* 範例：

  ```c
  void my_fn(uint8_t * p1, uint8_t p2[])
  {
     uint8_t index = 0U;
     uint8_t * p3;
     uint8_t * p4;
     *p1 = 0U;
     p1 ++;          /* not compliant - pointer increment               */
     p1 = p1 + 5;    /* not compliant - pointer increment               */
     p1[5] = 0U;     /* not compliant - p1 was not declared as an array */
     p3 = &p1[5];    /* not compliant - p1 was not declared as an array */
     p2[0] = 0U;
     index ++;
     index = index + 5U;
     p2[index] = 0U; /* compliant                                       */
     p4 = &p2[5];    /* compliant                                       */
  }
  uint8_t a1[16];
  uint8_t a2[16];
  my_fn(a1, a2);
  my_fn(&a1[4], &a2[4]);
  uint8_t a[10];
  uint8_t * p;
  p = a;
  *(p+5) = 0U;       /* not compliant                                   */
  p[5] = 0U;         /* not compliant                                   */
  ```

### Rule 17.5 (adv) (by U.Chen)

* The declaration of objects should contain no more than 2 levels of pointer indirection.
* 中文說明：對象聲明所包含的間接指針不得多於 2 級
* 多於 2 級的間接指針會嚴重減弱對程式碼行為的理解，因此應該避免。
* 範例：

    ```c
    typedef int8_t * INTPTR;
    struct s {
       int8_t *   s1;                             /* compliant     */
       int8_t **  s2;                             /* compliant     */
       int8_t *** s3;                             /* not compliant */
    };
    struct s *   ps1;                             /* compliant     */
    struct s **  ps2;                             /* compliant     */
    struct s *** ps3;                             /* not compliant */
    int8_t **  (  *pfunc1)();                     /* compliant     */
    int8_t **  ( **pfunc2)();                     /* compliant     */
    int8_t **  (***pfunc3)();                     /* not compliant */
    int8_t *** ( **pfunc4)();                     /* not compliant */
    void function( int8_t *   par1,
                   int8_t **  par2,
                   int8_t *** par3,               /* not compliant */
                   INTPTR *   par4,
                   INTPTR *   const * const par5, /* not compliant */
                   int8_t *   par6[],
                   int8_t **  par7[])             /* not compliant */
    {
       int8_t *   ptr1;
       int8_t **  ptr2;
       int8_t *** ptr3;                           /* not compliant */
       INTPTR *   ptr4;
       INTPTR *   const * const ptr5;             /* not compliant */
       int8_t *   ptr6[10];
       int8_t **  ptr7[10];
    }
    ```

* 類型解釋：
  * par1 和 ptr1 是指向 int8_t 的指針。
  * par2 和 ptr2 是指向 int8_t 的指針的指針。
  * par3 和 ptr3 是指向 int8_t 的指針的指針的指針。這是三級，因此不合適。
  * par4 和 ptr4 擴展開是指向 int8_t 的指針的指針。
  * par5 和 ptr5 擴展開是指向 int8_t 的指針的 const 指針的 const 指針。這是三級，因此不合適。
  * par6 是指向 int8_t 的指針的指針，因為陣列被轉化成指向陣列初始元素的指針。
  * ptr6 是 int8_t 類型的指針陣列。
  * par7 是指向 int8_t 的指針的指針的指針，因為陣列被轉化成指向陣列初始元素的指針。這是三級，因此不合適。
  * ptr7 是指向 int8_t 的指針的指針陣列。這是合適的。

### Rule 17.6 (req) (by Jackal)

* The address of an object with automatic storage shall not be assigned to another object that may persist after the first object has ceased to exist.
* 中文說明：自動存儲對象的地址不得分配給另一個對象,這可能會在第一個對像不復存在後持續存在。
* 如果一個自動對象的地址被分配給另一個更大範圍的自動對象，或者
一個靜態對象，或從函數返回，那麼包含地址的對象可能會在原始對象停止存在（並且其地址無效）的時間之後存在。
* 範例：

 ```C
 int8_t * foobar(void)
 {
     int8_t local_auto;
     return (&local_auto);   /* not compliant */
 }
 ```

## Structures and unions

### Rule 18.1(req) (by Weiren)

* All structure and union types shall be complete at the end of a translation unit.
* 中文說明： 所有 struct 和 union 類型應在翻譯單元的末尾完成。
* struct 和 union 的完整聲明應包含在讀取或寫入該結構的任何翻譯單元中。
* 指向不完整類型的指標本身是完整的並且是允許的，因此允許使用不透明指標。
* 有關不完整類型的完整描述，請參見 ISO 9899:1990 [2] 的第 6.1.2.5 節。
* 範例：
  * 不合規定的寫法：

    ```C
        struct tnode * pt; /* tnode 不完整 */
    ```

  * 合規定的寫法：

    ```C
        struct tnode * pt; /* tnode 在這一點上是不完整的 */
        struct tnode
        {
            int count;
            struct tnode * left;
            struct tnode * right;
        }; /* type tnode 現在已完整 */
    ```

### Rule 18.2(req) (by Mars)

* An object shall not be assigned to an overlapping object.
* 中文說明：一個物件不應該被指定到重疊分配的物件。
* 當兩個物件被創立並有重疊的記憶體位置且其中一個被複製成另外一個，這樣的行為是未定義的。

### Rule 18.3 (req) (by Noah)

* An area of memory shall not be reused for unrelated purposes.
* 中文說明：一個記憶體區塊不應該為了不相關的目的而重覆使用。
* 這項規定不允許你使用一塊記憶體儲存A資料，然後在不需要A資料的時段裡又用同一塊記憶體來儲存一個不相關的B資料，
但有時候這樣共用記憶體的方式在空間使用上是比較有效率的，例如union的使用，不過這樣的使用需要有文件紀錄(如18.4所述)。

### Rule 18.4 (req) (by Ray)

* Unions shall not be used.
* 中文說明：不得使用聯合。
* 規則 18.3 禁止將內存區域重用於無關目的。然而，即使內存被重用用於相關目的，仍然存在數據可能被誤解的風險。因此，該規則禁止出於任何目的使用聯合。
* 只要記錄了所有相關的實現定義的行為，對本規則的偏差就被認為是可以接受的。這可以通過參考設計文檔中編譯器手冊的實現部分在實踐中實現。
* 可能相關的實現行為類型有：
  * 填充 - 在聯合的末尾插入了多少填充
  * 對齊 - 聯合內任何結構的成員如何對齊
  * 字節序 — 是存儲在最低或最高內存地址的字的最高有效字節
  * 位序 - 字節內的位如何編號以及位如何分配給位域
* 如下情況是可以接受的：
  * (a)數據的打包和解包(Packing and unpacking data)，例如在發送和接收消息時
  * (b)實施變體記錄(Variant records)，前提是變體通過公共字段進行區分。沒有區分符(differentiator)的變體記錄被認為不適合在任何情況下使用。

* 數據的打包和解包(Packing and unpacking data)
* 在此示例中，聯合用於訪問32位字的字節，以便首先存儲通過網絡接收的字節最高有效字節。
* 此特定實現所依賴的假設是：
  * uint32_t 類型占用 32 位
  * uint8_t 類型占用 8 位
  * 實現將最高有效字節存儲在最低內存地址的字
* 實現字節接收和打包的代碼可以是：

 ```C
 typedef union {
  uint32_t word;
  uint8_t bytes[4];
 } word_msg_t;

 uint32_t read_word_big_endian (void)
 {
  word_msg_t tmp;
  tmp.bytes[0] = read_byte();
  tmp.bytes[1] = read_byte();
  tmp.bytes[2] = read_byte();
  tmp.bytes[3] = read_byte();
  return (tmp.word);
 }
 ```

 值得注意的是，上面的主體可以以可移植的方式編寫如下：

 ```C
 uint32_t read_word_big_endian (void)
 {
  uint32_t word;
  word = ((uint32_t)read_byte()) << 24;
  word = word | (((uint32_t)read_byte()) << 16);
  word = word | (((uint32_t)read_byte()) << 8);
  word = word | ((uint32_t)read_byte());
  return (word);
 }
 ```

 不幸的是，大多數編譯器在面對可移植實現時生成的代碼效率要低得多。
 當高執行速度或低程序內存使用比可移植性更重要時，使用聯合的實現可能是首選。

* 變體記錄(Variant records)
* 聯合通常用於實現變體記錄。 每個變體共享公共字段並具有特定於變體的附加字段。
* 此範例基於CAN校準協議(CCP)，其中發送到CCP客戶端的每個CAN消息共享兩個公共字段，每個字段為一個字節。最多可以跟隨6個附加字節，其解釋取決於存儲在第一個字節中的消息類型。
* 此特定實現所依賴的假設是：
  * uint16_t 類型占用 16 位
  * uint8_t 類型占用 8 位
  * 對齊和打包規則使得結構的 uint8_t 和 uint16_t 成員之間沒有間隙

* 為簡潔起見，此範例中僅考慮兩種消息類型。此處提供的代碼不完整，應僅用於說明變體記錄的目的，而不應視為CCP的模型實現。

 ```C
 /* 所有CCP消息共有的字段(The fields common to all CCP messages) */
 typedef struct {
  uint8_t msg_type;
  uint8_t sequence_no;
 } ccp_common_t;

 /* CCP connect message */
 typedef struct {
  ccp_common_t common_part;
  uint16_t station_to_connect;
 } ccp_connect_t;

 /* CCP disconnect message */
 typedef struct {
  ccp_common_t common_part;
  uint8_t disconnect_command;
  uint8_t pad;
  uint16_t station_to_disconnect;
 } ccp_disconnect_t;

 /* The variant */
 typedef union {
  ccp_common_t common;
  ccp_connect_t connect;
  ccp_disconnect_t disconnect;
 } ccp_message_t;

 void process_ccp_message (ccp_message_t *msg)
 {
  switch (msg->common.msg_type)
  {
  case Ccp_connect:
   if (MY_STATION == msg->connect.station_to_connect)
   {
    ccp_connect ();
   }
   break;
  case Ccp_disconnect:
   if (MY_STATION == msg->disconnect.station_to_disconnect)
   {
    if (PERM_DISCONNECT == msg->disconnect.disconnect_command)
    {
     ccp_disconnect ();
    }
   }
   break;
  default:
   break; /* ignore unknown commands */
  }
 }
 ```

## Preprocessing directives

### Rule 19.1(adv) (by Liou)

* #include statements in a file should only be preceded by other preprocessor directives or comments.

* 中文說明：文件中的#include語句僅應在其他預處理器指令或註釋之前。

* 範例：無。

* 補充說明：

  Noncompliant Code Example

  ```c
  #include <h1.h> /* Compliant */
  int32_t i;
  #include <f2.h> /* Noncompliant */
  ```

  Compliant Solution

  ```c
  #include <h1.h>
  #include <f2.h>

  int32_t i;
  ```

### Rule 19.2 (adv) (by U.Chen)

* Non-standard characters should not occur in header file names in #include directives.
* 中文說明：#include 指令中的標頭檔檔名中不應出現非標準字符。
* 在標頭檔檔名預處理標記的 < 和 > 限定符或 ” 和 ” 限定符之間使用了 ‘ ，\ ，或 /* 字符，該行為是未定義的。
* 不過如果開發環境的主機操作系統需要，則允許在文件名路徑中使用 \ 字符。

### Rule 19.3 (req) (by Jackal)

* The #include directive shall be followed by either a\<filename> or "filename" sequence.
* 中文說明："#include" 指令後面應該是 \<filename> 或 "filename" 序列

 ```c
 For example, the following are allowed.
 #include "filename.h"
 #include <filename.h>
 #define FILE_A "filename.h"
 #include FILE_A
  ```

### Rule 19.4 (req) (by Weiren)

* C macros shall only expand to a braced initialiser, a constant,a string literal, a parenthesised expression, a type qualifier, a storage class specifier, or a do-while-zero construct.
中文說明：C 的macros 應只能擴展 括弧初始化程序，常數，字符串文字，帶括號的表達式，類型修飾符，存儲類別說明符，或 do-while-zero 構造。
* Macros 只能允許這些用途。存儲類別說明符 和 類型修飾符 包括關鍵字，例如 extern、static 和 const。
* #define 的任何其他使用都可能導致在進行替換時出現意外行為，或者導致非常難以閱讀的代碼。
* 特別是，除了使用 do-while 結構外，Macro 不得用於定義語句或語句的一部分。
* Macro 也不應該重新定義語言的語法。
* Macro 替換列表中的任何類型 ( ) { } [ ] 的所有括號都應平衡。
* do-while-zero 構造（見下面的例子）是唯一允許在Macro 中包含完整語句的機制。
* do-while-zero 構造用於包裝一系列一個或多個語句並確保正確的行為。
* 注意：Macro 末尾的分號必須省略。

  * 不合規定的寫法：

    ```C
        #define int32_t long /* 改用 typedef */
        #define STARTIF if(  /* 括弧()不平衡而且語言重新定義 */
        #define CAT PI       /* 無括號表達式 */
    ```

  * 合規定的寫法：

    ```C
        #define PI 3.14159F                 /* 常數 */
        #define XSTAL 10000000              /* 常數 */
        #define CLOCK (XSTAL/16)            /* 常數表達式 */
        #define PLUS2(X) ((X) + 2)          /* Macro 擴展表達式 */
        #define STOR extern                 /* 存儲類別說明符 */
        #define INIT(value){ (value), 0, 0} /* 括弧初始化程序 */
        #define CAT (PI)                    /* 括號表達式 */
        #define FILE_A "filename.h"         /* 字符串文字 */
        #define READ_TIME_32() \
        do { \
        DISABLE_INTERRUPTS (); \
        time_now = (uint32_t)TIMER_HI << 16; \
        time_now = time_now | (uint32_t)TIMER_LO; \
        ENABLE_INTERRUPTS (); \
        } while (0)                         /* do-while-zero的例子 */
    ```

### Rule 19.5 (req) (by Mars)

* #Macros shall not be #define’d or #undef’d within a block.
* 中文說明：巨集內不應使用 #define 跟 #undef。
* 在C語言中的任何地方直接放上 #define 跟 #undef都是合法的，但如果把它們將它們放在塊中會產生誤導，因為這意味著範圍僅限於該區塊，但事實並非如此。
* 一般來說，#define 指令將放置在文件開頭附近，在第一個函數定義之前。
* 一般來說，#undef 指令不要用，請參考規則19.6。

### Rule 19.6 (req) (by Noah)

* #undef shall not be used.
* 中文說明：#undef不應該使用，它會容易造成混淆。

### Rule 19.7 (adv) (by Ray)

* A function should be used in preference to a function-like macro.
* 中文說明：應該優先使用函數而不是類似函數的宏。
* 雖然宏可以提供優於函數的速度優勢，但函數提供了更安全和更健壯的機制。
* 對於參數的類型檢查以及類似函數的宏可能多次評估參數的問題尤其如此。
* Noncompliant Code Example:

    ```c
    #define CUBE (X) ((X) * (X) * (X)) // Noncompliant

    void func(void)
    {
        int i = 2;
        int a = CUBE(++i); /* Noncompliant. Expands to: int a = ((++i) * (++i) * (++i)) */
        ...
    }
    ```

* Compliant Solution:

    ```c
    inline int cube(int i)
    {
        return i * i * i;
    }

    void func(void)
    {
        int i = 2;
        int a = cube(++i); // yields 27
        ...
    }
    ```

### Rule 19.8 (req) (by Liou)

* A function-like macro shall not be invoked without all of its arguments.

* 中文說明：不能在沒有所有參數的情況下調用類似函數的macro 。

* 範例：無。

### Rule 19.9 (req) (by U.Chen)

* Arguments to a function-like macro shall not contain tokens that look like preprocessing directives.
* 中文說明： 傳遞給類函數巨集的參數不能包含看似預處理指令的標記。
* 如果任何參數的行為類似預處理指令，使用巨集替代函數時的行為將是不可預期的。
* 範例：

    ```c
        #define SUM(A,B,C) ((A) + (B) + (C))

        void mc2_1909 ( void )
        {
           int16_t mc2_1909_s16a;
           int16_t mc2_1909_s16b = get_int16 ( );
           int16_t mc2_1909_s16c = get_int16 ( );

           mc2_1909_s16a = SUM( 5,
        #ifdef SW                               /* Noncompliant */
                                mc2_1909_s16b,
        #else
                                mc2_1909_s16c,
        #endif
                                0 );

        #ifdef SW                               /* Compliant */
           mc2_1909_s16a = SUM( 5, mc2_1909_s16b, 0);
        #else
           mc2_1909_s16a = SUM( 5, mc2_1909_s16c, 0);
        #endif

           use_int16 ( mc2_1909_s16a );
           use_int16 ( mc2_1909_s16b );
           use_int16 ( mc2_1909_s16c );

        }
    ```

### Rule 19.10 (req) (by Jackal)

* In the definition of a function-like macro each instance of a parameter shall be enclosed in parentheses unless it is used as the operand of # or ##.
* 中文說明： 在類函數宏的定義中,參數應括在括號中,除非它被用作# 或## 的操作數。

 ```c
 example define an abs function using:
 #define abs(x) (((x) >= 0) ? (x) : -(x))
 and not:
 #define abs(x) ((x >= 0) ? x : -x)

 Consider what happens if the second, incorrect, definition is substituted into the expression:
 z = abs( a - b );
 giving:
 z = ((a - b >= 0) ? a - b : -a - b);
 ```

* 補充說明： #與## 的操作數

 ```c
 宏中的#的功能是将其后面的宏参数进行字符串化操作（Stringizing operator），
 简单说就是在它引用的宏变量的左右各加上一个双引号。

 如定义好#define STRING(x) #x之后，下面二条语句就等价。

        char *pChar = "hello";

        char *pChar = STRING(hello);

 还有一个#@是加单引号（Charizing Operator）

 #define makechar(x)  #@x

        char ch = makechar(b);与char ch = 'b';等价。
  ```

 再讲下##的功能，它可以拼接符号（Token-pasting operator）。

 MSDN上有个例子：

 #define paster( n ) printf( "token"#n" = %d\n", token##n )

 int token9 = 100;

 再调用  paster(9);宏展开后token##n直接合并变成了token9。整个语句变成了

 printf( "token""9"" = %d", token9 );

 在C语言中字符串中的二个相连的双引号会被自动忽略，于是上句等同于

 printf("token9 = %d", token9);。

 即输出token9 = 100

### Rule 19.11 (req) (by Weiren)

* All macro identifiers in preprocessor directives shall be defined before use, except in #ifdef and #ifndef preprocessor directives and the defined() operator.
* 中文說明： 預處理器指令中的所有 macro 標識符都應在使用前定義，#ifdef 和 #ifndef 預處理器指令和 defined() 運算符除外。
* 如果嘗試在預處理器指令中使用標識符，而該標識符尚未定義，則預處理器有時不會發出任何警告，但會假定值為零。
* #ifdef、#ifndef 和defined() 用於測試 macro 是否存在，因此被排除在外。
* 在使用標識符之前，應考慮使用 #ifdef 測試。
* 請注意，預處理標識符可以通過使用 #define 指令或在編譯器調用時指定的選項來定義。但是，首選使用 #define 指令。
  * 不合規定的寫法：

    ```C
        #if x > 0 /* 如果未定義 x 假定為零 */
        #include SOMETHING_IMPORTANT
        #endif
    ```

  * 合規定的寫法：

    ```C
        #define x 10
        ...
        #if x > 0
        #include SOMETHING_IMPORTANT
        #endif

        #if defined ( y ) && ( y > 0 )
        ...
        #endif

### Rule 19.12 (req) (by Mars)

* There shall be at most one occurrence of the # or ## operators in a single macro definition.
* 中文說明：在一個巨集的定義中，應只能出現一個 # 或 ##。
* 因 # 和 ## 預處理器運算符未指定求值順序，為避免此問題，在任何單個巨集定義中只應使用一次任一運算符。

### Rule 19.13 (adv) (by Noah)

* The # and ## operators should not be used.
* 中文說明：#和##不應該使用，因為不同compiler可能會編譯出不同結果。

### Rule 19.14 (req) (by Ray)

* The defined preprocessor operator shall only be used in one of the two standard forms.
* 中文說明：定義的預處理器運算符只能以兩種標準形式之一使用。
* 定義的預處理器運算符僅有的兩種允許形式是：
  * defined ( identifier )
  * defined identifier

* 任何其他形式都會導致未定義的行為，例如：

    ```c
    #if defined(X > Y)  /* 不符合 - 未定義的行為 */

    #if defined X && defined Y && X > Y  /* 符合 */
    ```

* 在 #if 或 #elif 預處理指令擴展期間定義的令牌的生成也會導致未定義的行為，應避免，例如：

    ```c
    #define DEFINED defined
    #if DEFINED(X) /* not compliant - undefined behaviour */
    ```

### Rule 19.15 (req) (by Liou)

* Precautions shall be taken in order to prevent the contents of a header file being included twice.

* 中文說明：應採取預防措施以防止標頭檔的內容被包含兩次。

* 範例：

  ```c
  #ifndef LCD_MODULE_H
  #define LCD_MODULE_H
  ```

### Rule 19.16 (req) (by U.Chen)

* Preprocessing directives shall be syntactically meaningful even when excluded by the preprocessor.
* 中文說明：預處理指令在句法上應該是有意義的，即使是在被預處理器排除的情況下。
* 當預處理器指令排除一段源代碼時，每個排除語句的內容將被忽略，直到遇到 #else、#elif 或 #endif 指令（取決於上下文）。
* 如果這些被排除的指令之一的格式不正確，編譯器可能會在沒有警告的情況下忽略它，並帶來不幸的後果。
* 此規則的要求是所有預處理器指令即使出現在排除的代碼塊中也應在語法上有效。
* 特別是，確保#else 和#endif 指令後面沒有除空格之外的任何字符。 編譯器在執行此 ISO 要求方面並不總是一致。
* 範例：

    ```c
      #define AAA 2
      ...
      int foo(void)
      {
         int x = 0;
         ...
      #ifndef AAA
         x = 1;
      #else1         /* Not compliant */
         x = AAA;
      #else          /* Compliant */
         x = AAA;
      #endif
         ...
         return x;
      }
    ```

### Rule 19.17 (req) (by Jackal)

* All #else, #elif and #endif preprocessor directives shall reside in the same file as the #if or #ifdef directive   to which they are related.
* 中文說明：所有#else、#elif 和 #endif 預處理器指令應駐留在與它們所在的 #if 或 #ifdef 指令相同的文件中
  有關的。
* 範例：

    ```c

 file.c
 #define A
 ...
 #ifdef A
 ...
 #include "file1.h"

#

 #endif
 ...
 #if 1
 #include "file2.h"
 ...
 EOF

 file1.h
 #if 1
 ...
 #endif               /*Compliant*/
 EOF

 file2.h
 ...
 #endif               /*Not compliant*/
    ```

## Standard libraries

### Rule 20.1 (req) (by Weiren)

* Reserved identifiers, macros and functions in the standard library, shall not be defined, redefined or undefined.
* 中文說明： 標準庫中保留的標識符、macros 和 函數 不應被定義、重新定義或未定義。
* #undef 定義在標準庫中的 macro 通常是不好的做法。
* #define macro 名稱是 C 保留標識符、C 關鍵字或標準庫中任何 macro、對像或函數的名稱也是不好的做法。
* 例如，如果某些特定的保留字和函數名被重新定義或未定義，則它們會導致未定義的行為，包括 defined， __LINE__， __FILE__， __DATE__， __TIME__，__STDC__， errno and assert。
* 另見規則 19.6 關於#undef 的使用。
* 保留標識符由 ISO/IEC 9899:1990 [2] 第 7.1.3 節"Reserved identifiers"和第 6.8.8 節"Predefined macro names"定義。
* 標準庫中的 macro 是保留標識符的範例。
* 標準庫中的函數是保留標識符的範例。
* 標準庫中的任何標識符在任何情況（即在任何範圍或無論標題檔如何）中都被視為保留標識符。
* 7.13 "Future library directions" 中定義的保留標識符的定義、重新定義或取消定義是建議性的。
* 規則20.1適用於任何標頭檔。

### Rule 20.2 (req) (by Mars)

* The names of standard library macros, objects and functions shall not be reused.
* 中文說明：標準函數庫中的巨集、物件以及函數的名稱，不應重複使用。
* 當程序員使用了新版的標準函數庫中的巨集、物件以及函數的名稱（例如：增強行的功能或輸入數值檢查），被修改的巨集、物件以及函數的名稱都應該要有新的名字，這是為了避免混淆是否正在使用修改版本。
* 所以，舉例來說，如果寫了新版本的 “sqrt” 函數來檢查輸入是否為負，則新函數不應命名為 “sqrt”，應給予新的名稱。

### Rule 20.3 (req) (by Noah)

* The validity of values passed to library functions shall be checked.
* 中文說明：傳入library function的參數值都應該被檢查其有效性。
* 例如像許多math.h裡的函數，負數不能傳入sqrt或log函數，fmod的第二個參數不能為0等等。
* 有幾種應用方法可以滿足這條規則，包括:
  * 呼叫函數前先檢查參數值。
  * 在函數內做檢查處理，這適用在自製的library。
  * 做另一個函數把原函數包起來，然後在call原函數前先做檢查。
  * 透過靜態的確認傳入的參數永遠不會有無效值。
* 特別注意當檢查浮點數參數時，因為浮點數0是一個奇異點，所以建議要做等於0的檢查，雖然這違反了13.3的規定，
  不過這還是一個必要的檢查，避免function的運算結果趨近無限大，當參數值趨近於0時。

### Rule 20.4 (req) (by Ray)

* Dynamic heap memory allocation shall not be used.
* 中文說明：不得使用動態堆內存分配。
* 這排除了函數 calloc、malloc、realloc 和 free 的使用。
* 存在與動態內存分配相關的一系列未指定、未定義和實現定義的行為，以及許多其他潛在的陷阱。
* 動態堆內存分配可能會導致內存洩漏、數據不一致、內存耗盡、非確定性行為。
* 請注意，某些實現可能使用動態堆內存分配來實現其他功能（例如庫 string.h 中的功能）。 如果是這種情況，則也應避免使用這些功能。
* 範例()：

    ```c
    /* Noncompliant */
    int *b;
    void initialize()
    {
        b = (int*) malloc(1024 * sizeof(int)); // Noncompliant, could lead to an out-of-storage run-time failure.
        if (b == 0)
        {
            // handle case when dynamic allocation failed.
        }
    }

    /* Compliant */
    int b[1024]; // Compliant solution.
    ```

### Rule 20.5 (req) (by Liou)

* The error indicator errno shall not be used.

* 中文說明：不應使用錯誤指示符 errno。

* 範例：無。

### Rule 20.6 (req) (by U.Chen)

* The macro offsetof, in library <stddef.h>, shall not be used.
* 中文說明：不應使用<stddef.h>中的巨集 offsetof。
* 當這個巨集的操作數的類型不兼容或使用了位域時，它的使用會導致未定義的行為。
