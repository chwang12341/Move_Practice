# [Aptos學習筆記#6]Move進階使用 - 了解泛型 - Understanding Generics

## 了解泛型 - Understanding Generics

> 泛型在Move語言中是非常重要的，它讓Move在區塊鏈中很獨特，它是 Move 靈活性的重要來源
> 

## 泛型是什麼?

- RUST中定義: 泛型是具體類型或是其他屬性的抽象替代品
- 泛型讓大家可以僅編寫單個函數，而該函數可以應用於任何類型
- 此類型的函數被稱為模板: 一個可以應用於任何類型的模板處理程序
- Move中的泛型可以應用於結構體和函數的定義之中

## 結構體中定義的泛型

創建一個容納u8類型值的Box

```rust
module Storage {
    struct Box {
        value: u8
    }
}
```

很明顯這個Box只能包含u8類型的值，但是如果想要為u64類型或是bool類型創建相同的Box怎麼辦?

總不能為此創建u64類型的Box1和bool類型的Box2吧，這邊我們就可以使用泛型

```rust
module Storage {
    struct Box<T> {
        value: T
    }
}
```

這邊在結構體名字後面增加<T>，<…>裡面用來定義泛型，這邊T就是在結構體中模板化的類型，我們將T用作常規類型

其實類型T實際上並不存在，它是任何類型的佔位符

## 函數中的泛型 - In function signature

這邊我為這個結構創建一個構造函數，並以u64類型作為值

```rust
module Storage {
    // 構建一個泛型的結構體Box
    struct Box<T> {
        value: T
    }

    // 這邊<U64>被放進Box結構體中表示我們要使用類型為u64的Box結構體
    public fun create_box(value: u64): Box<u64> {
        Box<u64>{ value }
    }
}
```

如上帶有泛型的結構體定義起來有點複雜，因為我們需要指定它們的類型參數，像是如上範例需要把常規結構體的Box變成Box<u64>

上面這樣寫起來太麻煩，雖然Move中沒有限制<>中需要放什麼類型，但為了讓create_box方法更通用，有沒有更簡易的方法呢?

當然有，就是在函數中也使用泛型

```rust
module Storage {
    // 構建一個泛型的結構體Box
    struct Box<T> {
        value: T
    }
    
    // 在構建的函數中使用泛型
    public fun create_box<T>(value: T): Box<T> {
        Box<T> { value }
    }
    
    // 利用泛型函數獲得結構體中的值
    public fun value<T: copy>(box: &Box<T>): T {
        *&box.value
    }
}
```

## 函數中調用泛型

在Script中使用如何使用前面在Module中構建的泛型函數呢? 在函數調用中指定其類型即可

```rust
script {
    use {{sender}}::Storage;
    use 0x1::Debug;

    fun main() {
        // 利用泛型函數來創建一個bool類型的Box結構體
        let bool_box = Storage::create_box<bool>(true);
        // 取得bool_box(Box結構體)的值
        let bool_val = Storage::value(&bool_box);

        assert(bool_val, 0);

        // 利用泛型函數來創建一個u64類型的Box結構體
        let u64_box = Storage::create_box<u64>(1000000);
        // 取得u64_box(Box結構體)的值
        let _ = Storage::value(&u64_box);

        // 利用泛型函數來創建一個Box<u64>類型的Box結構體
        let u64_box_in_box = Storage::create_box<Storage::Box<u64>>(u64_box);

        // 要取得u64_box_in_box的值是非常有意思的
        // Box<u64>是一種類型，而Box<Box<u64>>也是一種類型
        // 這邊看起來用了兩層來取得該值
        let value: u64 = Storage::value<u64>(
            &Storage::value<Storage::Box<u64>>( // Box<u64> type
                &u64_box_in_box // Box<Box<u64>> type
            )
        );

        // 這邊我們已經能夠理解 Debug::print<T> 方法
        // 它也可以透過泛型的方法印出任何類型
        Debug::print<u64>(&value);
    }
}
```

這邊使用了三種類型來構建Box: u64, bool 和 Box<u64>，Box<u64>看起來比較複雜，但是只要我們習慣了，並且能夠了解泛型是如何運作的，它會成為我們非常重要的幫手

**重要筆記:**

透過將泛型添加到Box的結構體中，讓Box變得更抽象了，與它給我們功能相比，這種泛型的結構體定義起來相當簡單

我們可以創建任何類型的Box結構體，像是address, bool或u64等來創建另一個Box結構體或任何其它結構體

## abilities 限制符 - Contraints to check Abilities

前面的筆記中有提過了Move中的Abilities，它們一樣可以當成是泛型的限制符來使用，其使用的名稱和Abilities相同

```rust
**fun name<T: copy>() {} // 只允許值能夠被複製
fun name<T: copy + drop>() {} // 值可以被複製或是丟棄
fun name<T: key + store + drop + copy>() {} // 所有4項能力都能夠被表達**
```

也可以在結構體中的泛型參數使用

```rust
struct name<T: copy + drop> { value: T } **// T可以被複製和丟棄**
struct name<T: store> { value: T } **// T可以被儲存於全域中**
```

> 重要提醒: 記住+個符號，它可能沒有那麼直覺，Move是少數使用+在關鍵字列表的(Keyword List)
> 

例子: 使用限制符的結構體(struct)

```rust
module Storage {

    **// Box的內容可以被存儲於全域中**
    struct Box<T: store> has key, store {
        content: T
    }
}
```

> 重要觀念: 結構體的成員必須與結構體具備相同的 Abilities (除了 key 以外)
> 

舉例: 如果結構體具備 Copy 和 Drop 能力，那其成員也必須具備Copy 和 Drop，否則其成員不可以視為具備Copy和Drop能力

Move編譯器會讓我們在不遵守上述邏輯下通過，但是我們在運行的時候，如果需要使用此項能力會報錯喔

```rust
module Storage {
    // 不具備Copy和Drop能力的結構體
    struct Error {}

    // 創建一個具備Copy和Drop的結構體
    // 但其成員T並沒有寫下任何限制符
    struct Box<T> has copy, drop {
        contents: T
    }

    // 這個方法創建了包含無法複製和可丟棄內容的Box結構體
    public fun create_box(): Box<Error> {
        Box { contents: Error {} }
    }
}
```

上面這個例子會編譯成功，但如果我們要在Script中執行它

```rust
script {
    fun main() {
        {{sender}}::Storage::create_box() // 值被創建後，接著就被丟棄
    }   
}
```

我們就會得到一個報錯的結果，告訴我們Box結構體不具備Drop的能力

```rust
┌── scripts/main.move:5:9 ───
   │
 5 │   Storage::create_box();
   │   ^^^^^^^^^^^^^^^^^^^^^ Cannot ignore values without the 'drop' ability. The value must be used
   │
```

報錯的原因在於我們創建結構體時所使用的成員值不具有Drop的能力，也就是Contents不具有Box所要求的Abilities - Copy 與 Drop

> 重要提醒: 為了避免這樣的錯誤發生，我們應該盡可能地讓泛型參數的 Abilities 限制符與結構體本身的 Abilities 顯示地保持一樣
> 

例子: 在這次的例子中這樣的定義方式更加安全

```rust
// 這邊我們在父母結構體中加上限制符
// 這樣inner類型都要可以複製和丟棄了
struct Box<T: copy + drop> has copy, drop {
    contents: T
}
```

## **泛型中的多種類型 - Multiple types in generics**

我們也可以在泛型中使用多種類型，如使用單個類型的方式一樣，把多個類型放在<>之中，並用,逗號隔開

例子: 我們加入新的結構體類型Shelf，它可以容納兩種類型的Box結構體

```rust
module Storage {

    struct Box<T> {
        value: T
    }

    struct Shelf<T1, T2> {
        box_1: Box<T1>,
        box_2: Box<T2>
    }

    public fun create_shelf<Type1, Type2>(
        box_1: Box<Type1>,
        box_2: Box<Type2>
    ): Shelf<Type1, Type2> {
        Shelf {
            box_1,
            box_2
        }
    }
}
```

- Self的類型參數需與結構體中字段的定義類型順序相匹配
- 但泛型中的類型參數名稱就不需要相同，自行選擇合適的名稱就行了
- 每個類型的參數只有在定義範圍內有效，所以不需要將T1或T2和T1相匹配

例子: 執行多種泛型類型參數的方式與單個的相似

```rust
script {
    use {{sender}}::Storage;

    fun main() {
        let b1 = Storage::create_box<u64>(100);
        let b2 = Storage::create_box<u64>(200);

        // 我們可以使用任何類型，這邊使用相同類型一樣有效
        let _ = Storage::create_shelf<u64, u64>(b1, b2);
    }
}
```

在u64類型這個定義中最多可以有 18,446,744,073,709,551,615 (u64的大小)個泛型，基本上我們可能不會用到這個上限，所以盡情使用不用擔心限制

## 未被使用的類型參數 - Unused type params

並不是泛行中指定的每種類型參數都需要被使用到

```rust
module Storage {

    // 這兩種類型將會被用於標記
    struct Abroad {}
    struct Local {}

    // 更改後的Box將會具備目標屬性
    struct Box<T, Destination> {
        value: T
    }

    public fun create_box<T, Dest>(value: T): Box<T, Dest> {
        Box { value }
    }
}
```

可以在Script中被使用

```rust
script {
    use {{sender}}::Storage;

    fun main() {
        // 此值將會是 Storage::Box<bool> 類型
        let _ = Storage::create_box<bool, Storage::Abroad>(true);
        let _ = Storage::create_box<u64, Storage::Abroad>(1000);

        let _ = Storage::create_box<u128, Storage::Local>(1000);
        let _ = Storage::create_box<address, Storage::Local>(0x1);

        // 甚至可以使用u64類型當 Destination
        let _ = Storage::create_box<address, u64>(0x1);
    }
}
```

這邊我們使用泛型來標記類型，但實際上我們並沒有使用到它

後續當我們了解 Resource 的概念後，就會明白了它的重要性