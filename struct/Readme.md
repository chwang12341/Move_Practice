# [Aptos學習筆記#3]Move進階使用 - 結構Struct用法

## 目錄

- 結構體
- 如何定義結構體
- 遞歸結構體 Recursive Definition
- 訪問結構體字段
- 回收結構體 Destructing Structures
- 為結構體實現Getter函數 Implementing getter functions for struct fields

## 一. 結構體

- 為一個自定義類型，可以具有複雜數據，也可以不具有任何數據
- 由鍵值組成，可以想成使用類似於Python的Dict由”key-value”儲存
- key為屬性的名稱，value則為儲存的內容
- 使用struct關鍵字定義
- 結構體最多可以擁有4個能力
- 為Move創建自定義類型的唯一方法

## 二. 定義

- 結構體只能在模塊中進行定義，且以struct當關鍵字開頭
- 一個結構體最多可以擁有65535的字段
- 可以使用新定義出的類型，當另一個結構體的字段類型

```rust
struct Name {
    FIELD1: TYPE1,
    FILED2: TYPE2,
    FILED3: TYPE3,
    ...
}
```

- 例子

```rust
module M {
    
    struct Empty {}

    struct NewStruct {
        field1: address,
        field2: Empty,
        field3: u64
    }

    struct Human {
        field1: u8,
        field2: bool,
        field3: address,
        field4: bool,
        field5: u64,
				
        field6: NewStruct  // 可以使用新定義出的類型，當另一個結構體的字段類型
    }

}
```

- 透過定義這個結構體的模塊來訪問新自定義的類型

```rust
M::NewStruct;
// or
M::Human
```

## 三. 遞歸結構體 Recursive definition

- Move可以使用其他結構體當作成員，但不能遞歸使用自己本身的結構體
- 例子: 在相同結構體中遞歸使用本身的結構體當類型，是不被允許的

```rust
module M {
    struct SameStruct {
    
        field1: SameStruct
    }
}
```

創建結構體的

- 使用結構體的定義來創建該實例，只是傳入的是值不是類型
- 例子: 創建一個student的實例

```rust
module Human {
    
    struct Student {
        student_id: u8,
        age: u64
    }

    public fun new_student(s_id: u8, s_age: u64): Student {
        let student = Student {
            student_id: s_id,
            age: s_age
        };

        student    
    }
}
```

- 簡化創建的程式碼 - 透過傳遞與結構體的字段名稱香匹配的變量名稱
- 例子: new_student()使用了簡化的方法

```rust
module Human {
    
    struct Student {
        student_id: u8,
        age: u64
    }

    public fun new_student(student_id: u8, age: u64): Student {
        Student {
            student_id,
            age
        }
    }

    // 另一種寫法: Student { student_id, age }
}
```

- 創建一個空的結構體(沒有字段)，僅需使用花括號

```rust
module Example {
    struct Empty {}

    public fun empty(): Empty {
        Empty {}
    }
}
```

## 四. 訪問結構體字段

- 只有模塊(Module)可以訪問自己的字段(fields)，在模塊(Module)外部字段(fields)是屬於private，沒辦法被訪問
- 結構的字段只有在模塊內可見，在此模塊外(腳本或是其他模塊中)，它只是一種類型
- 可以透過.符號來訪問結構的字段(fields) ex. <struct>.<property>

```rust
public fun get_student_age(student: Student): u64 {
    student.age
}
```

- 若在同一個模塊中定義嵌套結構類型(Nested Struct Type)，可以使用類似的方法進行訪問<>

```rust
<struct>.<field>
<struct>.<field>.<nested_struct_field>
```

## 五. 回收結構體 Destructing Structures

- 使用語法 let <STRUCT DEF> = <STRUCT> 銷毀或解構一個結構

```rust
module Human {
    
    struct Student {
        student_id: u8,
        age: u64
    }

    public fun destroy(student: Student) {
        
        // 變數必須匹配結構
        // 必須指定所有結構
        let Student { student_id, age } = student;    
    
        // 當銷毀student被丟棄後
        // 它的字段現在是變數，可以被使用
        (student_id, age)
    }
}
```

- 在Move中禁止使用未使用到的變數，有時候可能需要在不使用其字段的情形下銷毀結構，對於未使用到的結構字段使用_表示

```rust
module Human {
    
    struct Student {
        student_id: u8,
        age: u64
    }

    public fun destroy(student: Student) {
        
        // 這樣銷毀結構，就不會產生未使用到的變數 
        let Student {student_id: _, age: _ } = student;

        // 或是只留student_id，然後不要產生age變數
        // let Student {student_id, age: _ } = student;    
    }
}
```

- 銷毀結構體現在看起來並沒有那麼重要，但在Resource中會扮演相當重要的角色

## 六. 為結構體實現 getter 函數 - Implementing getter-functions for struct fields

- 目的是讓結構體在外部(Module 以外)可以被讀取，所以需要使用一些方法
- 這些方法會讀取struct中的字段(fields)並將它們當作回傳值傳遞
- getter 方法的調用方式與結構體訪問字段的方法相同，但如果我們的模塊(Module)定義了多個結構體，那getter方法就可能會帶來不便
- 例子: 我們創建一個Student結構體，然後撰寫兩個getter方法來獲取裡面的字段
    
    ```rust
    module Human {
    
        struct Student {
            student_id: u8,
            age: u64
        }
    
        public fun new_student(student_id: u8, age: 64): Student {
            Student {student_id, age}
        }
    
        // 讓外部訪問student_id的方法
        public fun student_id(student: &Student): u8 {
            student.student_id
        }
    
        // 讓外部訪問age的方法
        public fun age(student: &Student): u64 {
            student.age
        }
    
        // ... fun destroy ...
    }
    ```
    
- 透過getter的方法，在Script中訪問模塊(Module)中的結構體字段(fields)

```rust
script {
    use {{sender}}::Human as H;
    use 0x1::Debug;

    fun main() {
        // 使用Module中創建Student結構的方法
        let student = H::new_student(1, 26);

        // 印出Student ID
        Debug::print<u8>(&H::student_id(&student)) 

        // 印出Age
        Debug::print<u8>(&H::age(&student))
    
        // 無法使用下邊這種直接訪問結構字段的寫法
        // let student_id = student.student_id;
        // let age = student.age;
    }
}
```

## Reference

[The Move Language - The Move Book](https://move-book.com/index.html)