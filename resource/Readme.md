# [Aptos學習筆記#8]Move進階使用 - Resource介紹一

## 目標

1. 可編程的 Resources - Programmable Resources
2. Resource 是什麼?
3. Key 和 Store Abilities - Key and Stor Abilities
4. Resource 概念 - Resource concept

## 可編程的 Resources - Programmable Resources

- Resource是Move中非常關鍵的功能，它讓Move變得獨特、安全和強大
- 這邊引述Diem網站上對Resource的觀點描述

> 1. Diem: Move的關鍵特性是能夠自訂義Resource類型，它能夠編碼具有豐富編程性的安全數字資產
> 

> 2. Diem: Resource在Move中就是一個普通的值，可以作為數據結構被儲存，或作為參數被傳遞給函數，也可以從函數中被傳回
> 
- Resource是一種特粗類型的結構體，它可以在Move程式中定義和創建新的資源，或使用現有的資源創建，因此，我們可以像使用任何其它數據 一樣管理數字資產 ( ex. Vector or Struct )

> 3. Diem: Move 類型系統為Resource提供特殊的安全保障
> 

> 4. Diem: Move中的Resource永遠不能被複製、重用或刪除丟棄，Resource類型僅能由定義該類型的Module創建或是銷毀
> 

> 5. Diem: 上述第4點的檢查是由Move的VM透過字節碼校驗機制 ( bytecode verification ) 強制執行，虛擬機將會拒絕運行任何未通過字節校驗機制的程式碼
> 

前面討論引用(Reference)和所有權(ownership)的筆記中，我們了解到了Move是如何保護範圍並控制變數的所有者範圍

前面討論泛型的筆記中，我們已經瞭解到一種特殊的種類匹配方式(kind-matching)可以用來區分可複製和不可複製的類型，所有這些討論到的特性都為Resource類型提供了安全性

> 6. 所有Diem的貨幣都是透過Diem::T類型實現的，像是LBR貨幣用Diem::T<LBR::T>來表示，和假設的美元貨幣用Diem::T<USD::T>表示
> 

> 7. Diem::T在Move語言中並沒有特殊地位，每個Move Resource都享有相同的保護
> 

參考資料: Move 白皮書([https://developers.diem.com/docs/technical-papers/move-paper/](https://developers.diem.com/docs/technical-papers/move-paper/))

## Resource 是什麼?

- Resource是一個在Move 白皮書中描述的概念
- 原本Resource是作為一種稱為resource的結構體實現的
- 但自從加入了Ability之後，Resource被實現成擁有Key和Store兩種Ability的結構體
- Resource本身就是一個非常適合用來儲存數字資產的類型，要實現Resource必須是不可複製且不可被丟棄的，且同時間，Resource必須可在帳戶之間儲存和轉移

**定義 - Definition**

Resource是一個只擁有 Key 和 Store 兩項Abilities的結構體0

```rust
module R {
    struct T has store, key {
        field: u8
    }
}
```

## Key 和 Store Abilities - Key and Stor Abilities

- Key Ability允許結構體 (Struct) 用來當成是存儲標識符
- Key 是一種儲存於頂層的能力 (Ability)  和 成為一個儲存 (storage)
- Store是儲存在密鑰 (key) 下的能力，Store能力允許儲存值

> **提醒: 雖然原生類型具備存儲(Store)能力，但他們沒有Key的能力，所以不能使用成頂層容器(top-level containers)**
> 

## Resource 概念 - Resource concept

原本Resource在Move中擁有自己的類型，但隨著能力(Abilities)的增加，它的概念變得更加抽象了，它可以被Key 和/或 Store 能力 (Abilities)實現出來，下面列出幾項限制

1. Resource儲存於帳戶之下，因此它只有在分配給帳戶時存在，並且只能透過此帳號訪問
2. 帳戶只能同時持有某一種類型的一個Resource，且該Resource必須具備Key的能力
3. Resource不能被複製或丟棄，但可以被儲存
4. Resource的值被須被使用，當Resource被創建或被其他帳戶取走時，它不能被丟棄，必須被儲存或解構