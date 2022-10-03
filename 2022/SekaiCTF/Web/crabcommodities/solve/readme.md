# Crabcommomdities

Lại một bài được viết bằng rust của tác giả Strellic, lúc làm mình chỉ hi vọng là bug logic hoặc bug nào đấy không liên quan nhiều về gốc rễ của rust (tại mình bị thọt ngôn ngữ này 🥺) và cuối cùng nó là integer overflow 😘.

Sau khi register account thì sẽ được dẫn đến `/game`

(over_view.png)

Về mặt tính năng, khởi điểm số tiền ta được cấp là 37000$ và có thể mua các sản phẩm ở góc dưới bên phải, mượn nợ (`Loan`), upgrade `Storage` ... và quan trọng là cần tới tận `2000000000$` để lấy được flag.

Check http request + kết hợp với source, ta chỉ gửi tên của item và quantity, giá của item được lưu ở phía server

(burp_req_1.png)

Vuln bài này xuất hiện ở hai chỗ và ta sẽ cần khai thác theo đúng thứ tự thì mới đủ tiền để mua flag, đầu tiên là `/upgrade`:

(vuln_1.png)

Theo những gì mình hiểu, biến `price` được khởi tạo bằng `let mut price = item.price;` sẽ mang kiểu i32 (<https://stackoverflow.com/questions/55903243/what-is-the-default-integer-type-in-rust>) và nhìn xuống dưới `price *= body.quantity;` => có khả năng bị integer overflow

Check `price` của item `Storage Upgrade` là `100000` và max `quantity` ta có thể gửi là `32767` đồng thời MAX của i32 là `2147483647` (<https://doc.rust-lang.org/std/primitive.i32.html>)

```bash
PS C:\Users\ASUS> python
Python 3.9.0 (tags/v3.9.0:9cf6752, Oct  5 2020, 15:34:40) [MSC v.1927 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> 32767*100000 - 2147483647
1129216353
>>>
```

=> có thể khai thác integer overflow khiến cho price bị set thành giá trị âm và cuối cùng là `user.game.money.set(user.game.money.get() - price as i64);` giúp cộng thêm tiền

Khi làm mình không gửi quantity là `32767` lên server vì sau khi gửi và muốn uprade thêm các item khác sẽ bị lỗi  `Too many upgrades purchased`:

(test_2.png)

-> tăng tiền lần 1 thành công

Overflow lần thứ hai là ở `/buy`, `let price = item.price * body.quantity;` và `user.game.money.set(user.game.money.get() - price as i64);` nhưng trước đó ta sẽ cần upgrade `More Commodities` để có thể mua được nhiều item hơn (hay để số to -> dễ overflow)

Upgrade `More Commodities`:

(more_items.png)

Sau một hồi mua rồi lại bán bán rồi mua 😅 thì money của mình cũng dư để mua flag 

(burp_req_2.png)

(temp2.png)

và flag:

(flag.png)