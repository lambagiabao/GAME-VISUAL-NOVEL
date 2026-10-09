# PHASE 1 - Phân tích source code hiện tại

## 1. Phạm vi Phase 1

Phase 1 chỉ đọc, kiểm tra và mô tả source code hiện có. Không viết lại
project, không tăng số lượng scenario và chưa triển khai các thay đổi thuộc
Phase 2 trở đi.

## 2. Cấu trúc project

```text
office_life/
├── run.py
├── client/
│   ├── main.py
│   ├── game/
│   │   ├── game_state.py
│   │   └── branching_engine.py
│   ├── managers/
│   │   ├── language_manager.py
│   │   └── save_manager.py
│   ├── services/
│   │   └── api_client.py
│   ├── localization/
│   │   ├── vi.json
│   │   └── en.json
│   └── data/scenarios/content.json
├── server/
│   ├── main.py
│   ├── auth/security.py
│   ├── database/db.py
│   ├── database/schema.sql
│   ├── dependencies.py
│   ├── routers/
│   │   ├── auth.py
│   │   ├── content.py
│   │   └── game.py
│   └── schemas/api.py
├── tests/
└── tài liệu .md và cấu hình
```

## 3. Main entry và game loop

`run.py` import `main()` từ `client.main` và khởi động game. `OfficeLifeApp`
trong `client/main.py` tạo cửa sổ Pygame 1280x720, clock 60 FPS, font
sans-serif, localization, save manager và branching engine.

Game loop hiện tại:

```text
pygame.event.get()
→ xử lý đóng cửa sổ, bàn phím, click button
→ gọi handle(action)
→ vẽ screen hiện tại
→ kiểm tra countdown summary
→ clock.tick(60)
```

Các screen đang được dispatch bởi `screen_name`:

- `menu`
- `game`
- `result`
- `library`
- `profile`
- `settings`
- `summary`

Mỗi button được tạo qua `button()` và lưu action thật trong `self.buttons`.
Click không gọi API giả hoặc function rỗng.

## 4. Scenario và branching

`client/data/scenarios/content.json` là nguồn dữ liệu scenario. Repository
hiện có 15 scenario, chia đúng 5 chapter, mỗi scenario có ít nhất 3 choice.

`BranchingEngine`:

1. Đọc và parse JSON thành `Scenario` và `Choice`.
2. Lấy scenario hiện tại từ `GameState.scenario_id`.
3. Kiểm tra `choice_id`.
4. Áp dụng score effect và character effect.
5. Ghi choice vào lịch sử.
6. Cộng XP và score.
7. Chuyển tới `next_scenario`.

Nhánh hiện tại đã khác nhau về `next_scenario`, effect, reaction và analysis.
Điểm cần mở rộng ở Phase 2 là điều kiện khóa/mở choice dựa trên state trước đó.

## 5. Score và Character State

`GameState` lưu:

- `empathy`
- `professionalism`
- `trust`
- `outcome`
- trust của `manager`, `colleague`, `customer`
- XP, score, completed scenario, choices và achievements

Mọi giá trị state được giới hạn từ 0 đến 100. `ending()` đã dùng nhiều điều
kiện, gồm skill stats và trust của colleague, thay vì chỉ cộng một tổng điểm.

## 6. Save system

`SaveManager` lưu `GameState` vào:

```text
client/data/saves/progress.json
```

Set được chuyển thành list khi ghi JSON và khôi phục lại thành set khi load.
Client tự load save lúc khởi động và save sau mỗi choice.

Giới hạn cần cải thiện ở phase Save/Load chuyên sâu: setting language và các
setting âm thanh chưa được đưa vào `GameState`; hiện tại ngôn ngữ là state của
phiên chạy.

## 7. Localization

`LanguageManager` đọc `vi.json` và `en.json`, hỗ trợ key dạng chấm và placeholder
như `{seconds}`. Scenario dialogue, choice, title, reaction và analysis đều có
hai ngôn ngữ. Hai file localization đã được kiểm tra parity key.

## 8. Backend/API và database

`server/main.py` tạo FastAPI application, CORS và các router `/api`.

Đã có:

- `/api/auth/register`
- `/api/auth/login`
- `/api/auth/me`
- `/api/auth/users`
- `/api/auth/users/me`
- scenario/chapter CRUD có admin guard
- progress
- scores
- achievements
- leaderboard
- analytics
- `/health`

`server/auth/security.py` dùng PBKDF2-SHA256 cho password và HMAC-SHA256
cho JWT. `server/dependencies.py` xác thực bearer token và role admin.

`server/database/db.py` dùng SQLite local để chạy ngay; `schema.sql` cung cấp
schema PostgreSQL tương ứng cho Supabase.

## 9. Chức năng đã hoàn thành

- Pygame client chạy desktop/offline.
- 5 chapter, 15 scenario.
- Branching dialogue và consequence effect.
- Bốn skill stats.
- Character state cơ bản.
- Post-scenario reaction/analysis.
- Multiple ending cơ bản.
- Profile, XP, score, summary.
- Localization Việt/English.
- Save/load local.
- FastAPI auth, role, content, progress, score, achievement, leaderboard,
  analytics.
- Test tự động cho API, branching, localization và Phase 1 structure.

## 10. Chức năng cần cải thiện theo các phase sau

### Phase 2 - Consequence và Character State

- Thêm `attitude` và `relationship level`.
- Cho state ảnh hưởng trực tiếp tới dialogue/choice bị khóa.
- Lưu snapshot effect của từng choice cho analysis.

### Phase 3 - Post-Scenario Analysis

- Hiển thị lựa chọn cụ thể và delta của từng chỉ số.
- Hiển thị consequence và gợi ý giao tiếp từ dữ liệu scenario.

### Phase 4 - Multiple Ending

- Đưa ending title/description/advice vào dữ liệu localization.
- Thêm replay và ghi nhận ending đã đạt.

### Phase 5 - Achievement

- Tạo catalog tối thiểu 10 achievement ở client và database.
- Tự động xét condition sau mỗi choice/chapter.

### Phase 6 - Profile và Analytics

- Bổ sung completed chapters, best score, replay count và average stats.
- Bổ sung admin analytics chi tiết hơn.

### Phase 7 - Bilingual

- Di chuyển các text UI còn hard-code trong `client/main.py` vào localization.

### Phase 8 - UI/UX, animation và audio

- Hover/click feedback, transition nhẹ và AudioManager không crash khi thiếu
  asset.

### Phase 9 - Save/Load và testing

- Lưu language/settings/ending.
- Bổ sung test click nhanh, replay, missing asset, reset progress và corrupted
  save.

## 11. Kiểm tra Phase 1

Đã chạy:

```powershell
python -m pytest -q
python -m compileall -q client server run.py
```

Kết quả hiện tại:

```text
7 passed
All Python files compile.
```

## 12. Kết luận

Source code hiện tại đã có nền tảng chạy được cho cả frontend và backend.
Không cần tạo lại project. Bước tiếp theo hợp lý là **PHASE 2 - Consequence +
Character State**, tập trung vào điều kiện state ảnh hưởng dialogue và choice
thay vì tăng số lượng scenario.
