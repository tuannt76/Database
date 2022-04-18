# tìm hiểu về quorum


- [tìm hiểu về quorum](#tìm-hiểu-về-quorum)
- [Nội dung](#nội-dung)
  - [Quorum là gì?](#quorum-là-gì)
  - [Weighted Quorum](#weighted-quorum)
- [Tài liệu tham khảo:](#tài-liệu-tham-khảo)


# Nội dung

  ## Quorum là gì?

  Là giải pháp tránh sự "split brain" trong cluster - một trường hopjq mà cluster được tách thành nhiều phần. Nhưng mỗi phần lại tiếp tục hoạt động như những cluster riêng biệt, có khả năng ghi vào cùng một dữ liệu và  có thể gây ra mất mát dữ liệu.

 + Để duy trì tính toàn vẹn và sẵn có của cluster, các hệ thống cluster sử dụng khái niệm này để ngắn ngừa sự mất mát dữ liệu. Một cluster có quorum khi có hơn một nửa số lượng các node đang có trạng thái online. Để giảm thiểu nguy cơ mất mát dữ liệu, mặc định pacemaker sẽ ngừng tất cả các resource nếu cluster không có quorum

+ quorum được hình thành bằng các sử dụng `voting system` khi mà một node trong cluster không hoạt động như mong muốn hoặc mất liên lạc với phần còn lại của cluster, các node còn lại trong cluster có thể thực hiện `vote` để cô lập và nếu cần thiết thì loại bỏ node đang bị lỗi, ... này ra khỏi cluster

+ Ví dụ: trong một cluster có 6 node và số node đang hoạt động cần có ít nhất là 4 node mới có thể hình thành lên quorum. Nếu đa số các node không hoạt động hoặc trở nên không khả dụng thì mặc định cụm không có quorum và pacemaker sẽ ngừng các dịch vụ trong cluster.
    
+ Trong trường hợp mà số lượng các node khả dụng và không khả dụng bằng nhau quorum có thể được cấu hình để có chính sách `tiebreaker` để tiếp tục quorum và duy trì các node trong cluster còn lại và dành quorum cho node mà có id thấp nhất.

## Weighted Quorum

- Số lượng nude hiện tại trong cụm (cluster) xác định kích thước cụm hiện tại.
- Không có cài đặt cấu hình nào cho phép bạn xác định danh sách tất cả các nút cụm có thể có. 
  - Mỗi khi một node tham gia vào cụm, tổng kích thước cụm sẽ tăng lên. 
  - Khi một nút rời khỏi cụm, một cách duyên dáng, kích thước cụm giảm. 
- Kích thước cụm xác định số phiếu bầu (votes) cần thiết để đạt được **Quorum**.

- Galera Cluster lấy `quorum vote` bất cứ khi nào `một nút không phản hồi` và bị nghi ngờ `không còn là một phần của cụm`. Bạn có thể tinh chỉnh điều này không có thời gian chờ phản hồi bằng cách sử dụng tham số `evs.suspect_timeout` . Cài đặt mặc định là 5 giây.

- Khi cụm lấy `quorum vote`, nếu phần lớn tổng số nút được kết nối từ trước khi ngắt kết nối vẫn còn, thì phân vùng đó vẫn hoạt động. Khi phân vùng mạng xảy ra, có các nút hoạt động ở cả hai phía của ngắt kết nối. 
  - Thành phần có `quorum` một mình tiếp tục hoạt động như `Primary Component` , 
  - Những thành phần không có `quorum` đi vào trạng thái `non-primary` và bắt đầu cố gắng kết nối với `Thành phần chính`.


![](./image/Croum_1.png)

- Công thức tính quorum (tức số node tối thiểu để cụm hoạt động bình thường): A majority (> 50%) of nodes
```
(Số node hoạt động) > (tổng số node của cụm)/2
```
- Quorum yêu cầu đa số, có nghĩa là bạn không thể có chuyển đổi dự phòng tự động trong **một cụm hai nút**. Điều này là do sự cố của một nút khiến nút còn lại tự động chuyển sang trạng thái `non-primary`.

- Các `cluster` có `số lượng node chẵn` có nguy cơ dẫn đến tình trạng não bị phân chia (**split-brain**). Nếu bạn mất kết nối mạng ở đâu đó giữa các phân vùng theo cách khiến **số lượng node bị chia đôi** chính xác, thì :
  - cả hai phân vùng đều không thể giữ lại `quorum`
  - cả hai đều chuyển sang trạng thái `non-primary`.

![](./image/Croum_2.png)

- Để bật chuyển đổi dự phòng tự động (**automatic failovers**), bạn cần sử dụng **ít nhất ba nút**. Hãy nhớ rằng điều này mở rộng ra các cấp độ cơ sở hạ tầng khác, vì những lý do tương tự:
  - **Single switch clusters** nên sử dụng **tối thiểu 3 node**.
  - **Clusters spanning switches** nên sử dụng **tối thiểu 3 switches**.
  - **Clusters spanning networks** nên sử dụng tối thiểu **3 networks**.
  - **Clusters spanning data centers** nên sử dụng tối thiểu **3 data centers**.

# Tài liệu tham khảo:
1. https://galeracluster.com/library/documentation/weighted-quorum.html
2. https://galeracluster.com/library/documentation/glossary.html#term-global-transaction-id
3. https://cuongquach.com/split-brain-trong-elasticsearch-cluster.html
