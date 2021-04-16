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

* 中文說明 : 不得依賴未定義或未指定的行為

* 該規則要求避免依賴於未定義和未指定的行為，而其他規則未專門解決該行為。 如果另一條規則明確涵蓋了特定行為，則僅在需要時才需要偏離該特定規則。

### Rule 1.3 (req)

### Rule 1.4 (req)
* The compiler/linker shall be checked to ensure that 31 character significance and case sensitivity are supported for external identifiers.
* 中文說明 : 應當檢查編譯器/鏈接器，以確保外部標識符支持31個字符的重要性和區分大小寫。
* [未定義7; 實施5、6]
* ISO標準要求外部標識符的前6個字符必須不同。但是，由於大多數編譯器/鏈接器至少允許31個字符有效（對於內部標識符而言），因此遵守此嚴格且無用的限制被認為是不必要的限制。必須檢查編譯器/鏈接器以確定此行為。如果編譯器/鏈接器不能夠滿足此限制，則使用編譯器的限制。

### Rule 1.5 (adv) (by Mars)

* Floating-point implementations should comply with a defined floating-point standard.
* 中文說明 : 浮點數的實作應該要遵守浮點數定義標準

* 浮點運算會帶來許多問題，其中有一些問題可以透過公認的標準來克服，ANSI/IEEE Std 754 [21] 就是一個合適的標準，規則 6.3 關於浮點數類型的定義提供了一個正在用的例子，如：
```
    /* IEEE 754 single precision floating point */
    typedef float float32_t;
```

## Language extensions

### Rule 2.1 (req) (by Noah)

* Assembly language shall be encapsulated and isolated in either (a) assembler functions, (b) C functions or (c) macros.
* 中文說明：組語必需是以(a) assembler functions、(b)C functions或是(c) macros的形式被獨立包起來使用
* 範例：
    * 不合規定的寫法：
        <pre><code>  void fn ( void )
        {
            DoSomething ( );
            asm ( "NOP" ); // Noncompliant, asm mixed with C/C++ statements
            DoSomething ( );
        }
        </code></pre>
    * 合規定的寫法：
        <pre><code>  void Delay ( void )
        {
            asm ( "NOP" ); // Compliant, asm not mixed with C/C++ statements
        }

        void fn ( void )
        {
            DoSomething ( );
            Delay ( ); // Compliant, Assembler is encapsulated
            DoSomething ( );
        }
        </code></pre>
### Rule 2.2 (req) (by Ray)

* Source code shall only use /* … */ style comments.
* 中文說明：源代碼只能使用/ *…* /樣式註釋。
* 在C90中不允許"//"的註釋樣式，甚至在C99之前不同的編譯器的行為可能有所不同。
* 範例：
    * 不合規定的寫法：
	    ```
		    // C99 style comments and C++ style comments
		```
    * 合規定的寫法：
        ```	
            /* Original C style comment
	                Can extend across multiple lines
            */
        ```

### Rule 2.3 (req)(by Liou)
*  The character sequence /* shall not be used within a comment.
* 中文說明 :字符序列/ *不得在註釋中使用。

*
	/* some comment, end comment marker accidentally omitted
	<<New Page>>
	Perform_Critical_Safety_Function (X);
	/* this comment is not compliant */ 
可能會省略掉註釋的結束標記，因此對安全性很高的功能將不被執行。

### Rule 2.4 (adv) (by U.Chen)
* Sections of code should not be “commented out”.
* 中文說明 : 程式碼區段不應該被"註釋掉"

* 當程式碼不需要被編譯執行時，應該用條件編譯來完成 ( 如：#if 或 #ifdef 加上註釋 )，用 /* 跟 */ 使程式碼不執行是危險的，因為C語言的編譯器不支援這樣的方式進行巢狀註釋(nested comments)，註釋失效後，程式碼中的原註釋內容可能會影響執行結果。

## Documentation

### Rule 3.1 (req)

### Rule 3.2 (req)
* The character set and the corresponding encoding shall be documented.
* 中文說明 : 字符集和相應的編碼應該文檔化。
* 例如，ISO 10646 [22]定義了字符集映射到數字值的國際標準。出於可移植性的考慮，字符常數和字串只能包含映射到已經文檔化的子集中的字符。源代碼以一個或多個字符集編寫。可選地，程序可以在第二個或多個字符集中執行。所有的源代碼和執行字符集都應映射到已經文檔化的子集中的字符

### Rule 3.3 (adv) (by Mars)
* The implementation of integer division in the chosen compiler should be determined, documented and taken into account.
* 中文說明：整數除法的實現在選擇的編譯器中，應該要被確認、記錄以及考慮到。

* 當兩個帶符號的整數做除法時，ISO相容的編譯器有潛在的可能產出兩個不同結果，
> 1. 編譯器可能會四捨五入(進位)並帶來負餘數或 (e.g. -5/3 = -1 remainder -2)
> 2. 也可能會四捨五入(捨去)並帶來正餘數 (e.g. -5/3 = -2 remainder +1)
* 重要的是，要確認後記錄下來並告訴程序員，編譯器在這兩種運算中屬於哪一種，尤其是第二種情況(比較少見)。

### Rule 3.4 (req) (by Noah)
* All uses of the #pragma directive shall be documented and explained.
* 中文說明：所有pragma的使用都需有文件來說明他的含意
* 範例：無

### Rule 3.5 (req) (by Ray)
* If it is being relied upon, the implementation defined behaviour and packing of bitfields shall be documented.
* 中文說明：實現定義(implementation defined)的行為和位域打包(packing of bitfields)都應該被記錄。
* 位域用於兩個主要用途：
> 1.以較大的數據類型（結合併集）訪問單個位或一組位。 不允許這種使用（請參見規則18.4）。
> 2.允許打包旗標或其他短長度數據以節省存儲空間。

* 建議特別聲明結構以保留位字段集，並且不要在同一結構內包含任何其他數據。
* 如果編譯器有一個開關來強制位字段遵循特定的佈局，那麼這可能會有助於這種證明。
* 範例：
    * 合規定的寫法：
        ```	
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

### Rule 3.6(req)(by Liou)
*  All libraries used in production code shall be written to comply with the provisions of this document, and shall have been subject 
   to appropriate validation. IEC 61508 Part 3
* 中文說明：生產代碼中使用的所有庫均應編寫為符合符合本文件的規定，並且應遵守進行適當的驗證。
* 範例:無

