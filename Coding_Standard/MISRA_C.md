

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
* 範例 : 無

### Rule 1.2 (req) (by U.Chen)
* No reliance shall be placed on undefined or unspecified behaviour.
* 中文說明 : 不得依賴[未定義或未指定的行為](https://en.cppreference.com/w/c/language/behavior)
* 該規則要求避免依賴於未定義和未指定的行為，而其他規則未專門解決該行為。 如果另一條規則明確涵蓋了特定行為，則僅在需要時才需要偏離該特定規則。

### Rule 1.3 (req) (by Jackal)
* Multiple compilers and/or languages shall only be used if there is a common defined interface standard for object code to which the languages/compilers/assemblers conform.
* 中文說明 : 僅當存在針對語言/編譯器/彙編器所遵循的目標代碼的通用定義接口標準時，才應使用多種編譯器和/或語言。
* 如果要使用C以外的語言來實現模塊或使用其他C編譯器進行編譯，則必須確保該模塊將與其他模塊正確集成。C語言行為的某些方面取決於編譯器，因此，對於所使用的編譯器，必須理解這些方面。
* stack使用情況，參數傳遞以及數據值的存儲方式（長度，對齊方式，鋸齒，重疊等）

### Rule 1.4 (req) (by Weiren)
* The compiler/linker shall be checked to ensure that 31 character significance and case sensitivity are supported for external identifiers.
* 中文說明 : 應當檢查編譯器/鏈接器，以確保外部標識符支持31個字符的重要性和區分大小寫。
* [未定義7; 實施5、6]
* ISO標準要求外部標識符的前6個字符必須不同。但是，由於大多數編譯器/鏈接器至少允許31個字符有效（對於內部標識符而言），因此遵守此嚴格且無用的限制被認為是不必要的限制。必須檢查編譯器/鏈接器以確定此行為。如果編譯器/鏈接器不能夠滿足此限制，則使用編譯器的限制。

### Rule 1.5 (adv) (by Mars)
* Floating-point implementations should comply with a defined floating-point standard.
* 中文說明 : 浮點數的實作應該要遵守浮點數定義標準
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
* Source code shall only use /\* … \*/ style comments.
* 中文說明：源代碼只能使用/\* … \*/樣式註釋。
* 在C90中不允許"//"的註釋樣式，甚至在C99之前不同的編譯器的行為可能有所不同。
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
* The character sequence /\* shall not be used within a comment.
* 中文說明 :字符序列 /\* 不得在註釋中使用。
    ```c
    /* some comment, end comment marker accidentally omitted
    <<New Page>>
    Perform_Critical_Safety_Function (X);
    /* this comment is not compliant */
    ```

可能會省略掉註釋的結束標記，因此對安全性很高的功能將不被執行。

### Rule 2.4 (adv) (by U.Chen)
* Sections of code should not be “commented out”.
* 中文說明 : 程式碼區段不應該被"註釋掉"
* 當程式碼不需要被編譯執行時，應該用條件編譯來完成 ( 如：#if 或 #ifdef 加上註釋 )，用 /\* 跟 \*/ 使程式碼不執行是危險的，因為C語言的編譯器不支援這樣的方式進行巢狀註釋(nested comments)，註釋失效後，程式碼中的原註釋內容可能會影響執行結果。

## Documentation

### Rule 3.1 (req) (by Jackal)
* All usage of implementation-defined behaviour shall be documented.
* 中文說明 : 實現定義的行為的所有用法應記錄下來。
* 該規則要求任何依賴於實現定義的行為（未由其他規則明確解決）都應記錄在案，例如，參考編譯器文檔。
* 如果另一條規則明確涵蓋了特定行為，則僅在需要時才需要偏離該特定規則。 有關這些問題的完整列表，請參見ISO / IEC 9899：1990附錄G [2]。

### Rule 3.2 (req) (by Weiren)
* The character set and the corresponding encoding shall be documented.
* 中文說明 : 字符集和相應的編碼應該文檔化。
* 例如，ISO 10646 [22]定義了字符集映射到數字值的國際標準。出於可移植性的考慮，字符常數和字串只能包含映射到已經文檔化的子集中的字符。源代碼以一個或多個字符集編寫。可選地，程序可以在第二個或多個字符集中執行。所有的源代碼和執行字符集都應映射到已經文檔化的子集中的字符

### Rule 3.3 (adv) (by Mars)
* The implementation of integer division in the chosen compiler should be determined, documented and taken into account.
* 中文說明：整數除法的實現在選擇的編譯器中，應該要被確認、記錄以及考慮到。
* 當兩個帶符號的整數做除法時，ISO相容的編譯器有潛在的可能產出兩個不同結果，
    1. 編譯器可能會四捨五入(進位)並帶來負餘數或 (e.g. -5/3 = -1 remainder -2)
    2. 也可能會四捨五入(捨去)並帶來正餘數 (e.g. -5/3 = -2 remainder +1)
* 重要的是，要確認後記錄下來並告訴程序員，編譯器在這兩種運算中屬於哪一種，尤其是第二種情況(比較少見)。

### Rule 3.4 (req) (by Noah)
* All uses of the #pragma directive shall be documented and explained.
* 中文說明：所有pragma的使用都需有文件來說明他的含意
* 範例：無

### Rule 3.5 (req) (by Ray)
* If it is being relied upon, the implementation defined behaviour and packing of bitfields shall be documented.
* 中文說明：實現定義(implementation defined)的行為和位域打包(packing of bitfields)都應該被記錄。
* 位域用於兩個主要用途：
    1. 以較大的數據類型（結合併集）訪問單個位或一組位。 不允許這種使用（請參見規則18.4）。
    2. 允許打包旗標或其他短長度數據以節省存儲空間。
* 建議特別聲明結構以保留位字段集，並且不要在同一結構內包含任何其他數據。
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

* 如果使用位字段，請注意潛在的陷阱和實現定義的行為（即非便攜式）。 尤其應該注意以下幾點：
    * 存儲單元中的位字段的對齊方式是實現定義的，即它們是從存儲單元的高端還是低端（通常是一個字節）分配的。
    * 位字段是否可以與存儲單元邊界重疊也由實現定義(例如：如果順序儲存一個6位字段和一個4位字段，那麼4位字段是全部從新的字節開始，還是其中2位佔據一個字節的剩餘2位，而其他2位開始於下個字節)。

### Rule 3.6 (req) (by Liou)
* All libraries used in production code shall be written to comply with the provisions of this document, and shall have been subject to appropriate validation. IEC 61508 Part 3
* 中文說明：生產代碼中使用的所有庫均應編寫為符合符合本文件的規定，並且應遵守進行適當的驗證。
* 範例:無

## Character sets

### Rule 4.1 (req) (by U.Chen)
* Only those escape sequences that are defined in the ISO C standard shall be used.
* 中文說明 : 只能使用 ISO C標準中定義的跳脫序列
* 僅允許使用ISO / IEC 9899：1990 [3-6]第6.1.3.4節和\0中的“簡單跳脫序列”。
禁止所有“十六進制跳脫序列”。
規則7.1也禁止使用\0以外的“八進制跳脫序列”。
* 簡單跳脫序列
    * \a 警報聲
    * \b Backspace
    * \f 換頁
    * \n 換行
    * \r 回車
    * \t 水平Tab
    * \v 垂直Tab
    * \\\ 反斜槓
    * \\' 單引號
    * \\" 雙引號
    * \\? 問號
    * \0 Null,什麼都不做

### Rule 4.2 (req) (by Jackal)
* Trigraphs shall not be used.
* 中文說明：不應使用三字母組合。
* 範例:
    ```c
    ??- represents a “~” (tilde) , ??) represents a “]”
    They can cause accidental confusion with other uses of two question marks.
    /* For example the string */
    "(Date should be in the form ??-??-??)"
    /* would not behave as expected, actually being interpreted by the compiler as */
    "(Date should be in the form ~~]"
    ```

## Identifiers

### Rule 5.1 (req) (by Weiren)
* Identifiers (internal and external) shall not rely on the significance of more than 31 characters.
* 中文說明：識別項（內部和外部）不得依賴於超過31個有效字符。
* ISO標準要求內部識別項的前31個字符必須不同，以確保代碼的可移植性。即使編譯器支持，也不應超過此限制。
* 這條規則應適用於所有名稱空間。
* 宏名稱也包括在內，且限制為31個字符適用於替換前後。
* ISO標準要求外部識別項的前6個字符必須不同，以確保最佳的可移植性。但是，此限制相當嚴格並且被認為是不必要的。該規則的目的是意圖放寬ISO要求達到與現代環境相對應的程度，並應確認支持實施31個字符/大小寫的有效性。
* 使用識別項名稱要注意的一个相關問題是發生在名稱之間只有一個字符或少數字符不同的情況，尤其是在名稱較長的情況。當名稱之間的區別是容易誤讀的字符，問題就比較明顯。比如 1(數字1)和(L的小寫)、0 和 O、2 和 Z、5 和 S，或者 n 和 h。
* 建議名稱的區別要顯而易見
* 在這個問題上的具體指導方針可以放在風格指南（參見4.2.2節）。

### Rule 5.2 (req) (by Noah)
* Identifiers in an inner scope shall not use the same name as an identifier in an outer scope, and therefore hide that identifier.
* 中文說明：內作用範圍(inner scope)和外作用範圍(outer scope)的定義如下所述，一個識別字(identifier)的作用範圍如果是整個檔案的話(file scope)，
  那就可以叫做是外作用範圍，如果一個識別字作用範圍是在一個區塊內的話(block scope)，就叫做是內作用範圍。
  此項目規定內作用範圍的識別字不可和外作用範圍的識別字使用相同名稱，因為第二次的內作用範圍識別字定義會把第一次的識別字定義隱藏起來，容易造成混淆。
  但如果第二次的識別字定義不會隱藏第一次的識別字定義，就不算違反此項規定。
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
* 中文說明：類型定義(typedef)的命名應是獨特的。
* 不管是什麼目的，typedef 都不應該重複定義相同的命名。
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

* 除了把 typedef 放在標頭檔，被多個源碼呼叫的情況外，在程式中的任何地方 (即使是完全相同的宣告)，都不應該重複使用 typedef 的命名。

### Rule 5.4 (req) (by Ray)
* A tag name shall be a unique identifier.
* 中文說明：Tag名稱應是唯一的標示符。
* tag名稱不得再用於定義其他tag或在程序內用於任何其他目的。
* ISO / IEC 9899：1990 [2]沒有定義當聚合聲明使用不同形式的類型說明符(struct或union)的tag時的行為。
* tag的所有用途都應該在struct類型說明符中，或者所有用途都應在union類型說明符中。
* 例如：
    ```c
    struct stag { uint16_t a; uint16_t b; };
    struct stag a1 = { 0, 0 }; /* 合規定 - 以上兼容 */
    union stag a2 = { 0, 0 }; /* 不合規定 - 與之前聲明不兼容 */
    void foo(void)
    {
        struct stag { uint16_t a; }; /* Not compliant - tag stag redefined */
    }
    ```

* 即使定義相同，也不得在源代碼文件中的任何地方重複相同的tag定義。
* 如果在頭文件中進行tag定義，並且該頭文件包含在多個源文件中，則不會違反此規則。

### Rule 5.5 (adv) (by Liou)
- No object or function identifier with static storage duration should be reused.

- 中文說明：具有靜態存儲期的對像或函數標識符不能重複使用。

- 範例：無。

- 補充說明：外部連結External,對應的關鍵字為extern 內部連結Internal，對應的關鍵字為static。

  ​				   用static修飾的變量或者函數的連結屬性為其作用領域只能僅限在本文件中 ，在其他文件中就不能     				   進行調用,不同文件中的內部函數是不會相互干擾的。

  ​				   Extern具有外部連結的識別項名稱所指定的函式或資料物件，會與任何其他具有外部連結的相同名     				   稱宣告所指定的函式或資料物件相同。

### Rule 5.6 (adv) (by U.Chen)
* No identifier in one name space should have the same spelling as an identifier in another name space, with the exception of structure member and union member names.
* 中文說明：一個命名空間中不應存在與另外一个命名空間中的標識符拼寫一樣的標示符，除了結構和聯合中的成員名字。
* ISO C 定義了許多不同的命名空間。技術上，在彼此獨立的命名空間中使用相同的名字以代表完全不同的項目是可能的，然而由於會引起混淆，通常不贊成這種做法，因此即使是在獨立的命名空間中名字也不能重複使用。
* 範例
    不建議的用法，其中 value 在不经意中代替了 record.value：
    ```c
    struct { int16_t key ; int16_t value ; } record ;
    int16_t value; /* Rule violation – 2nd use of value */
    record.key = 1;
    value = 0; /* should have been record.value */
    ```
    相比之下，下面的例子没有違背此規則，因為兩個成員名字不會引起混淆：
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
* 中文說明：單純的char類型應僅用於字符值的存儲和使用。

### Rule 6.2 (req) (by Noah)
* signed and unsigned char type shall be used only for the storage and use of numeric values.
* 中文說明：signed和unsigned char只能拿來做數值資料存儲使用。
  char types有三種，char、signed char和unsigned char。char只能拿來做字符或字串存儲使用，因為不同的compiler可能會把char當做signed char或unsinged char，
  所以不建議拿來儲存數值，要儲存數值的話需使用signed或unsigned char。

### Rule 6.3 (adv) (by Mars)
* typedefs that indicate size and signedness should be used in place of the basic numerical types.
* 中文說明：應使用標示了大小和符號的 typedefs 來取代基本的數值類型。
* 除了使用特定長度的 typedefs 外，不應使用的基礎數值類型包含　char, int, short, long, float, double 及其變化 signed, unsigned，
規則 6.3 幫助了我們了解儲存的大小，但因為不對稱行為的整型提升，這些定義無法保證可攜性 ( 6.10 章節 )，了解整數大小的實作方式仍然是一件重要的事情。
* 程序員需要注意到 typedefs 在這些定義下的實作方式。
* 舉例來說：下列是符合32bits的整數晶片並建議使用的範例，ISO (POSIX) typedefs 已被使用在這份文件的所有基本數值與字符型態：
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
* typedefs 不考慮長度是 bit 的型態。

### Rule 6.4 (req) (by Ray)
* Bit fields shall only be defined to be of type unsigned int or signed int.
* 中文說明：位域只能被定義為unsigned int或signed int類型。
* 某些類型不太適合在位域中使用，因為它們的行為是實現定義(implementation defined)。
* 範例：
    * 不合規定的寫法：
        ```c
        int b:3; /* 取值的範圍可能是0..7或-4..3。 */
        ```
    * 合規定的寫法：
        ```c
        unsigned int b:3;
        ```

### Rule 6.5 (req) (by Liou)
- Bit fields of signed type shall be at least 2 bits long.
- 中文說明：1 bit長度的有符號位域是無用的。
- 範例：無。

## Constants

### Rule 7.1 (req) (by U.Chen)
* Octal constants (other than zero) and octal escape sequences shall not be used.
* 中文說明：不得使用八進制常數（非零）和八進制跳脫序列。
* 任何以“ 0”（零）開頭的整數常量都被視為八進制。 因此，存在編寫固定長度常量的危險。
* 八進制跳脫序列可能會出現問題，因為無意間引入了十進制數字會終止八進制跳脫並引入另一個字符。
* 範例：
    * 以下用於三位數總線消息的數組初始化將無法按預期方式進行：
        ```c
        code[1] = 109;   /* equivalent to decimal 109 */
        code[2] = 100;   /* equivalent to decimal 100 */
        code[3] = 052;   /* equivalent to decimal 42  */
        code[4] = 071;   /* equivalent to decimal 57  */
        ```
    * 第一個表達式的值是實現定義的，因為字符常量由兩個字符“\10”和“9”組成。第二個字符常量表達式包含單個字符“\100”。如果基本執行字符集中未表示字符64，則其值將由實現定義。
        ```c
        code[5] = '\109';   /* implementation-defined, two character constant */
        code[6] = '\100';   /* set to 64, or implementation-defined           */
        ```
* 最好不要使用八進制常量或跳脫序列，並且要檢查是否出現任何情況。整數常數零（寫為一個數字）嚴格來說是一個八進制常數，但這是該規則的允許例外。另外，“\0”是唯一允許的八進制跳脫序列。

## Declarations and definitions

### Rule 8.1 (req) (by Jackal)
* Functions shall have prototype declarations and the prototype
shall be visible at both the function definition and call.
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
* 中文說明：在宣告與定義中的每一個函式參數類型應該要相同，函式回傳的類型也要相同。
* 參數的型態跟回傳值在原型與定義上必須吻合，這要求不只是基本的型態，還包含了 typedef names 跟 qualifiers(const, volatile...)。

### Rule 8.4 (req) (by Noah)
* If objects or functions are declared more than once their types shall be compatible.
* 中文說明：如果一個物件或函式被重覆宣告，他們的形態必需是相容的。**但實際應用上應避免重覆宣告發生**。

### Rule 8.5 (req) (by Ray)
* There shall be no definitions of objects or functions in a header file.

* 中文說明：標頭檔中不應有Objects或functions的定義。

* 標頭檔應用於聲明objects, functions, typedefs, and macros。

* 標頭檔不得包含或產生佔用存儲空間的objects或functions(或fragment of functions or objects)的定義，這清楚表明只有C文件包含可執行源代碼，而標頭檔僅包含聲明。



### Rule 8.6 (req) (by Liou)

- Functions shall be declared at file scope.

- 中文說明：函數應在文件範圍內聲明。

- 範例：無。

- 補充範例：

  ```
  class A {
  };

  void fun() {
    void nestedFun();  // Noncompliant; declares a function in block scope

    A a();      // Noncompliant; declares a function at block scope, not an object
  }
  ```



### Rule 8.7 (req) (by U.Chen)
* Objects shall be defined at block scope if they are only accessed from within a single function.
* 中文說明：如果僅從單個函數中訪問物件，則應在區塊範圍內定義。
* 在可能的情況下，物件的範圍應限於函數。文件範圍僅在物件需要具有內部或外部鏈接的情況下使用。在文件範圍內聲明物件的地方，適用規則8.10。除非必要，否則避免將標識符設置為全局是一種良好的做法。是否在最外層或最內層聲明物件在很大程度上取決於樣式。

### Rule 8.8 (req) (by Jackal)
* An external object or function shall be declared in one and only
one file.
* 中文說明：外部對像或函數應在一個文件中聲明，並且只能在一個文件中聲明。
```c
	Example:
	extern int16_t a;
	在featureX.h中，然後定義：

	#include <featureX.h>
	int16_t a = 0;
	一個項目中可能有一個頭文件，也可能有很多頭文件，但是每個外部對像或函		數只能在一個頭文件中聲明。
```

### Rule 8.9 (req)  (by Weiren)
* An identifier with external linkage shall have exactly one external definition.
* 中文說明：一個具有外部連結的識別項，應該具有精確的外部定義。
* 如果使用存在多個定義的識別項（在不同的檔案），或者使用根本不存在任何定義的識別項，這屬於Undefined行為。
* 不同文件中的多個定義是不允許的，即使它們定義是相同的。如果它們定義是不同的，或者將識別項初始化為不同的值，這顯然是很嚴重的。

### Rule 8.10 (req) (by Mars)
* All declarations and definitions of objects or functions at file scope shall have internal linkage unless external linkage is required.
* 中文說明：在文件的範圍內，除了有外部連結外，所有的物件與函式的宣告與定義都應有內部連結。
* 如果變數僅使用在同一個文件內的函式，就使用 static。相同的，如果函式僅在同一個文件內被呼叫，就使用 static。使用 static 儲存標示將確保僅在宣告的文件可以被識別，且可以避免在其他文件或函示庫中相同的標示符號混淆。

### Rule 8.11 (req) (by Noah)
* The static storage class specifier shall be used in definitions and declarations of objects and functions that have internal linkage.
* 中文說明：在宣告任何內部連結的物件或函式時，應加上static這個關鍵字。

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

- All automatic variables shall have been assigned a value before being used.
- 中文說明：變量應在使用前進行初始化，以避免由於垃圾值引起的意外行為。
- 範例：無。

- 補充範例：

Noncompliant Code Example

```
int function(int flag, int b) {
  int a;
  if (flag) {
    a = b;
  }
  return a; // Noncompliant - "a" has not been initialized in all paths
}
```

Compliant Solution

```
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
* 中文說明：應該使用大括號以指示和匹配陣列和結構的非零初始化構造。
* ISO C 要求陣列、結構和聯合的初始化列表要以一對大括號括起来（盡管不這樣做的行為
是未定義的）。本規則更進一步地要求，使用附加的大括號来指示嵌套的结构。
* 範例：
    二维陣列初始化的有效（在 ISO C 中）形式，但第一個與本規則相違背：
    ```c
    int16_t y[3][2] = { 1, 2, 3, 4, 5, 6 }; /* not compliant */
    int16_t y[3][2] = { { 1, 2 }, { 3, 4 }, { 5, 6 } } ; /* compliant */
    ```
* 在結構中以及在結構、陣列及其他類型的嵌套组合中，規則類似。
* 還要注意的是，陣列或結構的元素可以通過只初始化其首元素的方式初始化（为 0 或
NULL）。如果選擇了這樣的初始化方法，那麼首元素應该被初始化為 0（或 NULL），此時不
需要使用嵌套的大括號。
### Rule 9.3 (req) (by Jackal)
* In an enumerator list, the “=” construct shall not be used to explicitly initialise members other than the first, unless all items are explicitly initialised.
* 中文說明：在枚舉數列表中，除非所有項目均已明確初始化，否則不得使用“ =”結構來明確初始化除第一個成員以外的成員。

```
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
    3. 表達式是函式參數 ，或
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

	DisplayBuffer = ((uint32_t)(TRIPFUELKM * 1000U)/ 1609);	// Noncompliant;

	DisplayBuffer = (uint32_t)TRIPFUELKM * 1000U/ 1609;	// Compliant;
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

- If the bitwise operators ~ and << are applied to an operand of underlying type unsigned char or unsigned short, the result shall be immediately cast to the underlying type of the operand.
- 中文說明：如果位運算符 ~ 和 << 應用在基本類型為 unsigned char 或 unsigned short 的操作數，結果應該立即強制轉換為操作數的基本類型。
- 範例：

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
    		float（* p）（float）=（float（*）（float））&f; //不符合規定
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
- Limited dependence should be placed on C’s operator precedence rules in expressions.
- 中文說明：有限的依存關係應放在運算符優先級上。運算符優先級的規則很複雜，可能導致錯誤。因此，在複雜的陳述中應使用括號來進行說明。但是，這並不意味著應該在每次操作前後都加括號。
- 範例：
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
    如果函數是通過函數指針調用的，那麼函數標示符和函數參數運算次序是不可信任的。
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
              /* j is set to the sizeof the type of i which is an int */
              /* i is not set to 1234.                                */
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
* 主要運算式 ( Primary expressions ) 定義在 ISO/IEC 9899:1990 [2] 的 6.3.1 節。本質上，他們可以是單一的識別字、常數或括起來的表達式。這個規則主要是用在，當不只一個識別字或常數被運算，就必須要括起來。括號在這種情況下，對於可讀性以及確保程式的預期行為來說非常重要。如果表達式只有使用邏輯 && 或 || 其中一種，就不需要使用括號。
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

- The right-hand operand of a shift operator shall lie between zero and one less than the width in bits of the underlying type of the left-hand operand.

- 中文說明：移位運算符的右手操作數應該位於零和某數之間，這個數要小於左手操作數的基本類型的位寬。

- 範例：

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
	i = a += 2, a + b;  	// Noncompliant. What's the value of i ?

	a[1, 2] = 3; 		// Noncompliant: 1 is ignored. This is not an access to a multidimensional array.

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
* 應使用內建的運算元或函式進行浮點操作，因為這兩個方式的存儲細節都隱藏了存儲細節。

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

- Tests of a value against zero should be made explicit, unless the
  operand is effectively Boolean.

- 中文說明：非布林值對零的測試應該是明確的

- 範例：

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

- All non-null statements shall either:
  (a) have at least one side-effect however executed, or
  (b) cause control flow to change.

- 中文說明：

  所有非空語句（non-null statement）應該：

  a) 不管怎樣執行都至少有一個副作用（side-effect），或者

  b) 可以引起控制流的轉移

- 範例：

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
*  The goto statement shall not be used.

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
		buffer[i] = 0;			/* Even a single statement must be in braces */
	}

	while ( new_data_available )
		process_data ();		/* Incorrectly not enclosed in braces */
		service_watchdog ();	/* Added later but, despite the appearance (from the indent) it is actually not part of the body of the while statement, and is executed only after the loop has terminated */
	```

### Rule 14.9(req) (by Liou)

- An if (expression) construct shall be followed by a compound statement. The else keyword shall be followed by either a compound statement, or another if statement.

- 中文說明：if（表達式）結構應該跟隨有復合語句。 else 關鍵字應該跟隨有復合語句或者另外的 if 語句。

- 範例：

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
   		break;  	// break is required here

		case 1:    	// empty clause, break not required

		case 2:
   		break;  	// break is required here

		default:   	// default clause is required
   		break;  	// break is required here, in case a future
           			// modification turns this into a case clause
	}
	​```c
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

- Every switch statement shall have at least one case clause.

- 中文說明：每個switch語句應至少包含一個case子句。

- 範例：

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
          /* break is required here, in case a future 	              modification turns this into a case clause */
  }
  ```
### Rule 16.1(req)
### Rule 16.2(req)
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
* 中文說明：函式中，識別字用於宣告與定義，都應該一致。
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

- A pointer parameter in a function prototype should be declared as pointer to const if the pointer is not used to modify the addressed object.

- 中文說明：如果不使用該函數原型中的指針參數來修改所尋址的對象，則該指針參數應聲明為const指針。

- 範例：

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

