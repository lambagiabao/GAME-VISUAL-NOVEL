# Advanced Edition - PHASE 1: Inspect và lập kế hoạch

## 1. Kết quả inspect

Repository hiện tại đã có source code chạy được, không cần tạo lại project.

### Frontend

- [run.py](run.py): entry point gọi `client.main.main`.
- `client/main.py`: Pygame window, event loop, menu, gameplay, result,
  library, profile, settings và summary.
- `client/game/game_state.py`: stats, character trust, XP, score, completed
  scenarios, choices và achievements.
- `client/game/branching_engine.py`: parse scenario JSON, xử lý choice effects,
  character effects và next scenario.
- `client/managers/language_manager.py`: tải VI/EN và format placeholder.
- `client/managers/save_manager.py`: save/load JSON local.
- `client/services/api_client.py`: HTTP client JWT/progress.
- `client/data/scenarios/content.json`: 15 scenario, 5 chapter, tối thiểu 3
  choice mỗi scenario.

### Backend

- `server/main.py`: FastAPI app, CORS, lifespan và routers.
- `server/routers/auth.py`: register, login, profile, admin user list.
- `server/routers/content.py`: scenario/chapter CRUD có admin guard.
- `server/routers/game.py`: progress, score, achievement, leaderboard,
  analytics.
- `server/auth/security.py`: PBKDF2 password hashing và HMAC JWT.
- `server/dependencies.py`: current user và role admin.
- `server/database/db.py`: SQLite local schema.
- `server/database/schema.sql`: PostgreSQL/Supabase schema.
- `server/schemas/api.py`: Pydantic validation.

### Kiểm thử hiện tại

- API health, register, protected endpoint.
- Branching và effects.
- Localization.
- Phase 1 structure, scenario count và localization key parity.

## 2. Chức năng đã có

- 5 chapter, 15 scenario.
- Branching dialogue và score consequence.
- Bốn stats: empathy, professionalism, trust, outcome.
- Character trust cơ bản.
- Scenario result/reaction/analysis.
- Multiple ending cơ bản.
- XP, score, profile và final summary.
- Save/load local.
- VI/EN cho dữ liệu game và UI chính.
- FastAPI authentication, authorization và game APIs.
- Countdown final summary, quay về main menu và replay từ đầu.

## 3. Khoảng trống so với Advanced Edition

1. Communication Coach chưa có model nhận xét riêng gồm đánh giá, điểm mạnh,
   điểm cần cải thiện, lời khuyên và tác động chỉ số.
2. Decision History mới chỉ lưu choice tối giản, chưa lưu timestamp, delta,
   character change và consequence chi tiết.
3. Replay chưa có snapshot riêng và màn hình compare.
4. Profile chưa có radar chart và Communication Skill Level.
5. Unexpected Events chưa có data model và điều kiện kích hoạt.
6. Final Assessment và Decision Making chưa có.
7. Certificate chưa có.
8. Achievement/analytics chưa tích hợp đầy đủ với gameplay local.
9. UI còn tập trung trong một file lớn; animation/audio/tutorial chưa hoàn chỉnh.
10. Save schema chưa lưu language, settings, replay, assessment và certificate.

## 4. Kế hoạch triển khai theo phase

### PHASE 2 - Communication Coach + Decision History

Tạo các model/helper dùng lại trong game engine:

- `CoachFeedback`
- `DecisionRecord`
- delta stats và delta character state
- timestamp và consequence

Mở rộng result screen và thêm Decision History screen. Dữ liệu phải lấy từ
choice thực tế, không tạo dữ liệu giả.

### PHASE 3 - Replay & Compare

Thêm snapshot lượt chơi chính/replay, replay scenario đã hoàn thành và compare
cards. Replay không cộng trùng XP/achievement và không ghi đè save chính.

### PHASE 4 - Skill Radar + Communication Skill Level

Tự vẽ radar bằng `pygame.draw`, tính level theo ngưỡng độc lập của bốn stats và
tiến độ scenario.

### PHASE 5 - Unexpected Events

Thêm event data có điều kiện rõ ràng, choice/effects riêng và lưu event progress.
Ưu tiên trigger xác định theo choice/state thay vì random không kiểm soát.

### PHASE 6 - Final Assessment

Sau khi hoàn thành 5 chapter, mở assessment 5-7 quyết định, có Decision Making
riêng và báo cáo tổng hợp.

### PHASE 7 - Certificate

Cấp certificate chỉ sau khi assessment hợp lệ; lưu mã, ngày, score, skill level
và hỗ trợ VI/EN. Ưu tiên hiển thị/in ảnh trước khi thêm thư viện PDF.

### PHASE 8 - Achievement + Analytics

Tích hợp unlock condition thật cho achievement mới, chống claim trùng và đồng bộ
analytics client/backend.

### PHASE 9 - UI/UX + Bilingual

Tách screen khi cần, thêm hover/click/fade nhẹ, tutorial và audio manager
không crash khi thiếu asset. Di chuyển hard-code UI còn sót vào localization.

### PHASE 10 - Save/Load + Regression

Version hóa save schema, migration an toàn, kiểm thử replay, corrupted save,
missing asset, đổi ngôn ngữ, reset và login/logout.

## 5. Nguyên tắc chuyển phase

Mỗi phase phải:

1. Sửa trực tiếp source hiện tại.
2. Không tăng số lượng 15 scenario.
3. Không tạo module trùng chức năng.
4. Có test logic mới và regression test.
5. Chạy `python -m pytest -q` và compile source.
6. Cập nhật tài liệu sau khi behavior đã chạy ổn định.

## 6. Trạng thái Phase 1

Phase 1 đã hoàn tất: repository đã được inspect, các chức năng hiện tại và
khoảng trống đã được xác định, kế hoạch Advanced Edition đã được ghi lại.
Phase 2 là bước tiếp theo và chưa được triển khai trong phase này.
