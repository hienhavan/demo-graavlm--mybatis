# MyBatis Native Configuration  

## Giới thiệu  

Demo này nhằm hỗ trợ tích hợp **MyBatis** trong môi trường **GraalVM Native Image**.  
Bình thường, MyBatis dựa vào **reflection** và **runtime class generation**, nhưng trong môi trường Native Image, những cơ chế này có thể bị hạn chế hoặc không hoạt động.  

### Vì sao cần demo này?  

- **Hỗ trợ chạy MyBatis trên Native Image**: GraalVM loại bỏ reflection không cần thiết để tối ưu hóa hiệu suất. Tuy nhiên, MyBatis lại phụ thuộc vào reflection.  
- **Tránh lỗi liên quan đến reflection**: Nếu không có cấu hình phù hợp, MyBatis sẽ gặp lỗi khi chạy trong môi trường Native Image.  
- **Cung cấp giải pháp xử lý trước (AOT Processing)**: Tự động đăng ký thông tin cần thiết cho Native Image mà không cần can thiệp thủ công.  

---

## Cách hoạt động  

Cấu hình này thực hiện các công việc chính:  

1. **Đăng ký các Bean cần thiết cho MyBatis**  
   - `MyBatisBeanFactoryInitializationAotProcessor` đảm bảo MyBatis hoạt động với Native Image.  
   - `MyBatisMapperFactoryBeanPostProcessor` xử lý mapper để tương thích với môi trường AOT.  

2. **Xử lý trước các thành phần của MyBatis**  
   - Định danh các mapper và kiểu dữ liệu liên quan.  
   - Đăng ký các proxy và tài nguyên XML để tránh lỗi runtime.  

3. **Tối ưu hóa khả năng tương thích với GraalVM**  
   - Đăng ký reflection cần thiết.  
   - Tránh khởi tạo các class không cần thiết trong quá trình build.  

---
