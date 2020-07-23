

```swift
struct MyColor: Hashable {
    let red: UInt8
    let green: UInt8
    let blue: UInt8
    
    // Synthesized by compiler
    func hash(into hasher: inout Hasher) {
        hasher.combine(self.red)
        hasher.combine(self.green)
        hasher.combine(self.blue)
    }
    
    // Default implementation from protocol extension
    var hashValue: Int {
        var hasher = Hasher()
        self.hash(into: &hasher)
        return hasher.finalize()
    }
}

let cyan = MyColor(red: 0x00, green: 0xFF, blue: 0xFF)

let yellow = MyColor(red: 0xFF, green: 0xFF, blue: 0x00)

print(cyan.hashValue == yellow.hashValue);
```

