**Mutable Dictionary Keys in Swift: A Bug Waiting to Happen**

Let's imagine we have cartItems dictionary that has product as key and number of items as value
```swift
struct Product: Hashable {
    var id: Int
    var name: String
}

var cartItems = [Product: Int]()

var testProduct = Product(id: 1, name: "test")

cartItems[testProduct] = 2 //cartItems dictionary takes a separate copy of testProduct

testProduct.name = "test-updated"

cartItems[testProduct] // returns nil!

```
What happened:
- When we added the testProduct to the dictionary, the dictionary takes a separate copy of the **testProduct**
- Updating the original testProduct will not reflect in the separate copy stored in the dictionary
- When we needed to get the value of this product after its name property has been updated, we got nil
- This happened because the hash value of the testProduct after the update (id = 1 and name = "test-updated") will point to different bucket than the bucket of the old instance (id = 1 and name = "test"), and we will not be able to get the value 
- But if we were somehow saving the original product instance data (before the update), we will be able to retrieve the value

`cartItems[Product(id: 1, name: "test")] // this will return 2`

so even though we were able to retrieve the value, using mutable properties as hash keys should still be avoided because you might not have a separate copy of the instance data before the update and then the data will be lost in this case

**What if we used a class instead of struct, will it be different?**

Actually **yes**, the class case is even worse because the data will be lost and the dictionary data will be corrupted 
```swift 
class Product {
    var id: Int
    var name: String
    
    init(id: Int, name: String) {
        self.id = id
        self.name = name
    }
}

extension Product: Equatable {
    static func == (lhs: Product, rhs: Product) -> Bool {
        lhs.id == rhs.id && lhs.name == rhs.name
    }
}

extension Product: Hashable {
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
        hasher.combine(name)
    }
}

var cartItems = [Product: Int]()

var testProduct = Product(id: 1, name: "test")

cartItems[testProduct] = 2 // cartItems adds another reference to the testProduct object

testProduct.name = "test-updated"

cartItems[testProduct] // returns nil

cartItems[Product(id: 1, name: "test")] // returns nil
```
- since Product is a class, the cartItems dictionary will point to the same object, so any updates to the original object will be seen by the dictionary
 
But the cartItems dictionary will **not** recalculate the hash value and move the value to the correct bucket after the update
- so let's say the original data (id: 1, name: "test") produced a hash value that mapped to bucket 4
- `cartItems[testProduct]` (id: 1, name: "test-updated") will most probably produce different hash value and then different bucket so we can't retrieve the value, which is understood
- but here in `cartItems[Product(id: 1, name: "test")]` we know the dictionary rule that same key should produce same hash value so this should lead to bucket 4, so the value is not lost

actually **no** the value is lost and this line will return nil, because before retrieving the value from the bucket, another check is made using the == of Equatable protocol: **are the dictionary key and the key you are using the same?**

In our case, the current key stored in the dictionary for this value is: (id = 1 and name = "test-updated") while the key we are using is: (id = 1 and name = "test") so the check fails

so using mutable properties as hash keys should still be avoided
