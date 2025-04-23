# Bai5_Triggger
# Bài tập 5 của Nguyễn Đức Việt - K225480106075 - Lớp K58KTP - MÔN HQT CSDL
## Nội dung bài tập 5:
1. Dựa trên cơ sở là csdl của Đồ án
2. Tìm cách bổ xung thêm 1 (hoặc vài) trường phi chuẩn
   (là trường tính toán đc, nhưng thêm vào thì ok hơn,
    ok hơn theo 1 logic nào đó, vd ok hơn về speed)
   => Nêu rõ logic này!
3. Viết trigger cho 1 bảng nào đó, 
   mà có sử dụng trường phi chuẩn này,
   nhằm đạt được 1 vài mục tiêu nào đó.
   => Nêu rõ các mục tiêu 
4. Nhập dữ liệu có kiểm soát, 
   nhằm để test sự hiệu quả của việc trigger auto run.
5. Kết luận về Trigger đã giúp gì cho đồ án của em.
## A. Đồ án PTTKHT: Phân tích thiết kế hệ thống quản lí quán trà sữa
### Cơ sở dữ liệu
#### Các bảng
![image](https://github.com/user-attachments/assets/3951c29b-1e99-436b-9c3d-9100916fdfa4)
![image](https://github.com/user-attachments/assets/482b63c2-0042-4488-a292-c1d52b1d28c8)
![image](https://github.com/user-attachments/assets/5924c01e-54d9-4eac-9616-20a359b6394f)
![image](https://github.com/user-attachments/assets/b071478a-f3d5-40aa-b02b-f9f09697d607)
#### Sơ đồ thực thể liên kết
![image](https://github.com/user-attachments/assets/ce237392-2d29-464d-865d-1106781b299f)
#####LOGIC
Logic cải thiện:
-    Tăng tốc truy vấn: Thay vì phải đếm số lượng món trong ChiTietDonHang bằng COUNT mỗi khi cần biết số món trong một đơn hàng, trường SoLuongMon lưu sẵn giá trị này, giảm chi phí tính toán, đặc biệt khi bảng ChiTietDonHang có nhiều bản ghi.
-    Hỗ trợ phân tích: Số lượng món là thông tin hữu ích để phân tích hành vi khách hàng (ví dụ: khách hàng thường gọi bao nhiêu món mỗi lần) hoặc tối ưu quy trình kinh doanh (chuẩn bị nguyên liệu, dự đoán thời gian xử lý đơn).
-   Tối ưu báo cáo: Các báo cáo thống kê như "số món trung bình mỗi đơn hàng" sẽ nhanh hơn vì không cần truy vấn liên bảng.
##### Trigger
-   Trigger: Tạo trigger TR_CapNhat_SoLuongMon trên bảng ChiTietDonHang để tự động cập nhật SoLuongMon trong bảng DonHang khi có thao tác thêm, xóa hoặc sửa chi tiết đơn hàng.
```sql
-- Trigger tự động cập nhật ThanhTien khi thêm/sửa ChiTietDonHang
CREATE TRIGGER TR_CapNhat_ThanhTien
ON ChiTietDonHang
AFTER INSERT, UPDATE
AS
BEGIN
    UPDATE ct
    SET ct.ThanhTien = ct.SoLuong * m.Gia
    FROM ChiTietDonHang ct
    JOIN MonAn m ON ct.MaMon = m.MaMon
    WHERE EXISTS (
        SELECT 1
        FROM inserted i
        WHERE i.MaDH = ct.MaDH AND i.MaMon = ct.MaMon
    );
END;
GO

-- Trigger tự động cập nhật TongTien khi thêm/sửa/xóa ChiTietDonHang
CREATE TRIGGER TR_CapNhat_TongTien
ON ChiTietDonHang
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    UPDATE dh
    SET dh.TongTien = ISNULL((
        SELECT SUM(ThanhTien)
        FROM ChiTietDonHang ct
        WHERE ct.MaDH = dh.MaDH
    ), 0)
    FROM DonHang dh
    WHERE dh.MaDH IN (
        SELECT MaDH FROM inserted
        UNION
        SELECT MaDH FROM deleted
    );
END;
GO

-- Trigger tự động cập nhật SoLuongMon khi thêm/sửa/xóa ChiTietDonHang
CREATE TRIGGER TR_CapNhat_SoLuongMon
ON ChiTietDonHang
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    UPDATE dh
    SET dh.SoLuongMon = ISNULL((
        SELECT COUNT(DISTINCT MaMon)
        FROM ChiTietDonHang ct
        WHERE ct.MaDH = dh.MaDH
    ), 0)
    FROM DonHang dh
    WHERE dh.MaDH IN (
        SELECT MaDH FROM inserted
        UNION
        SELECT MaDH FROM deleted
    );
END;
-- Trigger cảnh báo sdt k đủ 10 số
CREATE TRIGGER TR_Check_SDT_KhachHang
ON KhachHang
AFTER INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM inserted
        WHERE LEN(Sdt) != 10 OR ISNUMERIC(Sdt) = 0
    )
    BEGIN
        -- Chỉ cần RAISERROR là đủ
        RAISERROR(N'Số điện thoại phải có đúng 10 chữ số và là số!', 16, 1);
        -- Không cần ROLLBACK TRANSACTION
    END
END;

```
####Test Trigger
![image](https://github.com/user-attachments/assets/b61a0866-7755-4661-b202-db16452d5870)
![image](https://github.com/user-attachments/assets/9cbd2bdd-94d2-418a-8c59-b640e38c50b5)
![image](https://github.com/user-attachments/assets/a54beeca-3aba-48f3-9a66-28393859d7cd)
![image](https://github.com/user-attachments/assets/738eba23-56f5-4edb-94dd-95b7688d1ae5)
![image](https://github.com/user-attachments/assets/320d218b-9cd9-45b5-8706-e52a8168bc18)
![image](https://github.com/user-attachments/assets/23956d7e-10d7-418e-8379-5f851d10e5d7)




