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
## Class chính 
# MyBatis Native Image - Cấu hình và Giải thích  

## 1. `MyBatisBeanFactoryInitializationAotProcessor`  

### 📌 Chức năng  
Hỗ trợ Native Image bằng cách đăng ký thông tin cần thiết trước khi biên dịch.  

### 🔹 Công việc chính  
- Xác định danh sách `MapperFactoryBean` trong **Bean Factory**.  
- Đăng ký các class cần **reflection**, **proxy**, tài nguyên **XML**.  
- Hỗ trợ **SQL Provider** (`@SelectProvider`, `@InsertProvider`, …) để tránh lỗi runtime.  

---

## 2. `MyBatisMapperFactoryBeanPostProcessor`  

### 📌 Chức năng  
Xử lý các Bean `MapperFactoryBean` để đảm bảo hoạt động trong môi trường Native Image.  

### 🔹 Công việc chính  
- Kiểm tra xem **MyBatis** có tồn tại trong **classpath** không.  
- Điều chỉnh **constructor** và **target type** của `MapperFactoryBean` nếu cần thiết.  

---

## 3. `MyBatisMapperTypeUtils`  

### 📌 Chức năng  
Hỗ trợ xử lý **kiểu dữ liệu trả về** và **tham số của Mapper**.  

### 🔹 Công việc chính  
- Chuyển đổi **`Type` của phương thức Mapper** thành **`Class<?>`**.  
- Giúp Native Image hiểu rõ về các **kiểu dữ liệu** sử dụng trong MyBatis.  

---

## 🔥 Lưu ý khi sử dụng **MyBatis với GraalVM**  

✅ **Tránh sử dụng reflection không cần thiết**  
- GraalVM hạn chế reflection, vì vậy cần **đăng ký rõ ràng** để tránh lỗi runtime.  

✅ **Sử dụng `@MapperScan` hợp lý**  
- Nếu không, có thể gặp lỗi khi Native Image **không nhận diện** được các mapper.  

✅ **Đảm bảo tài nguyên XML được bao gồm**  
- Các file XML cần được khai báo trong **`resources`** khi build Native Image.  

