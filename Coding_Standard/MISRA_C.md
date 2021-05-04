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

### Rule 9.2 (req)
### Rule 9.3 (req)

## Arithmetic type conversions

### Implicit type conversions

### Rule 10.1 (req)

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

### Explicit conversions (casts)

### Rule 10.3 (req)
### Rule 10.4 (req)
### Rule 10.5 (req)


