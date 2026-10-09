# Office Life - Giải thích mã nguồn và thuật toán

## 1. Tổng quan kiến trúc

```text
Người chơi
   ↓
Frontend Pygame (client/)
   ↓ HTTP REST + JWT (khi bật backend)
Backend FastAPI (server/)
   ↓
MariaDB/MySQL XAMPP (SQLite chỉ dành cho test)
```

Game có hai chế độ:

- **Offline:** Pygame đọc scenario từ `client/data/scenarios/content.json` và lưu
  tiến độ vào `client/data/saves/progress.json`.
- **Tài khoản:** `client/services/api_client.py` gọi FastAPI để đăng nhập, đồng bộ
  snapshot tiến độ. Save tài khoản nằm trong bảng SQL `game_saves`; save khách
  offline riêng nên không ghi đè tiến độ account. Backend kết nối MariaDB bằng
  PyMySQL; SQLite có thể bật riêng cho kiểm thử bằng `DATABASE_URL`.

## 2. Frontend

### `run.py`

Điểm vào duy nhất của ứng dụng. File import `main()` từ `client.main` rồi
khởi động vòng lặp game bằng `python run.py`.

### `client/main.py`

Đây là bộ điều phối giao diện:

1. Khởi tạo Pygame, cửa sổ, clock và font sans-serif.
2. Nạp ngôn ngữ, save local và `BranchingEngine`.
3. Vẽ các màn hình menu, gameplay, kết quả, thư viện, profile, settings và
   tổng kết.
4. Nhận sự kiện chuột/phím, bao gồm form đăng nhập/đăng ký.
5. Khi người chơi bấm lựa chọn, gọi engine, lưu state local hoặc lên SQL và
   chuyển màn hình.
6. Khi kết thúc scenario cuối, hiển thị summary và bộ đếm 8 giây; hết giờ quay
   về menu chính.

Vòng lặp chính có dạng:

```text
đọc event → xử lý input → vẽ màn hình → kiểm tra timer → giới hạn 60 FPS
```

Các class/function trong file đã có docstring tiếng Việt ngay tại source để
giải thích nhiệm vụ, dữ liệu đầu vào/đầu ra và luồng chính. Những đoạn cần hiểu
theo thứ tự (khởi tạo app, render screen, xử lý lựa chọn và countdown) có thêm
comment tại chỗ. Comment không lặp lại các phép gán đơn giản để mã vẫn dễ đọc.

Form tài khoản hỗ trợ đặt caret bằng click, kéo chuột để chọn, `Ctrl+A`,
`Shift` + phím điều hướng, xóa/thay thế vùng chọn, hiện/ẩn mật khẩu và chuyển
nút tài khoản ở menu thành đăng xuất khi người chơi đã đăng nhập.

### `client/game/game_state.py`

`GameState` là dữ liệu phiên chơi:

- `scenario_id`: scenario hiện tại.
- `stats`: empathy, professionalism, trust, outcome.
- `characters`: mức tin tưởng của manager, colleague, customer.
- `completed`, `choices`, `xp`, `score`, `achievements`; mỗi quyết định lưu
  timestamp, delta chỉ số và delta quan hệ nhân vật.

`apply_effects()` cộng/trừ chỉ số và giới hạn trong 0..100. `ending()` chọn
ending bằng nhiều điều kiện, không chỉ một tổng điểm.

### `client/game/branching_engine.py`

Engine đọc JSON, chuyển dữ liệu thành `Scenario` và `Choice`.

Thuật toán `choose()`:

```text
nhận choice_id
→ kiểm tra choice có thuộc scenario hiện tại không
→ áp dụng score effects
→ cập nhật character state
→ ghi scenario/choice, timestamp và delta vào lịch sử
→ cộng XP và score
→ chuyển sang next_scenario; hội thoại kế tiếp có thể đổi theo quan hệ
```

### `client/managers/language_manager.py`

Đọc `vi.json` hoặc `en.json`, tìm key dạng `game.summary` và thay biến như
`{seconds}`. Vì vậy text giao diện không cần rải trực tiếp trong logic.

### `client/managers/save_manager.py`

Tuần tự hóa/khôi phục `GameState` thành JSON, giữ tương thích save khách cũ.
State tài khoản được gửi tới endpoint `/api/progress/state`; backend lưu JSON
snapshot theo user ID và trả lại sau khi đăng nhập.

### `client/services/api_client.py`

Lớp HTTP client dùng `httpx`, tự dùng prefix `/api`, truyền username/email
đăng nhập và thêm JWT vào header `Authorization`. Lỗi đồng bộ được hiện cho
người chơi, không giả lập thông báo lưu thành công.

### `client/managers/audio_manager.py`

Tạo và phát sound effect xác nhận khi người chơi chọn case. Âm thanh được tạo
bằng PCM runtime nên không cần asset `.wav` bắt buộc. Nếu mixer hoặc thiết bị
audio không khởi tạo được, manager tự chuyển sang silent mode để gameplay vẫn
chạy bình thường. Người chơi có thể bật/tắt âm thanh tại Settings; lựa chọn
được lưu tại `client/data/settings.json` và file cài đặt cục bộ được gitignore.

### `client/localization/`

- `vi.json`: text tiếng Việt.
- `en.json`: text tiếng Anh.

### `client/data/scenarios/content.json`

Thư viện 15 scenario thuộc 5 chapter. Mỗi scenario có dialogue, 3 lựa chọn,
điểm ảnh hưởng, character effects, phản ứng, phân tích, scenario kế tiếp và có
thể khai báo dialogue variants phụ thuộc trạng thái quan hệ.

## 3. Backend

### `server/main.py`

Tạo FastAPI app, bật CORS, khởi tạo database khi startup và gắn các router:

- `/api/auth`
- `/api/scenarios`
- `/api/progress`
- `/api/scores`
- `/api/achievements`
- `/api/leaderboard`
- `/api/analytics`

### `server/routers/auth.py`

Xử lý đăng ký, đăng nhập, thông tin user, cập nhật profile và danh sách user
dành cho admin. Mật khẩu luôn được hash, không lưu plain text.

### `server/routers/content.py`

CRUD scenario/chapter. Endpoint ghi dữ liệu dùng `admin_only`, nên user thường
không thể tạo, sửa hoặc xóa nội dung.

### `server/routers/game.py`

Lưu/đọc progress snapshot, scores, achievements, leaderboard và analytics.
Snapshot và thành tựu người chơi được lọc theo user lấy từ JWT; bảng xếp hạng
và thống kê điểm dựa trên tiến độ thực tế. Leaderboard công khai phân trang;
endpoint `/api/leaderboard/me` yêu cầu JWT để trả đúng hạng toàn cục của user.

### `server/auth/security.py`

Chứa hash/verify password và tạo/giải mã JWT.

### `server/dependencies.py`

`current_user()` kiểm tra bearer token, đọc user từ database. `admin_only()`
kiểm tra role admin trước khi cho phép thao tác quản trị.

### `server/database/db.py`

Tạo kết nối MariaDB qua PyMySQL và chạy schema InnoDB từ
`server/database/mariadb_schema.sql` nếu chưa tồn tại. Các
bảng chính gồm users, scenarios, chapters, progress, `game_saves`, scores,
achievements và user_achievements. SQLite chỉ được chọn riêng bằng
`DATABASE_URL` để chạy kiểm thử.

### `server/reset_password.py`

Lệnh quản trị local đặt mật khẩu mới theo username/email. `getpass` không hiện
mật khẩu khi nhập; database chỉ nhận password hash do hàm xác thực tạo ra.

### `server/database/schema.sql`

Schema PostgreSQL dùng để tạo bảng tương đương trên Supabase.

### `server/schemas/api.py`

Pydantic models kiểm tra input/output API, độ dài password, email và dữ liệu
scenario.

## 4. Luồng đăng nhập

```text
Pygame Account screen
→ POST /api/auth/register hoặc /api/auth/login
→ validate Pydantic
→ hash password
→ INSERT/SELECT users
→ tạo JWT
→ GET /api/progress/state
→ khôi phục snapshot của đúng tài khoản
```

Khi gọi API bảo vệ:

```text
Bearer token
→ decode JWT
→ lấy user
→ kiểm tra role nếu là admin
→ chạy nghiệp vụ
```

## 5. Luồng gameplay

```text
Scenario
→ Dialogue
→ Choice
→ Effects
→ Character State
→ History + consequence deltas
→ Contextual dialogue variants
→ Score/XP
→ Next Scenario
→ Save Progress
→ Result hoặc Final Summary
```

Ở summary, đồng hồ dùng `time.monotonic()` để tính số giây đã trôi qua. Khi
đạt 8 giây, game quay lại menu chính và không đóng ứng dụng.

## 6. Kiểm thử và chạy

```powershell
python run.py
python -m uvicorn server.main:app --reload
python -m pytest -q
```

Test hiện có kiểm tra API auth, endpoint bảo vệ, branching, effects và
localization.

## 7. Danh mục chức năng từng file trong dự án

Phần này ghi vai trò của từng file mã nguồn/cấu hình đang có. Các file
`__init__.py` không có nghiệp vụ riêng; chúng đánh dấu package Python. Những
package rỗng `client/screens/` và `client/services/` hiện là khung tổ chức,
không phải nơi đang triển khai màn hình hay API client: UI đang tập trung trong
`client/main.py`, API client trong `client/services/api_client.py`.

### File ở thư mục gốc

| File | Chức năng |
|---|---|
| `run.py` | Gọi `client.main.main()` để khởi chạy game desktop. |
| `requirements.txt` | Khai báo thư viện Python cho Pygame, FastAPI, MariaDB, xác thực, HTTP và test. |
| `.env.example` | Mẫu các biến cấu hình database, JWT, địa chỉ API và tên game; không chứa thông tin đăng nhập thật. |
| `.gitignore` | Loại trừ cấu hình bí mật, cache, save/database local và các file sinh khi chạy khỏi Git. |

### Frontend `client/`

| File | Chức năng |
|---|---|
| `client/main.py` | Lớp `OfficeLifeApp`: khởi tạo game, điều phối màn hình, vẽ nút/chữ/chỉ số, xử lý input, form đăng nhập/đăng ký, lưu game, thông báo, hồ sơ, leaderboard và vòng lặp Pygame. `draw_video_audio_controls()` giữ nút loa nhỏ ở góc clip và fade thanh âm lượng khi hover; `set_video_volume_from_mouse()` xử lý kéo thanh; `close()` dọn tài nguyên. Hàm `main()` là điểm vào ứng dụng. |
| `client/game/game_state.py` | `GameState`: lưu scenario hiện tại, bốn chỉ số, quan hệ nhân vật, lựa chọn, lịch sử/hậu quả, điểm, XP và thành tựu; áp dụng hiệu ứng, hoàn tất scenario và xác định ending. |
| `client/game/branching_engine.py` | `Choice`, `Scenario`, `BranchingEngine`: nạp catalog JSON, lấy hội thoại theo quan hệ, kiểm tra lựa chọn, áp dụng hiệu ứng và chuyển nhánh. |
| `client/managers/save_manager.py` | Serialize/deserialize `GameState`; đọc, ghi và đặt lại save khách local. |
| `client/managers/language_manager.py` | Tải locale, tra key lồng nhau, nội suy biến và chuyển đổi VI/EN. |
| `client/managers/audio_manager.py` | Tạo/phát âm báo lựa chọn, bật/tắt âm thanh và lưu tùy chọn; cho phép game tiếp tục nếu audio không khả dụng. |
| `client/managers/intro_video_manager.py` | Giải mã video trang chủ bằng FFmpeg ở chiều ngang tối đa 960 px; phát âm thanh gốc, phát trọn clip 26 giây một lần rồi seek về giây 11 để lặp đoạn chờ; điều chỉnh volume/mute, tạm dừng khi rời menu và dọn file âm thanh tạm khi đóng. Mốc chỉnh nằm ở `INTRO_END_SECONDS` và `LOOP_START_SECONDS`; đoạn cắt soundtrack nằm trong `_extract_audio()`. |
| `client/services/api_client.py` | Gửi yêu cầu HTTP tới FastAPI: đăng ký/đăng nhập/đăng xuất, profile, snapshot tiến độ, điểm, leaderboard, thành tựu và analytics; thêm token JWT cho endpoint cần xác thực. |
| `client/data/scenarios/content.json` | Dữ liệu tĩnh của 15 tình huống/5 chapter: hội thoại VI/EN, lựa chọn, hiệu ứng, phản ứng, phân tích, nhánh tiếp theo và biến thể hội thoại. |
| `client/localization/vi.json` | Chuỗi giao diện tiếng Việt, thông báo, menu, gameplay, account và cài đặt. |
| `client/localization/en.json` | Bản dịch tiếng Anh có cùng cấu trúc key với locale tiếng Việt. |
| `client/data/saves/progress.json` | Save khách được tạo trong lúc chạy; không cần có sẵn trong bản source. |
| `client/data/settings.json` | Tùy chọn cục bộ như âm thanh được tạo khi chạy; không cần có sẵn trong bản source. |
| `client/assets/**/.gitkeep` | Giữ các thư mục dự kiến cho background, nhân vật, font, icon, ảnh và âm thanh trong Git; hiện không phải asset game thực tế. |
| `client/assets/videos/office_life_intro.mp4` | Video intro cần người dùng thêm; phát một lượt từ 0–26 giây, sau đó lặp đoạn 11–26 giây. Đặt video có âm thanh tại đường dẫn cố định này. Nút loa và thanh chỉnh âm lượng xuất hiện ở góc dưới bên phải clip. |
| `client/__init__.py`, `client/game/__init__.py`, `client/managers/__init__.py`, `client/screens/__init__.py`, `client/services/__init__.py` | Đánh dấu các thư mục frontend là package Python; các file này hiện không chứa nghiệp vụ riêng. |

### Backend `server/`

| File | Chức năng |
|---|---|
| `server/main.py` | Tạo FastAPI app, khởi tạo/kết thúc tài nguyên database theo lifespan, gắn router, cung cấp `/health` và endpoint gốc. |
| `server/config.py` | Đọc và cung cấp cấu hình từ môi trường/`.env`, gồm URL database, secret JWT và thiết lập ứng dụng. |
| `server/database/db.py` | Chọn kết nối MariaDB hoặc SQLite theo `DATABASE_URL`, thực thi truy vấn, khởi tạo schema và migration account; adapter chuyển placeholder truy vấn theo dialect. |
| `server/database/mariadb_schema.sql` | Schema runtime MariaDB/InnoDB: bảng account, scenario/chapter, tiến độ, snapshot, điểm và thành tựu cùng index/foreign key. |
| `server/database/schema.sql` | Schema PostgreSQL tham khảo; không được runtime hiện tại sử dụng. |
| `server/auth/security.py` | Hash/verify mật khẩu và tạo/giải mã access token JWT. |
| `server/dependencies.py` | Đọc bearer token, xác thực người dùng hiện tại và chặn endpoint admin đối với role không phù hợp. |
| `server/schemas/api.py` | Pydantic schema cho dữ liệu request/response; chuẩn hóa và kiểm tra account, tiến độ, scenario, chapter, score và achievement. |
| `server/models/entities.py` | Các kiểu dữ liệu entity nội bộ cho user và scenario. |
| `server/services/user_service.py` | Hàm tiện ích hash/kiểm tra mật khẩu của service user; luồng auth chính dùng `server/auth/security.py`. |
| `server/routers/auth.py` | API đăng ký/đăng nhập, xem và cập nhật user hiện tại, liệt kê user cho admin. |
| `server/routers/content.py` | API đọc scenario/chapter và CRUD nội dung; thao tác ghi yêu cầu quyền admin. |
| `server/routers/game.py` | API snapshot/progress/score, leaderboard công khai và hạng cá nhân, achievements/claim và analytics. |
| `server/reset_password.py` | Công cụ terminal cục bộ đặt mật khẩu mới theo username/email; nhập mật khẩu ẩn và chỉ lưu hash. |
| `server/__init__.py`, `server/auth/__init__.py`, `server/database/__init__.py`, `server/models/__init__.py`, `server/routers/__init__.py`, `server/schemas/__init__.py`, `server/services/__init__.py` | Đánh dấu package backend Python; không chứa endpoint/nghiệp vụ riêng. |

### Kiểm thử `tests/`

| File | Chức năng |
|---|---|
| `tests/conftest.py` | Cấu hình fixture kiểm thử và dọn database SQLite tạm sau test. |
| `tests/test_api.py` | Kiểm tra health/đăng ký, quyền truy cập, schema/index/foreign key, MariaDB adapter/lỗi kết nối, migration account, hash mật khẩu, snapshot và API client. |
| `tests/test_game_engine.py` | Kiểm tra nhánh game, chỉ số/quan hệ, dialogue variants, localization, âm thanh, giao diện menu/account/leaderboard, form chọn/xóa text và hiện/ẩn mật khẩu. |
| `tests/test_project_structure.py` | Kiểm tra các thư mục bắt buộc, số chapter/lựa chọn trong catalog và tính nhất quán key localization. |
| `tests/__init__.py` | Đánh dấu thư mục test là package Python. |

### Tài liệu dự án

| File | Chức năng |
|---|---|
| `README.md` | Hướng dẫn cài đặt, cấu hình XAMPP, chạy game/backend, database, test và bản đồ mã nguồn. |
| `README_CODE.md` | Giải thích kiến trúc, các phần triển khai chính, luồng game/API và chức năng từng file. |
| `ARCHITECTURE.md` | Tóm tắt thành phần và ranh giới frontend/backend/database đang hoạt động. |
| `DATA_FLOW.md` | Mô tả luồng đăng nhập, gameplay, lưu save, leaderboard và hoạt động offline. |
| `PHASE1_ANALYSIS.md` | Báo cáo phân tích/phạm vi phase ban đầu của dự án. |
| `ADVANCED_PHASE1_ANALYSIS.md` | Phân tích và kế hoạch mở rộng Advanced Edition; không phải danh sách đảm bảo tính năng đã triển khai. |
