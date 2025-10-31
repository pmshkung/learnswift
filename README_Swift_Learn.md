# Learn Swift

## lazy 關鍵字
```swift
在 Swift 中，lazy 是一個延遲初始化(lazy initialization)的關鍵字。它用在屬性(property)宣告前，
表示該屬性的值不會在物件建立時立刻初始化，而是第一次被存取時才會執行初始化。適合用在「初始化成本高、
但不一定會用到」的屬性。

1. 延遲初始化
   避免不必要的資源消耗（例如需要大量記憶體、計算或網路存取的屬性）。
2. 必須是變數（var），不能是常數（let）
   因為 let 代表不可改變，而 lazy 屬性會在稍後才設定初值。
3. 支援閉包（closure）初始化
   常用來做複雜的屬性設定

- 成本高但不一定會用到的屬性
class DataLoader {
    lazy var bigData: [String] = {
        print("Loading big data...")
        return Array(repeating: "Data", count: 1_000_000)
    }()
}

let loader = DataLoader()
// 尚未存取 bigData → 不會初始化
print("Ready")
// 第一次用到時才初始化
print(loader.bigData.count)

- 需要依賴 self 的屬性
因為 self 只有在初始化完成後才能使用，lazy 可以延遲到物件完全建立後再處理。
class Person {
    var name: String
    lazy var greeting: String = "Hello, my name is \(self.name)"

    init(name: String) {
        self.name = name
    }
}

let p = Person(name: "Webb")
print(p.greeting) // "Hello, my name is Webb"

- UI 元件建立（例如 iOS UIKit / SwiftUI 中）
UI 元件通常需要額外設定，lazy 可避免在初始化時就馬上建構，等需要時才生成。
class MyViewController: UIViewController {
    lazy var button: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("Click Me", for: .normal)
        btn.backgroundColor = .blue
        return btn
    }()
}

- 計算結果，且只需要計算一次
某些計算量大但結果固定的屬性，可以用 lazy 儲存，避免每次呼叫都重算。
class MathHelper {
    lazy var primes: [Int] = {
        print("Calculating primes...")
        return (2...10_000).filter { n in
            (2..<n).allSatisfy { n % $0 != 0 }
        }
    }()
}

let math = MathHelper()
// 尚未使用 → 不會計算
print("Before accessing primes")
// 第一次使用 → 開始計算
print(math.primes.count)

- 搭配閉包初始化需要多行設定的屬性
用 lazy 可以把屬性初始化邏輯包在閉包裡，程式碼更清晰。
class Config {
    lazy var settings: [String: Any] = {
        var dict = [String: Any]()
        dict["theme"] = "dark"
        dict["fontSize"] = 14
        dict["language"] = "zh-TW"
        return dict
    }()
}
```
## @objc的意思

是一個Swift 與 Objectvie-C 的橋接屬性，它的作用是：  
明確指定這個 Swift 類別在 Objective-C 以及 Core Data runtime（底層仍是 Objective-C）中所對應的名稱。  

當 Swift 類別要繼承 NSManagedObject（也就是 Core Data 的物件類型）時，系統必須能夠：
- 在執行階段用字串名稱找到這個類別
- 正確對應到 .xcdatamodeld 檔案中定義的「Entity Name」  

Core Data 模型（.xcdatamodeld）中的每個 Entity 都有一個名稱，例如：  
```yaml
Entity Name: HistoryEntity
Class: HistoryEntity
Module: (YourModuleName)
```
在程式碼中，我們會有：
```swift
@objc(HistoryEntity)
public class HistoryEntity: NSManagedObject { }
```
這樣 Core Data 在執行階段就能透過字串 "HistoryEntity" 找到對應的 Swift 類別。  

## @NSManaged

一個屬性修飾詞，告訴 Swift 編譯器「此屬性由 Core Data 在執行時動態管理」。  

## @nonobjc

作用為用來告訴編譯器：這個Swift 成員(方法、屬性或初使化器)，不要暴露給 objective-C runtime  
繼承NSManagedObject或NSObject，會自動將成員暴露給objective-C runtime(會自動加上@objc)。  
```swift
extension HistoryEntity {
    @nonobjc public class func fetchRequest() -> NSFetchRequest<HistoryEntity> {
        return NSFetchRequest<HistoryEntity>(entityName: "HistoryEntity")
    }

    @NSManaged public var id: String?
}

```
這裡的@nonobjc 代表：不讓fetchRequest 方法暴露給Objective-C runtime
- Core Data自已不需要透過Objective-C selector 呼叫這個方法
- 這個方法純屬 Swift 的便利函式
- 若暴露出去反而會污染 Objective-C 的命名空間

## DispatchQueue 中的.barrier 作用
./barrier 是CGD(Grand Central Dipatch)中一個重要標記，用於控制並行佇列中的執行順序。  
當你在並行佇列(concurrent queue)中使用.barrier標記時，它會建立一個屏障：  
執行順序：  
- 等待：等待所有在barrier之前送出的工作完成
- 獨佔執行：barrier工作會獨佔整個佇列執行(此時不會有其他工作同時執行)
- 繼續：barrier工作完成後，後續的工作才能繼續執行

## DispatchQueue.global(qos: .userInitiated)的意義
qos = Quality of Service  
告訴系統這個任務的重要程度與即時性需求，讓系統決定該任務應該在哪個優先層級(Priority)執行。  
常見的QoS等級  
- .userInteractive:最高優先級，需要立即回應使用，如：UI更新
- .userInitiated:使用者觸發、需快速完成但非立即，如：讀檔
- .default:一般優先級，如：一般背景任務
- .utility:可長時間執行的任務，如：下載、匯出檔案
- .background:最低優先級，不會影響使用者體驗的任務，如：備份
- .unspecified:未指定，系統自行推斷

