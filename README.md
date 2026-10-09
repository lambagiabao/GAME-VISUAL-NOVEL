# Office Life

Office Life là game visual novel 2D giúp người chơi luyện giao tiếp công sở.
Người chơi chọn cách phản hồi trong các tình huống; lựa chọn làm thay đổi chỉ
số kỹ năng, quan hệ nhân vật, lời thoại tiếp theo và kết quả của hành trình.

## Tìm nhanh trong dự án

- Chạy game: `python run.py` (hướng dẫn tại **Chạy game và backend**).
- Đổi video: chép file vào [`client/assets/videos/`](client/assets/videos/).
- Đổi mốc cắt/lặp hoặc xử lý âm thanh video:
  [`client/managers/intro_video_manager.py`](client/managers/intro_video_manager.py).
- Sửa bố cục/nút trang chủ và thanh âm lượng:
  [`client/main.py`](client/main.py).
- Sửa danh sách và cách replay tình huống:
  `draw_library()`, `library_play:<scenario_id>` và
  `finish_library_replay()` trong [`client/main.py`](client/main.py).
- Xem chức năng từng file:
  [README_CODE.md](README_CODE.md).

## Tính năng hiện có

- 15 tình huống trong 5 chapter, có nhánh lựa chọn và lời thoại thay đổi theo
  trạng thái quan hệ.
- Bốn chỉ số kỹ năng: thấu cảm, chuyên nghiệp, niềm tin và kết quả; có thêm
  quan hệ với quản lý, đồng nghiệp và khách hàng.
- Màn hình kết quả cho biết phản ứng, phân tích, thay đổi chỉ số/quan hệ, XP
  và lịch sử quyết định.
- Thư viện cho phép chọn chơi thử lại bất kỳ tình huống nào; lượt luyện tập
  chạy độc lập, không ghi đè save, điểm hoặc thứ hạng.
- Lưu tiến độ offline; đăng ký/đăng nhập và đồng bộ save tài khoản lên backend.
- Bảng xếp hạng toàn server có phân trang và đánh dấu người chơi đang đăng nhập.
- Hồ sơ, thống kê, thành tựu, cài đặt âm thanh, đổi ngôn ngữ Việt/Anh.
- Form tài khoản có đặt lại mật khẩu cục bộ, hiện/ẩn mật khẩu, đặt con trỏ và
  chọn/xóa/thay thế văn bản bằng chuột hoặc bàn phím.
- Video menu dài 26 giây: phát toàn clip lúc mở game, sau đó lặp đoạn giây
  11–26; âm thanh gốc được phát cùng video.
- Backend FastAPI có xác thực JWT, phân quyền admin và API quản trị nội dung.

## Trạng thái lưu trữ

- **Đang dùng khi chạy ứng dụng:** MariaDB/MySQL của XAMPP, kết nối qua
  PyMySQL. Backend tự khởi tạo schema trong
  [`server/database/mariadb_schema.sql`](server/database/mariadb_schema.sql).
- **Kiểm thử:** SQLite được dùng trong test và có thể cấu hình riêng bằng
  `DATABASE_URL`.
- [`server/database/schema.sql`](server/database/schema.sql) là schema tham
  khảo PostgreSQL. Runtime hiện tại **không** có adapter PostgreSQL/Supabase.
- Client không kết nối database trực tiếp: dữ liệu tài khoản đi qua FastAPI.
  Save offline của khách độc lập với snapshot account trên server.

## Yêu cầu

- Python 3.11 trở lên (cần phiên bản Python tương thích với các dependencies
  trong `requirements.txt`).
- XAMPP với MySQL/MariaDB nếu muốn sử dụng tài khoản và đồng bộ server.
- Windows PowerShell cho các lệnh ví dụ dưới đây.

## Cài đặt

Tại thư mục gốc của dự án:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Tạo bản cấu hình cục bộ từ mẫu nếu chưa có `.env`:

```powershell
Copy-Item .env.example .env
```

Trong XAMPP Control Panel, bật **MySQL**. Tạo database và user ứng dụng riêng
bằng phpMyAdmin hoặc chạy SQL dưới đây trong tab SQL (đổi mật khẩu mẫu trước
khi chạy):

```sql
CREATE DATABASE office_life CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'office_life_app'@'127.0.0.1' IDENTIFIED BY 'CHANGE_THIS_PASSWORD';
GRANT ALL PRIVILEGES ON office_life.* TO 'office_life_app'@'127.0.0.1';
FLUSH PRIVILEGES;
```

Điền `DATABASE_URL` trong `.env` theo định dạng:

```text
mysql+pymysql://office_life_app:MAT_KHAU_CUA_BAN@127.0.0.1:3306/office_life?charset=utf8mb4
```

Thay `MAT_KHAU_CUA_BAN` bằng mật khẩu đã tạo. Không đưa mật khẩu thật vào
README, không chia sẻ `.env` và không commit file này. Nếu database/user đã
tồn tại thì không cần chạy lại các lệnh `CREATE`.

## Chạy game và backend

Game offline:

```powershell
python run.py
```

Để dùng đăng ký, đăng nhập, đồng bộ save và bảng xếp hạng, bật XAMPP MySQL rồi
mở terminal thứ nhất tại thư mục dự án:

```powershell
python -m uvicorn server.main:app --reload
```

Giữ backend đang chạy. Mở terminal thứ hai và chạy:

```powershell
python run.py
```

Trong game, vào **ĐĂNG NHẬP TÀI KHOẢN** để đăng ký hoặc đăng nhập. Backend mặc
định ở `http://127.0.0.1:8000`; có thể đổi địa chỉ bằng `API_BASE_URL`. Kiểm
tra backend tại `http://127.0.0.1:8000/health`, xem API tại
`http://127.0.0.1:8000/docs`.

Backend không chạy được nếu MySQL chưa bật hoặc `DATABASE_URL` sai. Lệnh
`python -m uvicorn` dùng được cả khi chương trình `uvicorn` chưa có trên
PATH; nếu báo thiếu module, kích hoạt virtual environment và cài
`requirements.txt`.

### Thêm video intro trang chủ

Chép video MP4 có cả hình và âm thanh vào:

```text
client/assets/videos/office_life_intro.mp4
```

Nếu tên video khác, chỉ cần đổi tên file thành `office_life_intro.mp4`. Video
được phát một lượt từ đầu đến giây 26; sau đó game lặp lại đoạn từ giây 11 đến
hết clip. Âm thanh video cũng lặp cùng đoạn chờ và tạm dừng khi rời trang chủ.
Nếu chưa có file, game vẫn chạy với hình minh họa mặc định và hiện đường dẫn
cần đặt video.

Góc dưới bên phải clip luôn có nút loa nhỏ để tắt/bật tiếng. Di chuột tới góc
này để hiện thanh trượt âm lượng gọn; rời chuột thì thanh tự ẩn sau một nhịp
fade. Âm lượng mặc định là 75% và lựa chọn chỉ áp dụng cho lần chạy hiện tại.
Nếu máy tải clip 4K chậm, nên xuất bản bản 720p hoặc 1080p; game cũng giảm
khung giải mã xuống tối đa 960 pixel chiều ngang.

#### Muốn đổi thời điểm cắt và đoạn lặp?

Mở [`client/managers/intro_video_manager.py`](client/managers/intro_video_manager.py):

| Thành phần | Chức năng |
|---|---|
| `IntroVideoManager.INTRO_END_SECONDS` | Mốc thời gian (giây) kết thúc intro phát một lần. Hiện đặt `26.0`. |
| `IntroVideoManager.LOOP_START_SECONDS` | Mốc (giây) bắt đầu đoạn chờ được lặp. Hiện đặt `11.0`. |
| `IntroVideoManager._extract_audio()` | Dùng FFmpeg cắt soundtrack đoạn lặp từ `LOOP_START_SECONDS` đến `INTRO_END_SECONDS`. |
| `IntroVideoManager._open_decoder()` | Tìm và giải mã frame; bộ lọc `scale=960:-2` giảm chiều ngang video để giảm bộ nhớ và tải render. |
| `IntroVideoManager.update()` | Theo dõi thời gian phát; đến `INTRO_END_SECONDS` thì seek hình về `LOOP_START_SECONDS` và bắt đầu lặp âm thanh. |
| `IntroVideoManager.set_volume()` / `toggle_mute()` | Điều chỉnh âm lượng hoặc tắt/bật tiếng qua điều khiển ở góc video. |
| `OfficeLifeApp.draw_video_audio_controls()` trong `client/main.py` | Vẽ nút loa nhỏ và thanh âm lượng chỉ hiện khi rê chuột vào góc clip; `set_video_volume_from_mouse()` xử lý vị trí click/kéo. |

Ví dụ: muốn intro phát 8 giây rồi lặp từ giây 8 đến hết clip, đổi hai hằng
số thành `INTRO_END_SECONDS = 8.0` và `LOOP_START_SECONDS = 8.0`.

## Database và tài khoản

Backend tự tạo/cập nhật các bảng MariaDB khi khởi động. Các bảng chính:

| Bảng | Chức năng |
|---|---|
| `users` | Tên đăng nhập, email, hash mật khẩu, quyền và trạng thái tài khoản |
| `game_saves` | Snapshot tiến độ game của mỗi tài khoản |
| `progress`, `scores` | Tiến độ scenario và dữ liệu điểm |
| `scenarios`, `chapters` | Nội dung game quản lý qua API |
| `achievements`, `user_achievements` | Danh mục thành tựu và thành tựu đã đạt |

Mật khẩu gốc không được lưu; chỉ có password hash. Để xem tài khoản trong
phpMyAdmin, chọn database `office_life` → bảng `users` → **Browse**. Không sửa
`password_hash` trực tiếp.

Nếu quên mật khẩu, chọn **QUÊN MẬT KHẨU?** trên màn hình đăng nhập để xem hướng
dẫn, hoặc chạy `python -m server.reset_password` trong terminal. Đây là công
cụ đặt lại cục bộ cần truy cập máy chạy backend/database; dự án chưa có luồng
khôi phục mật khẩu qua email.

## Kiểm thử

```powershell
python -m pytest -q
python -m compileall -q client server tests
```

Test dùng database SQLite riêng biệt; không cần xóa dữ liệu MariaDB/XAMPP để
chạy test.

## Bản đồ mã nguồn

| Vị trí | Vai trò |
|---|---|
| [`run.py`](run.py) | Điểm khởi động desktop game |
| [`client/main.py`](client/main.py) | Điều phối giao diện, input, tài khoản và các màn hình Pygame |
| [`client/game/`](client/game/) | Game state, branching, hiệu ứng và kết quả lựa chọn |
| [`client/managers/`](client/managers/) | Lưu tiến độ, ngôn ngữ và âm thanh |
| [`client/services/api_client.py`](client/services/api_client.py) | Giao tiếp HTTP với backend |
| [`client/data/scenarios/content.json`](client/data/scenarios/content.json) | Nội dung 15 tình huống |
| [`client/localization/`](client/localization/) | Chuỗi giao diện tiếng Việt và tiếng Anh |
| [`server/`](server/) | FastAPI, xác thực, database, nghiệp vụ API |
| [`tests/`](tests/) | Kiểm thử game, API và cấu trúc dự án |

Danh mục **từng file mã nguồn**, gồm vai trò của các module và test, nằm trong
[README_CODE.md](README_CODE.md). Kiến trúc hệ thống và luồng dữ liệu được mô
tả lần lượt trong [ARCHITECTURE.md](ARCHITECTURE.md) và
[DATA_FLOW.md](DATA_FLOW.md).

## Triển khai

Ứng dụng hiện được hướng dẫn chạy desktop local. Có thể thử đóng gói client
web bằng `pygbag`, nhưng đây chưa phải cấu hình phát hành web đã được xác nhận
cho tất cả tính năng; luồng tài khoản cần backend truy cập được từ client.
Để triển khai server ra Internet, cần cấu hình HTTPS, `JWT_SECRET_KEY` an
toàn, database quản lý riêng và `API_BASE_URL` phù hợp. Không dùng thông tin
đăng nhập XAMPP phát triển cho môi trường công khai.
