# [Aptos學習筆記#7]Move進階使用 -  使用Vector管理集合 - Managing Collections with Vector

## 目錄

1. **為什麼需要Vector?**
2. **Vector是什麼?**
3. **內聯 Vector 定義的十六進制數組與字符 - Hex and Bytestring literal for inline vector definitions**
4. **Vector 速查表 - Vector Cheatsheet**

## 一. 為什麼需要Vector?

前面的筆記中我們已經很熟悉結構體的類型了，它讓我們可以創建自己的類型並儲存複雜的數據，但**有時候我們需要更動態、可擴展性核可管理性的功能，這邊Move提供了向量Vector的概念來處理**

## 二. Vector是什麼?

- 用於儲存數據集合的內置類型( build-in type)
- 集合數據能夠是任何類型(但僅一種)
- Vector的功能實際上是由Move VM提供的
- 使用Vector唯一的辦法就是使用Move標準庫和native函數

```rust
script {
    use 0x1::Vector;

    fun main() {
        // 使用泛型創建一個空的 Vector
        let a = Vector::empty<u8>();
        let i = 0;

        // 將值塞入剛剛創建的Vector中
        while (i < 10) {
            Vector::push_back(&mut a, i);
            i = i + 1;
        };

        // 現在印出Vector的長度
        let a_len = Vector::length(&a);
        0x1::Debug::print<u64>(&a_len);

        // 移除掉兩個元素
        Vector::pop_back(&mut a);
        Vector::pop_back(&mut a);

        // 再次印出Vector長度
        let a_len = Vector::length(&a);
        0x1::Debug::print<u64>(&a_len);
    }
}
```

Vector最多可以儲存U64個單個非引用類型(single non-reference type)的值

例子: 為了理解Vector如何幫助管理巨大的存儲量，這邊撰寫一個模塊 ( Module )

```rust
module Shelf {

    use 0x1::Vector;

    struct Box<T> {
        value: T
    }

    struct Shelf<T> {
        boxes: vector<Box<T>>
    }

    public fun create_box<T>(value: T): Box<T> {
        Box { value }
    }

    // 對於不可複製的內容本方法不可行
    public fun value<T: copy>(box: &Box<T>): T {
        *&box.value
    }

    public fun create<T>(): Shelf<T> {
        Shelf {
            boxes: Vector::empty<Box<T>>()
        }
    }

    // 將Box的值移入Vector
    public fun put<T>(shelf: &mut Shelf<T>, box: Box<T>) {
        Vector::push_back<Box<T>>(&mut shelf.boxes, box);
    }
    
    // 將Vector中的元素移出
    public fun remove<T>(shelf: &mut Shelf<T>): Box<T> {
        Vector::pop_back<Box<T>>(&mut shelf.boxes)
    }

    // 計算Vector的長度
    public fun size<T>(shelf: &Shelf<T>): u64 {
        Vector::length<Box<T>>(&shelf.boxes)
    }
}
```

在Script中，我們首先創建一個 Shelf，並為其提供幾個 Box，並觀察如何在Module中使用 Vector

```rust
script {
    use {{sender}}::Shelf;

    fun main() {

        // 創建一個shelf和兩個u64類型的Box
        let shelf = Shelf::create<u64>();
        let box_1 = Shelf::create_box<u64>(99);
        let box_2 = Shelf::create_box<u64>(999);

        // 將剛剛創建的兩個Box推進Vector
        Shelf::put(&mut shelf, box_1);
        Shelf::put(&mut shelf, box_2);

        // 印出長度 - 結果為2
        0x1::Debug::print<u64>(&Shelf::size<u64>(&shelf));

        // 拿走最後一個推進Vector的Box
        let take_back = Shelf::remove(&mut shelf);
        let value     = Shelf::value<u64>(&take_back);

        // 驗證我們拿出來的Box值是999
        assert(value == 999, 1);

        // 再次印出Vector長度 - 結果為1
        0x1::Debug::print<u64>(&Shelf::size<u64>(&shelf));
    }
}
```

Vector非常強大，它允許我們存儲大量數據(最大長度為 18,446,744,073,709,551,615)，並讓我們在索引的存儲中使用它

## 三. 內聯 Vector 定義的十六進制數組與字符 - Hex and Bytestring literal for inline vector definitions

Vector也代表著表示字符串

VM 支援將vector<u8>當作參數傳遞給腳本中的主函數的方式

例子: 我們也可以在使用十六進制文本在Script或Module中定義一個vector<u8>

```rust
script {

    use 0x1::Vector;

    // 這是在main中傳入參數的方式
    fun main(name: vector<u8>) {
        let _ = name;

        // 這是我們使用文字的方式
        // 這邊表示的是一個hello world的字符串
        let str = x"68656c6c6f20776f726c64";

        // 十六進制的文字也傳給我們vector<u8>
        Vector::length<u8>(&str);
    }
}
```

有個簡單的方式就是使用字節串文字(bytestring literals)

```rust
script {

    fun main() {
        // 使用b"<我們想輸入的字符串>"的方式更簡易
        let _ = b"hello world";
    }
}
```

它們被當成是ASCII 字符串，也被表示為vector<u8>

## 四. Vector 速查表 - Vector Cheatsheet

在標準庫中的簡短Vector方法速查表

1. 創建一個類型為<E>的空Vector

```rust
Vector::empty<E>(): vector<E>;
```

1. 取得Vector長度

```rust
Vector::length<E>(v: &vector<E>): u64;
```

1. 將元素推向Vector的末端

```rust
Vector::push_back<E>(v: &mut vector<E>, e: E);
```

1. 獲取對Vector的可變引用，不可變引用可以使用Vector::borrow()

```rust
Vector::borrow_mut<E>(v: &mut vector<E>, i: u64): &E;
```

1. 從Vector末端取走一個元素

```rust
Vector::pop_back<E>(v: &mut vector<E>): E;
```

**標準庫中的Vector模塊:** 

[move/Vector.move at main · diem/move](https://github.com/diem/move/blob/main/language/move-stdlib/sources/Vector.move)