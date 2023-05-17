# [Aptos學習筆記#9]Move進階使用 - Resource介紹二 - 發送人作為Signer - Sender as Signer

## 目錄

1. Signer 是什麼？
2. Scripts 中的 Signer
3. 標準庫中的 Signer 模塊 - Signer module in standard library
4. 模塊中的 Signer - Signer in module

## 一. Signer 是什麼?

- Signer與Signatures或字面的簽名並沒有直接關係，對Move的VM而言，Signer只是代表發送人(Sender)
- Signer是一種原生不可複製的類型(類似於Resource)
- 它用來保存交易發送者的地址
- Signer類型表示發送者的權限，換句話說就是使用Signer表示訪問發送者的地址跟資源

> Signer類型只具備一種能力 ( Ability ) - Drop
> 

## 二. Scripts 中的 Signer

- 由於Signer屬於原生類型，因此使用時必須創建它
- 它不像Vector一樣可以直接在程式碼中創建，但可以作為腳本參數被接收

```rust
script {
    // signer是一個已經擁有的值
    fun main(account: signer) {
        let _ = account;
    }
}
```

- Signer參數由VM(客戶端 CLI)自動載入到我們的腳本中，也就是說我們沒有辦法也不需要手動將它傳遞到腳本中
- Signer一直都會是一個Reference，就算標準庫(ex. 在Diem中是DiemAccount)可以訪問到Signer的實際值，但用這個值的函數是私有的 ( private )，並沒有辦法在其他地方使用或傳遞Signer的值

> 目前最常用來當Signer類型的變數名稱是Account
> 

## 三. 標準庫中的 Signer 模塊 - Signer module in standard library

原生類型需要原生函數，對於Signer類型的原生函數包含在0x1::Signer模塊

例子: 擷取Signer模塊中的兩個方法，一個是原生方法(native)，另一個是一般的Move方法，一般的Move方法使用起來更方便，因為它使用了取值運算符(*)來複製地址

Module

```rust
module Signer {
    // Borrows the address of the signer
    // Conceptually, you can think of the `signer`
    // as being a resource struct wrapper arround an address
    // ```
    // resource struct Signer { addr: address }
    // ```
    // `borrow_address` borrows this inner field
    native public fun borrow_address(s: &signer): &address;

    // Copies the address of the signer
    public fun address_of(s: &signer): address {
        *borrow_address(s)
    }
}
```

當我們要在Script中使用它的時候

Script

```rust
script {
    // 0x1::Signer::address_of(&account) 使用Signer模塊中的address_of函數來複製地址
    fun main(account: signer) {
        let _ : address = 0x1::Signer::address_of(&account);
    }
}
```

## 四. 模塊中的Sginer - Signer in module

例子: 我們透過在Module使用Singer模塊中的address_of函數，來當作腳本在調用時的代理取值函數

```rust
module M {
    use 0x1::Signer;

    // 代理Signer模塊中的address_of函數
    public fun get_address(account: signer): address {
        Signer::address_of(&account)
    }
}
```

> 使用&singer作為參數的類型方法，明確表示我們正在使用發送者的地址
> 

**重要觀念:**

使用signer類型的重要原因在於明確的表示哪些函數需要發送人(Sender)的權限，哪些則是不需要，因此，函數不能欺騙用戶在未經過授權的情況下訪問其Resource