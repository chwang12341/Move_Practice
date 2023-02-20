# [Aptos學習筆記#2]Move基本使用 - 函數規則

## Move中的函數

函數

- 函數以fun當關鍵字來開頭 ex. fun my_function_name(arg1: u8, arg2: u64, arg3: bool): u64 {}

## 腳本中的函數(Function in Script)

- 腳本(Script)中只能有一個被視為main的函數
    - main作為交易被執行，是受到很多限制的，可以有參數，但是沒有返回值
    - main應該被用來操作發布模塊中的其他功能
- 例子: 用以檢查地址是否存在的腳本

```rust
script {
    use 0x1::Account;

    fun main(addr: address) {
        assert(Account::exists(addr), 1);
    }
}
```

## 模塊中的函數(Function in Module)

- 模塊是一組發佈的函數與結構體，可以處理一項或多項任務
- 函數的所有潛能只有在模塊中能夠展現，相較之下在腳本中使用函數功能是非常限制的
- 例子: 在模塊中寫一個會回傳1的函數

```rust
module Math {
    fun one(): u8 {
        1
    }
}
```

- 說明: 定義一個Math模塊，裡面有一個函數one，函數會回傳u8類型的1，由於1是返回值，所以後面沒有加分號(;)

## 函數的參數

參數的規則

- 參數需要指定類型，且使用逗號隔開每個參數
- 函數的返回值類型在括號與冒號後面 ex. fun one(): <返回值類型> {}
- 例子: 將兩數相乘的函數寫進模塊中，並在腳本中使用該函數

模塊(Module): 撰寫兩數相乘函數

```rust
module Math {
    
    public fun multiply(a: u64, b: u64): u64 {
        a * b
    }

    fun one(): u8 {
        1
    }
}
```

腳本(Script): 導入兩數相乘函數到main函數中使用

```rust
script {
    use 0x1::Math; //這邊使用 0x1，但可以是你指定的地址
    use 0x1::Debug;

    fun main(x: u64, y: u64): u64 {
    
    let result = Math::multiply(x, y);

    Debug::print<u64>(&result);
    }

}
```

## Return 返回值

- return使函數結束並傳回結果
- 例子: 根據不同條件狀況，返回不同結果

```rust
module Condition {
    
    public fun condition_return(x: u8): bool {
        if (x > 5) {
            return true
        };

        if (x == 5) {
            true
        } else {
            false
        }
    }
}
```

## 多個返回值

- 使用()來返回多個值
- 例子: 撰寫一個比較兩數誰大的函數，並返回哪個數大與是否相等的布林值

Module: 撰寫兩數比較的函數

```rust
module Math {

    public fun compare(x: u8, y: u8): (u8, bool) {
        if (x > y) {
            (x, false)
        } else if (x < y) {
            (b, false)
        } else {
            (a, true)
        }
    }
}
```

Script: 使用兩數比較的函數

```rust
script {
    use 0x1::Debug;
    use 0x1::Math;

    fun main(x: u8, y: u8) {
        let (max, is_equal) = Math::compare(35, 100);
        
        assert(is_equal, 1) // assert(condition, code)

        Debug::print<u8>(&max)
    }
}
```

補充 assert用法: assert(condition, code)，就是if前面那個condition條件成立，拋出後面那個code ex. assert(condition ≤ 10, 0) 參考連結: [https://www.theblockbeats.info/news/31991](https://www.theblockbeats.info/news/31991)

## 函數可見性

- 我們在撰寫模塊(Module)時，希望有些函數是可以被其他人使用的，有些則希望是隱藏起來的，這時候函數可見性的修飾符號就發揮作用了
- 預設的情況下，模塊(Module)中的函數都是被定義為私有(private)的，無法在腳本(Script)中使用
- 使用public關鍵字來定義可以被腳本使用的公開函數
- 例子: 在Module中定義一個public函數，並從Script中使用它

Module

```rust
module Math {
    
    public fun multiply(a: u64, b: u64): u64 {
        a * b
    }

    fun one(): u8 {
        1
    }
}
```

Script

```rust
script {
    use 0x1::Math;

    fun main() {
        Math::multiply(10, 50);
    }
}
```

備註: multiply有被定義成public函數，所以可以被使用，而one函數沒有，所以不能被使用

## 如何訪問私有函數

- 私有函數如果都沒辦法被使用的話，那就沒有任何意義了
- 可以在public函數中去調用private函數來執行一些任務

Module

```rust
module Math {
    
    public fun is_one(x: u8): bool{
        x == one()
    }

    fun one(): u8 {
        1
    }
}
```

這樣就可以達到使用私有函數，但又不需要開放私有函數出去給其他人使用了

## Native 函數

- 為一種稱為Native的特殊函數，提供Move原本提供額外的功能
- Native函數由VM本身定義的，且在不同的實作中可能有所不同
- Native函數並沒有用Move語法實現，沒有函數體，直接用分好來結尾
- 用native關鍵字來標記Native函數，並且與函數可見性的修飾符不衝突，可以同時使用
- 例子: 定義一個Native函數，並同時為public函數(以下是Diem標準庫的例子)

```rust
module Signer {
    native public fun borrow_address(s: &signer): &address;
}
```