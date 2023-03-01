# [Aptos學習筆記#4]Move進階使用 - 類型的 Abilities - Types with Abilities - Copy & Drop 介紹

## 目錄

- **Abilities 是什麼**
- **四種類型分別說明 - Copy, Drop, Store and Key**
- **Abilities 語法**

## 一. Abilities 是什麼

- Move具有獨特的類型系統(type system)，非常彈性且可客製化
- Move的類型系統最多可以被四種限制符所修飾，也就是具備最多四種能力(abilities)
- 這四種能力定義了類型(type)如何被使用、刪除或是存儲

## 二. 四種類型分別說明 - Copy, Drop, Store and Key

- Copy: 值可以被複製，在RUST中當Struct或變數被取用的時候，它的所有權也會被拿走，但如果我們不想讓後面的函數或是變數拿走其所有權，就需要複製一份或借用，而Copy就是賦予複製一份的能力
- Drop: 在作用域範圍內結束時可以被丟棄，Move中嚴格規定當變數結束後需要被處理，所以我們需賦予其Drop的能力，否則會報錯
- Key: 可以作為全局存儲操作的鍵值，存儲在帳號之下，可以成為Resource
- Store: 值可以被儲存在全局存儲中，而這是可以存在帳本的資料，不僅只是模組內部運算方便而定義的

## 三. Abilities 語法

> 原生和內建類型的 abilities 是預先已經定義好的且不可以改變，像是integers, vector, address 和 boolean 類型的值先天具備有 copy、drop 和 store 的 ability
> 

結構體的 ability 添加的語法

```rust
struct <Struct Name> has ABILITY [, ABILITY] { [FIELDS] }
```

例子

```rust
module Vending {

    // 用逗號隔開多個abilities
    struct Beverage has copy, drop, store {
          year: u64
    }
    
    // 單一ability也是可行的
    struct Storage has key {
        beverage: vector<Beverage>
    }
    
    // 沒有ability的寫法
    struct Empty {}
}
```

不帶 Abilities 限制符的結構體 - Struct with no Abilities

- 如果不帶任何 abilities 可能會發生的狀況

Module

```rust
module Human {
    struct Student {
        student_id: u8,
        point: u64
    }

    public fun new_student(student_id: u8, point: u64): Student {
        Student {student_id, point}
    }
}
```

Script

```rust
script {
    use {{sender}}::Human;

    fun main() {
        Human::new_student(5, 9000);
    }
}
```

- 執行時產生的錯誤

```rust
error: 
   ┌── scripts/main.move:5:9 ───
   │
 5 │     Human::new_student(5, 9000);
   │     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Cannot ignore values without the 'drop' ability. The value must be used
   │
```

這邊使用Country::new_country()創建了一個值，但這個值並沒有傳遞到任何其他地方，所以他應該會在函數執行完後被丟棄

但是我們定義的Country類型並沒有加入 Drop 的 ability，所以執行時就報錯了

## Drop

- 我們為 Student 這個結構體加上 drop 的 Ability，這樣這個結構體的所有實例都將可以被丟棄了

Module

```rust
module Human {
    struct Student has drop { // 添加 drop ability 
        student_id: u8,
        point: u64
    }

    public fun new_student(student_id: u8, point: u64): Student {
        Student {student_id, point}
    }
}
```

Script

```rust
script {
    use {{sender}}::Human;

    fun main() {
        Human::new_student(5, 9000); // 可以順利 drop 掉
    }
}
```

可以執行了!

> 備註: Drop 的 ability 只能用在 drop 的行為，解構(Destructing)不需要 Drop
> 

## Copy

- 這邊我們想要複製一個結構體，在還未使用 Copy 關鍵字情況之下，會產生的問題

```rust
script {
    use {{sender}}::Human;

    fun main() {
        let student = Human::new_student(5, 9000);
        let _ = copy student;
    }
}
```

執行過程產生的問題

```rust
┌── scripts/main.move:6:17 ───
   │
 6 │         let _ = copy student;
   │                 ^^^^^^^^^^^^ Invalid 'copy' of owned value without the 'copy' ability
   │
```

結構中加上 copy 這個 ability

```rust
module Human {
    struct Student has drop, copy { // 添加 copy 和 drop ability 
        student_id: u8,
        point: u64
    }

    public fun new_student(student_id: u8, point: u64): Student {
        Student {student_id, point}
    }
}
```

可以執行了!

## 總結

- 原生的類型(Primitive Types)自帶 store、copy 和 drop 的能力
- 預設情況下結構(Struct)沒有任何 abilities
- Copy 和 Drop abilities 定義值可以個別被複製跟丟棄
- 一個結構體可以最多具備有4項 abilities