# BaseJson4

![CocoaPods](https://img.shields.io/cocoapods/v/BaseJson4.svg) ![Platform](https://img.shields.io/badge/platforms-iOS%209.0+%20%7C%20macOS%2010.10+-3366AA.svg)

BaseJson4 讓你輕鬆的把 Json字串 轉換到 物件模型, 同樣也能把模型轉出 Json字串<br>
*此套件只適用 Swift 4.0 以上

## 為什麼要用這個?

當我們呼叫一個 Service API, 後端伺服器經常是回應一個 JSON字串, 過去 Swift 3 只能轉換成字典(Dictionary)或陣列(Array), 直到現在 Swift 4 才能直接轉換成一個自訂的物件模型, 這個套件讓你輕鬆使用 Swift 4 的這項特徵.

原本舊的作法可能看起來像這樣:

```swift
if let statusesArray = try? JSONSerialization.jsonObject(with: data, options: .allowFragments) as? [[String: Any]],
    let user = statusesArray[0]["user"] as? [String: Any],
    let username = user["name"] as? String {
    // Finally we got the username
}
```

如果你有使用 [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) 看起來會像這樣:

```swift
let json = JSON(data: dataFromNetworking)
if let userName = json[0]["user"]["name"].string {
  //Now you got your value
}
```

但仍舊有一個缺點, 欄位名稱必須手動輸入, 例如上面範例的 ["user"] , 手打名稱可能有機會輸入錯字, 或一時忘記完整名字.

那麼, 使用 BaseJson4 後看起來會像這樣:

```swift
  if let user = jsonStr.toObj(type: User.self) {
    // 直接使用 User 物件, User 可以是一個 Class 或 Struct
    print("user.name=\(user.name)")
    let age = user.age
  }
```

## 系統需求 Requirements

- iOS 9.0+ | macOS 10.10+
- Xcode 9

## 如何安裝 Manually

1. 下載本套件的 BaseJson4.swift 檔
2. 把這個檔案加進你的 xcode 專案裡
3. 安裝完成了

## 如何使用 Usage

### ======================================================
### 1. Json to Obj
我們先用一段 json字串 來示範
```json
{"id":66, "birthday":"1997-05-08", "height": 180.61, "name":"小軒", "gender":"M", "age": 29, "friends": [ {"name":"小明", "isFriend": true}, {"name":"小華", "isFriend": false, "test":1} ]}
```

首先先建立物件模型(Model), 這裡我們使用 class, 當然你也可以使用 struct
```swift
class User: BaseJson4 {
    var name: String? = nil
    var gender: String? = nil
    var age: Int = 0
    var birthday: String? = nil
    var friends: [Friend]? = nil
    var height: Double = 0    
}

class Friend: BaseJson4 {
    var name: String? = nil
    var isFriend: Bool = false
}

```

json資料來源類型可以是 String 或 Data<br>
BaseJson4 幫 String/Data 加上了一個擴展功能叫做 toObj<br>
例如:
```swift
let jsonStr = "{\"id\":66, \"birthday\":\"1997-05-08\", \"height\": 180.61, \"name\":\"小軒\", \"gender\":\"M\", \"age\": 29, \"friends\": [ {\"name\":\"小明\", \"isFriend\": true}, {\"name\":\"小華\", \"isFriend\": false, \"test\":1} ]}"

if let user = jsonStr.toObj(type: User.self) {
    let desc = user.description()   // description()可以傾印出此物件的所有屬性值
    print("物件內容 ==> \(desc)")
    // .... 已經可以直接使用物件了
    let age = user.age
    let name = user.name
    if let friends = user.friends {
        for friend in friends {
          print("friend name=\(friend.name)")
        }
    }
}
```
我們已經成功的把一段 Json字串 轉成了一個 物件Model, 轉換只需要一行程式碼 String.toObj<br>

輸出的 log 如下:
```text
物件內容 ==> 
<User:
 name=Optional("小軒")
 gender=Optional("M")
 age=29
 birthday=Optional("1997-05-08")
 friends=Array count=2 [ <Friend: name=Optional("小明") isFriend=true>, <Friend: name=Optional("小華") isFriend=false> ]
 height=180.61
>

friend name=Optional("小明")
friend name=Optional("小華")
```

是不是很輕鬆容易呢<br>
json字串中有一欄叫做 birthday, 它是一個生日日期, 我們可能會希望它能直接轉成一個 Date 物件<br>
接下來我們示範如何使用日期格式<br>
首先把我們的 User 物件的 birthday 改成 Date 類型
```swift
class User: BaseJson4 {
    var name: String? = nil
    var gender: String? = nil
    var age: Int = 0
    var birthday: Date? = nil       // <--- 改成 Date類型
    var friends: [Friend]? = nil
    var height: Double = 0    
    
    static func dateFormats() -> [String: String]? {
        return [CodingKeys.birthday.stringValue: "yyyy-MM-dd"]  // <--- 指定格式為yyyy-MM-dd
    }
}
```
這樣一來 birthday 就變成了一個 Date 物件了<br>
修改後輸出的 log 如下:
```text
物件內容 ==> 
<User:
 name=Optional("小軒")
 gender=Optional("M")
 age=29
 birthday=Optional(1997-05-07 16:00:00 +0000)
 friends=Array count=2 [ <Friend: name=Optional("小明") isFriend=true>, <Friend: name=Optional("小華") isFriend=false> ]
 height=180.61
>
```
### ======================================================
### 2. Obj to Json

再來我們要如何把一個物件輸出成 json字串呢<br>
很容易, 同樣只要一行程式碼:
```swift
let jsonStr = user.toJson()
print("輸出的 json 字串 = \(jsonStr)")
```
print結果如下:
```text
輸出的 json 字串 = {"age":29,"gender":"M","friends":[{"name":"小明","isFriend":true},{"name":"小華","isFriend":false}],"name":"小軒","birthday":"1997-05-08","height":180.61000000000001}
```
預設是適合傳給api的緊縮格式, 如果你希望產生適合人類閱讀的json可以加上參數
```swift
let jsonStr = user.toJson(.prettyPrinted)  // <--- 加上 prettyPrinted 參數
```
加上參數後的 print結果變成這樣:
```text
輸出的 json 字串 = {
  "age" : 29,
  "gender" : "M",
  "friends" : [
    {
      "name" : "小明",
      "isFriend" : true
    },
    {
      "name" : "小華",
      "isFriend" : false
    }
  ],
  "name" : "小軒",
  "birthday" : "1997-05-08",
  "height" : 180.61000000000001
}
```

以上歡迎幫忙翻譯成英文版 ^_^








































