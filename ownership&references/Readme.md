# [Aptos學習筆記#5]Move進階使用 - 所有權和引用 - Ownership & References

目錄

- 作用域中的變數
- move 和 copy
- 引用 References
- Borrow 檢查 Borrow Checking
- 取值運算 Dereferencing
- 引用原生類型 Referencing Primitive Types

## 一. 作用域中的變數

> 每個變數只有一個所有者作用域。當所有者作用域結束後，變數就會被清除
> 
- 所有者是擁有某變數的作用域
- 變數可以在作用域內被定義(像是在Script中使用let來定義變數)
- 也可以作為參數傳遞給作用域

- 每個變量只可以有一個所有者，所以當把變數作為參數傳遞給函數時，該函數就會是新的所有者，並且第一個函數已不再擁有該變數的所有權，也可以說第二個函數接管了變數的所有權
- 例子: 變數ａ先作為參數傳遞給了第一個函數，所以接下來當有第二個函數要使用變數a的時候就會報錯

Script: 設定一個變數a，先以參數傳遞到第一個函數，所有權轉給了第一個函數，第一個函數執行完後，沒有返回變數a，所以第二個函數執行的時候，就報錯了

```rust
script {
    use {{sender}}::M;

    fun main() {
        // Module::T 是一個結構體 struct
        let a : Module::T = Module::create(10);

				// 變數a離開了main函數
        // 且被放到了新的作用域 M::value 函數中
        M::value(a);

        // 變數已經不存在 main 函數中
	      // 這段程式碼不會編譯 compile
        M::value(a);
    }
}
```

Module: 函數中返回的是u8類型的參數值，參t已經被drop了，所以上面Script中當執行完第一個函式後，變數a被drop了，第二個函數接著執行的時候才會報錯

```rust
module M {

    struct T { value: u8 }

    public fun create(value: u8): T {
        T { value }
    }

    // 參數t被傳進了函數 value 中
    // value 函數拿走了它的所有權
    public fun value(t: T): u8 {
        t.value
    }
    // 函數value的作用域結束，t被刪除了，只有u8格式的值回傳
    // t 已經不存在
}
```

從結果來看，當函數 value() 結束時，t已經不存在了，返回的只是一個u8類型的值。

如何讓t依然可以用呢? 有一個快速的解決方法就是返回一個元組，該元組包含原始變數和其他結果，但 Move 還有一個更好的解決方案。

## 二. move 和 copy

Ｍove VM中有兩個字節碼: MoveLoc 和 CopyLoc，

反映到 Move 語言層面，它們分別為 move  和 copy

### 關鍵字 move

當變數傳到另一個函數時，MoveLoc指令會被使用，它會被 move

- 例子: 平常我們並不會顯示地使用關鍵字move，但程式碼依然可以正常運行，這邊我們顯示地使用來示範

```rust
script {
    use {{sender}}::M;

    fun main() {
        let a : Module::T = Module::create(10);
        
        // 變數a被移動到函數作用域中
        M::value(move a);

        // main作用域中的變數a被刪除了
    }
}
```

### 關鍵字 copy

使用時機: 想保留變數的值，同時僅將值的副本傳遞給某個函數

- 例子: 當我們第一次調用函數 value 時，將變數a的副本傳給函數，並保留a在本地作用域(main function)中，以便第二次調用函數時再次使用變數a

```rust
script {
    use {{sender}}::M;

    fun main() {
        let a : Module::T = Module::create(10);
        
        // 使用關鍵字 copy 來clone整個結構體
        // 也可以這樣使用 let a_copy = copy a
        M::value(copy a);
        // 變數a還在main作用域中，所以可以被使用
        M::value(a); // won't fail, a is still here
    }
}
```

**重要筆記:** 

- 使用 copy ，其實我們實際上複製了變數值，從而增加了程序佔用的內存。
- 複製的數據量如果很大，內存的消耗可能會很高
- 在區塊鏈中，交易執行佔用的內存資源是會消耗交易費的，每個Byte都會影響交易的執行費用，因此如果過多的使用 copy 會浪費很多交易費用

利用引用，可以幫助我們避免不必要的 copy 從而節省一些費用

## 三. 引用 - References

- 指向變數的鏈接(通常是內存中的某個片段)
- 可以將其傳遞到程序的其他部分，而無需移動變數值

> 引用 (使用&符號來表示) 讓我們可以使用直而無需擁有所有權
> 
- 例子: 在參數類型T前面加入了&引用符號，將參數類型T轉換成了T的引用&T

**Move  支援兩種類型的引用**

- 不可變引用 & (ex. &T): 不可變的引用讓我們可以在不更改值的情況下讀取值
- 可變引用 &mut (ex. &mut T): 可變引用給了我們讀取和更改值的能力
- 例子: 在Module中寫入兩個函數，一個使用不可變的引用，另一個使用可變的引用

Module

```rust
module M {
    struct T { value: u8 }

    // 回傳值為非引用的類型
    public fun create(value: u8): T {
        T { value }
    }

    // 不可變的引用，只允許讀取
    public fun value(t: &T): u8 {
        t.value
    }

    // 可變的引用允許讀取和改變其值
    public fun change(t: &mut T, value: u8) {
        t.value = value;
    }
}
```

Script

```rust
script {
    use {{sender}}::M;

    fun main() {
        let t = M::create(10);

        // 直接創建一個引用
        M::change(&mut t, 20);

        // 將一個引用賦予到一個變數
        let mut_ref_t = &mut t;

        // 使用可變的引用，並改變其值為100
        M::change(mut_ref_t, 100);

        // 不可變的引用
        let value = M::value(&t);

        // this method also takes only references
        // printed value will be 100
        0x1::Debug::print<u8>(&value);
    }
}
```

> 使用不可變的引用(&)從結構體中讀取數據，使用可變的引用(&mut)來修改它們
> 

> 透過使用適當類型的引用，我們可以更加安全地讀取模塊，因為這樣的方式能告訴程式的閱讀者，該變數是否會被修改
> 

## 四. Borrow 檢查 - Borrow Checking

Move 控制了我們使用參考(Reference)的方式，這樣可以防止意外發生

- 例子: 透過例子了解這個運作方法

Module

```rust
module Borrow {

    struct B { value: u64 }
    struct A { b: B }

    // 用內部B來創建A
    public fun create(value: u64): A {
        A { b: B { value } }
    }

    // 給一個可變的引用到內部B
    public fun ref_from_mut_a(a: &mut A): &mut B {
        &mut a.b
    }

    // 改變B
    public fun change_b(b: &mut B, value: u64) {
        b.value = value;
    }
}
```

Script

```rust
script {
    use {{sender}}::Borrow;

    fun main() {
        // 創建一個結構體(struct)A
        let a = Borrow::create(0);

        // 從mut A中獲取可變的引用B
        let mut_a = &mut a;
        let mut_b = Borrow::ref_from_mut_a(mut_a);

        // 改變B
        Borrow::change_b(mut_b, 100000);

        // 從A中獲得另一個可變的引用
        // get another mutable reference from A
        let _ = Borrow::ref_from_mut_a(mut_a);
    }
}
```

結果: 執行上是沒有問題的，我們使用對A的可變引用來獲得其內部結構B的可變引用(&mut a.b)，然後改變B

如果我們交換了最後兩段表達式，先創建一個新的A可變引用，而B的可變引用依然存在，會如何呢?

```rust
let mut_a = &mut a;
let mut_b = Borrow::ref_from_mut_a(mut_a);

let _ = Borrow::ref_from_mut_a(mut_a);

Borrow::change_b(mut_b, 100000);
```

程式會報錯

```rust
┌── /scripts/script.move:10:17 ───
    │
 10 │         let _ = Borrow::ref_from_mut_a(mut_a);
    │                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Invalid usage of reference as function argument. Cannot transfer a mutable reference that is being borrowed
    ·
  8 │         let mut_b = Borrow::ref_from_mut_a(mut_a);
    │                     ----------------------------- It is still being mutably borrowed by this reference
    │
```

這段程式無法編譯，因為 &mut A 被 &mut B 借用了，如果我們在對它的內容具有可變引用的時候同時更改 A，就會陷入一個狀況，就是A可以更改但是對它內容的引用依然還在此處，如果沒有實際的B，mut_b會指向哪裡?

**我們得到一些結論**

1. Move透過Borrow Checking阻止了一些狀況，造成編譯錯誤，編譯器建構一個 Borrow 圖並禁止移動借用(borrow)的值，這就是Move在區塊鏈中很安全的原因之一
2. 我們可以從引用(reference)創建引用(reference)，這樣新的引用將會借用原始的引用，可變引用可以創建不可變與可變的引用，但不可變引用僅能創建不可變引用
3. 當引用被借用時，它不能任意被移動，因為其它值依賴於它

## 五. 取值運算 - Dereferencing

透過取值運算來獲得引用所指向的值 - 符號使用星號*表示

> 取值運算實際上產生了一個副本，但要確定這個值具有 Copy 這個 ability 能力
> 

```rust
module M {
    struct T has copy {}
    
    // 這邊的t值是一個引用類型
    public fun deref(t: &T): T {
        *t
    }
}
```

> 取值運算不會將原始值移動move到目前的作用域中，而是實際上生成了一個副本
> 

```rust
module M {
    struct H has copy {}
    struct T { inner: H }

    // ...

    // 我們透過不可變的引用一樣可以做到
    public fun copy_inner(t: &T): H {
        *&t.inner
    }
}
```

有一個方法可以用來複製一個結構體的內部字段(fields)，使用*&來引用並取值

透過使用*&，我們複製了結構體的內部值

## 六. 引用原生類型 - Referencing primitive types

- 原生類型，由於他們的簡單性，所以不需要作為引用傳遞，而是會以複製的方式操作
- 即使我們按值將它們傳遞給函數，它們也將保留在當前的範圍之中
- 可以刻意的使用move關鍵字來移動，但由於原生類型的大小非常小，所以複製它們甚至比透過引用傳遞或是移動move它們更便宜

```rust
script {
    use {{sender}}::M;

    fun main() {
        let x = 10;
        M::do_smth(a);
        let _ = x;
    }
}
```

即使我們沒有使用引用來傳遞x，這個script也會編譯，添加copy是不必要的，因為VM已經將它放入