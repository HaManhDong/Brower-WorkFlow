###**1. Introduce**

####**Các trình duyệt sẽ được đề cập đến**
Hiện nay, có 5 trình duyệt được sử dụng chính trên desktop: Chrome, Firefox, Internet Explorer, Opera và Safari. Trên điện thoại thì có Android Browser, iPhone, Opera Mini và Opera Mobile, UC Browser, trình duyệt của Nokia S40/S60 và Chrome, tất cả ngoại trừ Opera browsers đều dựa trên Webkit. Tôi sẽ đưa ra các ví dụ về các trình duyệt mã nguồn mở, đó là Firefox and Chrome, và Safari.

####**Chức năng chính của trình duyệt**
Chức năng chính của trình duyệt là lấy tài nguyên của trang web mà bạn muốn truy cập và hiển thị nội dung trang web đó lên trên cửa sổ của trình duyệt. Tài nguyên của 1 trang web thường là 1 tài liệu HTML, nhưng đôi khi cũng có thể dưới dạng PDF, ảnh ... Vị trí của tài nguyên được xác định bởi người dùng sử dụng 1 URI (Uniform Resource Identifier).

Cách để 1 trình duyệt thông dịch và hiển thị tài liệu HTML tuân theo những quy định về HTML và CSS. Những quy định này được thiết lập bởi tổ chức W3C (World Wide Web Consortium) - là tổ chức lập ra các chuẩn cho web. 

Các giao diện người dùng của các trình duyệt có một số điểm khá giống nhau, đó là:

+ Có thanh nhập địa chỉ URL
+ Có 2 button là Back và Forward trên thanh công cụ
+ Có button tải lại trang 
+ Có button home để trở về giao diện chính của trình duyệt
+ Có thanh bookmark 

Điểm kỳ là là giao diện người dùng của các trình duyệt không phải tuân theo bất cứ quy tắc nào, mà nó chỉ được hình thành từ các trải nghiệm của người dùng qua nhiều năm và qua sự bắt chước lẫn nhau. 

####**Cấu trúc cấp cao của các trình duyệt**
Dưới đây là các thành phần chính của 1 trình duyệt:

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/layers.png)

**Hình 1:** cấu trúc của 1 trình duyệt

**1. User Interface (UI):** bao gồm thanh địa chỉ, các button back/forware, button home, menu bookmarking, ... tất cả các phần của trình trình duyệt được hiển thị lên, ngoại trừ của sổ mà bạn thấy nội dung của trang web mà bạn yêu cầu.

**2. Brower engine:** Điều khiển hoạt động giữa UI và Rendering engine.

**3. Rendering engine:** chịu trách nhiệm hiển thị nội dung trang web, ví dụ nếu trang web trả về từ server là HTML thì Rendering engine sẽ phân tích cú pháp HTML và CSS để hiển thị nội dung đã được phân tích lên trên màn hình.

**4. Networking:** cung cấp network calls ví dụ như HTTP request, sử dụng các implementations khác nhau cho mỗi nền tảng độc lập khác nhau. (???)

**5. UI Backend:** được dùng để vẽ lên các tiện ích (widgets) cơ bản như 

**6. JavaScript Interpreter:** được dùng để phân tích và thực thi các câu lệnh JavaScript.

**7. Data storage:** đây là lớp persistence (1 loại lưu trữ lâu dài), dùng để lưu trữ các dữ liệu cần thiết tại local như cookie. Các trình duyệt cũng hỗ trợ các cơ chế lưu trữ như localStorage, IndexedDB, WebSQL và FileSystem.

**Chú ý:** các trình duyệt giống như Chrome sẽ chạy 1 ***rendering engine*** cho mỗi tab được mở. Điều này nghĩa là mỗi tab sẽ được chạy trong 1 tiến trình riêng biệt. 

###**2. The Rendering engine**
Nhiệm vụ của rendering engine đó chính là render ra kết quả của trang web từ tài liệu HTML để hiển thị lên trên cửa sổ của trình duyệt. Theo mặc định, rendering engine có thể hiển thị được các tài liệu HTML, XML và các hình ảnh. Đối với các kiểu dữ liệu khác, nó có thể dùng các plugin hoặc extension để hiển thị, ví dụ như để hiển thị tài liệu PDF thì rendering engine sử dụng plugin PDF viewer. Tuy nhiên trong tài liệu này, ta sẽ chỉ tập trung và trường hợp sử dụng chính của rendering engine: hiển thị HTML và các hình ảnh được định dạng sử dụng CSS. 

####**Các Rendering engine**
Mỗi trình duyệt sẽ sử dụng các rendering engine khác nhau: Internet Explorer sử dụng Trident, Firefox sử dụng Gecko, Safari sử dụng Webkit, Chrome và Opera (từ phiên bản 15) sử dụng Blink - 1 phân loại của Webkit. 

Webkit là 1 rendering engine mã nguồn mở, được bắt đầu xây dựng giống như 1 engine cho Linux platfrom và sau đó được sửa đổi bởi Apple để hỗ trợ cho Mac và Windows. Các bạn vào trang [webkit.org](https://webkit.org/) để biết thêm thông tin.

####**Luồng hoạt động chính**

Rendering engine sẽ bắt đầu nhận nội dung của tài liệu được yêu cầu từ tầng networking theo từng đoạn, mỗi đoạn có kích thước là 8kb. Sau đó, rendering engine thực hiện luồng như sau:

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/flow.png)

**Hình 2:** luồng hoạt động cơ bản của ***rendering engine***

B1: parsing HTML để xây dựng lên cây DOM

B2: xây dựng lên render tree 

B3: bố trí (sắp xếp) render tree

B4: vẽ render tree

Rendering engine sẽ bắt đầu phân tích tài liệu HTML và convert các element thành các node trong cây DOM, hay còn được gọi là ***"content tree"***. Rendering engine sẽ phân tích style của data, cả trong file CSS và trong các thẻ element. Các định dạng về style cùng với cấu trúc cây DOM được sử dụng để tạo ra 1 cây mới, đó là ***render tree***. Render tree chỉ chứa các node cần thiết để tạo nên được trang web.

![](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/render-tree-construction.png)

Các node trong render tree có các thuộc tính hình ảnh như màu sắc và kích thước. Các node này được sắp xếp theo thứ tự được hiển thị lên màn hình.

Sau khi xây dựng được render tree sẽ đến quá trình ***'layout'*** (bố trí). Quá trình này nhắm xác định chính xác tọa độ của 1 node trên màn hình. Bước tiếp theo là vẽ, render tree sẽ được load hết và mỗi node trong đó sẽ được vẽ lên màn hình bằng cách sử dụng tầng UI backend.

Điều quan trọng phải hiểu rằng đây là 1 quá trình từ từ (gradual process). Để người dùng có trải nghiệm tốt hơn, rendering engine sẽ cố gắng hiển thị nội dung lên màn hình sớm nhất có thể. Nó sẽ không đợi cho đến khi tất cả HTML được phân tích trước khi bắt đầu xây dựng và bố trí (sắp xếp) render tree. Từng phần của nội dung trang web sẽ được phân tích và hiển thị lên trong khi vẫn tiếp tục xử lý các phần nội dung còn lại.

####**Ví dụ về luồng hoạt động chính**

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/webkitflow.png)

**Hình 3:** luồng hoạt động chính của **Webkit**

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image008.jpg)

**Hình 4:** Luồng hoạt động chính của **Gecko**

Từ hình 3 và 4, ta thấy rằng mặc dù **Webkit** và **Gecko** sử dụng thuật ngữ hơi khác nhau nhưng luồng hoạt động cơ bản là như nhau.

Gecko gọi cây được tạo lên từ các element là ***'Frame tree'***, trong đó mỗi element là 1 frame. Còn Webkit sử dụng thuật ngữ ***'Render tree'***, bao gồm cả ***'Render objects'*** ở trong đó. Webkit dùng thuật ngữ ***'Layout'*** cho việc đặt chỗ các element, trong khi Gecko sử dụng ***'Reflow'***. ***'Attachment'*** là thuật ngữ mà Webkit dùng để thể hiện sự kết nối các node trong DOM và các node trong cây CSS để tạo nên render tree. Nhưng Gecko có 1 sự khác biệt nhỏ trong phần này, đó là Gecko có 1 lớp ***'Content Sink'*** giữa HTML và cây DOM, và đây là lớp để tạo nên các element trong cây DOM. Bây giờ ta sẽ cùng đi vào tìm hiểu từng thành phần một.

###**3. Quá trình phân tích và tạo nên cây DOM**

####**3.1 Phân tích - tổng hợp**

