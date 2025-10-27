# prj-BRS
A Books Recommendation System Using Machine Learning

Hệ thống hoạt động dựa trên kiến trúc Microservice (dịch vụ vi mô), bao gồm hai phần chính chạy đồng thời:

🖥️ Ứng dụng Web (Laravel): Đóng vai trò là giao diện (frontend) và quản lý người dùng.

🧠 Dịch vụ AI (Flask): Đóng vai trò là "bộ não" (backend) chuyên xử lý và cung cấp các gợi ý.

Dưới đây là luồng hoạt động chi tiết khi một người dùng yêu cầu gợi ý:

Luồng hoạt động của BRSystem
Đây là quy trình 10 bước từ đầu đến cuối:

Phần 1: Ứng dụng Web (Laravel)
Xác thực người dùng:

Người dùng truy cập trang http://127.0.0.1:8000.

Hệ thống Laravel Breeze yêu cầu họ Đăng nhập (Log in) hoặc Đăng ký (Register).

Sau khi đăng nhập thành công, người dùng được chuyển đến trang /dashboard.

Người dùng gửi yêu cầu:

Trên trang Dashboard, người dùng nhập tên một cuốn sách (ví dụ: "The Hobbit") vào ô tìm kiếm và nhấn nút "Tìm gợi ý".

Controller tiếp nhận:

Form được gửi (POST) đến route /recommendations.

BookController (trong Laravel) tiếp nhận yêu cầu này.

Controller lấy 2 thông tin quan trọng:

Book Title: "The Hobbit" (từ Request).

User ID: Ví dụ: 512 (từ auth()->id(), tức ID của người vừa đăng nhập).

Laravel gọi "Bộ não" AI:

BookController sử dụng Http::post để gửi một yêu cầu API đến Dịch vụ Flask (đang chạy ở http://127.0.0.1:5001/hybrid_recommend).

Yêu cầu này mang theo một gói dữ liệu JSON chứa: { "user_id": 512, "book_title": "The Hobbit" }.

Phần 2: Dịch vụ AI (Flask)
Flask nhận nhiệm vụ:

Server app.py (Flask) nhận được yêu cầu tại endpoint /hybrid_recommend.

Nó đọc dữ liệu JSON và biết rằng: "Người dùng 512 muốn có gợi ý dựa trên sách 'The Hobbit'".

Chạy Mô hình Hybrid (Đây là phần cốt lõi):

Bước 6a: Lọc (Content-Based):

Hàm get_content_recommendations được gọi.

Nó sử dụng ma trận cosine_sim (đã tải từ file .pkl) để tìm ra 20 cuốn sách có nội dung (tên sách + tác giả) giống nhất với "The Hobbit".

Bước 6b: Xếp hạng (Collaborative Filtering):

Hệ thống duyệt qua 20 cuốn sách vừa tìm được.

Với mỗi cuốn sách, nó sử dụng mô hình svd_model (đã tải từ file .pkl) để dự đoán: "Người dùng 512 sẽ đánh giá cuốn sách này bao nhiêu điểm?" (ví dụ: "Artemis Fowl" - 8.5 điểm, "The Lost Boy" - 7.2 điểm...).

Bước 6c: Tổng hợp:

Danh sách 20 cuốn sách này được sắp xếp lại, dựa trên điểm dự đoán từ cao đến thấp.

Hệ thống chọn ra 10 cuốn sách có điểm dự đoán cao nhất.

Flask trả kết quả:

Dịch vụ Flask gửi một phản hồi JSON chứa danh sách 10 cuốn sách cuối cùng quay trở lại cho Laravel. (ví dụ: { "recommendations": ["Artemis Fowl", "The Eyre Affair", ...] }).

Phần 3: Quay lại Ứng dụng Web (Laravel)
Controller nhận kết quả:

BookController (trong Laravel) nhận được danh sách 10 cuốn sách từ Flask.

Hiển thị ra Giao diện:

Controller tải lại view dashboard.blade.php.

Nó truyền danh sách 10 cuốn sách này vào view.

Người dùng thấy Gợi ý:

File Blade (dashboard.blade.php) sử dụng vòng lặp @foreach để hiển thị 10 cuốn sách được gợi ý ra màn hình cho người dùng.

Sơ đồ luồng dữ liệu tóm tắt
Đây là cách dữ liệu di chuyển trong hệ thống của bạn:

Người dùng ➡️ Laravel (Gửi Form) ➡️ Laravel (Lấy User ID) ➡️ Flask (Nhận Yêu cầu) ➡️ Flask (Lọc Nội dung + Xếp hạng Cộng tác) ➡️ Flask (Trả JSON) ➡️ Laravel (Nhận JSON) ➡️ Laravel (Hiển thị View) ➡️ Người dùng

===

Installation and run

Bước 1: Lấy Mã nguồn (Clone)
Đầu tiên, họ cần tải mã nguồn từ repository GitHub của bạn về máy.

Bash

# Mở Terminal và chạy lệnh sau
git clone [URL_repository_GitHub_của_bạn]

# Di chuyển vào thư mục dự án vừa tải về
cd [Tên_thư_mục_dự_án]
Bước 2: Cài đặt 🖥️ Ứng dụng Web (Laravel)
Phần này cài đặt giao diện web và quản lý người dùng.

Di chuyển vào thư mục Laravel:

Bash

cd laravel_app
Cài đặt các thư viện PHP (Composer):

Bash

composer install
Tạo file môi trường (.env): Sao chép file .env.example thành file .env.

Bash

# (Dành cho Windows)
copy .env.example .env
Cấu hình Cơ sở dữ liệu:

Mở công cụ quản lý CSDL (như phpMyAdmin) và tạo một database mới (ví dụ: brsystem_db).

Mở file .env vừa tạo và cập nhật các dòng sau:

Code snippet

DB_DATABASE=brsystem_db
DB_USERNAME=root
DB_PASSWORD= (mật khẩu của bạn, có thể bỏ trống)
Sinh khóa ứng dụng (App Key):

Bash

php artisan key:generate
Chạy Migration: Lệnh này sẽ tạo các bảng trong cơ sở dữ liệu (như users, password_resets...):

Bash

php artisan migrate
Cài đặt các gói JavaScript/CSS:

Bash

npm install
Biên dịch tài nguyên (Assets):

Bash

npm run dev
Bước 3: Cài đặt 🧠 Dịch vụ AI (Flask)
Phần này cài đặt "bộ não" AI để cung cấp gợi ý. Đây là bước quan trọng nhất vì nó liên quan đến các mô hình đã huấn luyện.

Di chuyển vào thư mục Dịch vụ AI:

Bash

cd ../ml_service
# (Nếu bạn đang ở root, chỉ cần gõ: cd ml_service)
Tạo và kích hoạt môi trường ảo (venv):

Bash

# 1. Tạo môi trường ảo
python -m venv venv

# 2. Kích hoạt môi trường (Windows)
.\venv\Scripts\activate
Cài đặt các thư viện Python: File requirements.txt mà chúng ta đã tạo sẽ tự động cài đặt tất cả các thư viện cần thiết (như Flask, Pandas, Surprise...):

Bash

pip install -r requirements.txt
Huấn luyện lại Mô hình (Bước Bắt buộc): Các file mô hình (.pkl) là các file nhị phân rất lớn và theo thông lệ, chúng không được đưa lên GitHub. Do đó, người dùng mới phải tự chạy lại các notebook để tạo ra chúng.

Khởi động Jupyter Notebook:

Bash

jupyter notebook
Chạy Notebook 1: Mở và chạy toàn bộ file notebooks/03-Hybrid_Dataset_Creation.ipynb. Bước này để tạo ra bộ dữ liệu final_df.

Chạy Notebook 2: Mở và chạy toàn bộ file notebooks/04-Hybrid_Model_Training.ipynb. Bước này sẽ huấn luyện và lưu các file .pkl vào đúng thư mục models/.

Bước 4: Chạy Ứng dụng 🚀
Sau khi đã cài đặt xong cả hai phần, họ cần mở hai Terminal và chạy song song.

Terminal 1 (Chạy "Bộ não" AI):

Bash

cd [Đường_dẫn_đến_dự_án]/ml_service
.\venv\Scripts\activate
python app.py
Giữ Terminal này chạy.

Terminal 2 (Chạy "Web" Laravel):

Bash

cd [Đường_dẫn_đến_dự_án]/laravel_app
php artisan serve
Giữ Terminal này chạy.

Bây giờ, họ có thể truy cập http://127.0.0.1:8000 trên trình duyệt để đăng ký và sử dụng trang web.
