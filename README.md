# Roblox Bearbrick 3D World

by thuc.hoang MACZT

> Game web đơn tệp (single-file HTML) kiểu Roblox Obby / leo thang, kết hợp bắn súng và bom champir, đồ họa 3D bằng Three.js, giao diện tiếng Việt. Chạy trực tiếp trên trình duyệt, không cần cài đặt.

---

## 🎮 Giới thiệu

**Roblox Bearbrick 3D World** là trò chơi webphinsh kiểu ô-bê (obby / climb) với nhân vật phong cách Bearbrick (khối gỗ/vitse verein bloc có nút bumpy trên bề mặt). Người chơi điều khiển nhân vật di chuyển, nhảy, bắn laser, ném bom champir, thu thập gem, leo thang tránh chướng ngại vật, và săn mục tiêu trong mê cung.

Game có nhiều chế độ chơi, nhiều bản đồ, hệ thống skin sưu tập (12 skin), cửa hàng skin, lưu tiến trình cục bộ (`localStorage`), và hỗ trợ cả máy tính (PC) lẫn thiết bị di động (touch).

---

## 🗺️ Chế độ chơi & Bản đồ

| Chế độ | Tên | Bản đồ | Mô tả |
|--------|-----|--------|-------|
| `climb` | Leo thang | Map 1: Tháp Không Gian (18 bậc) | Leo platform, tránh chướng ngại, thu thập đủ gem để hoàn thành |
| `climb` | Leo thang | Map 2: Pháo Đài Không Gian (20 bậc) | Tương tự Map 1, thêm ray trượt nghiêng 15° |
| `hunt` | Săn bia | Map 1: Mê Cung Cổ Điển | Bia đứng yên, bắn hạ số lượng mục tiêu quy định |
| `hunt` | Săn bia | Map 2: Mê Cung Chuyển Động | Bia di chuyển chậm, bố cục tường khác |
| `runner` | Chạy tự động | — | Chạy theo làn, đổi làn bằng A/D hoặc joystick, Space nhảy |
| `bomber` | Đặt bom | Map 1: Đảo Bomber; Map 2: Đảo Nước | Đặt bom, phá thùng, hạ quái và Boss; map 2 có nước chặn đường nhưng không chặn vụ nổ |

Mỗi bản đồ climb bắt đầu ở **độ khó 1⭐**. Thắng map sẽ lưu độ khó kế tiếp riêng cho map đó và thêm **4 tầng** vào đường leo, xen kẽ ray trượt và vòng xoay.

Mỗi map Bomber có tiến độ riêng và hai màn, mỗi màn có đồng hồ riêng: level 1 là **2 phút/màn**, mỗi level tiếp theo tăng **1 phút/màn**. Đồng hồ được đặt lại khi chuyển sang màn Boss. Người chơi bắt đầu với 2 bom đồng thời; bot không coi bom là vật cản.

Boss ở cả hai map đều có chiêu bắn tia laser. Chiêu còn lại ở map 1 nổ khói quanh Boss với bán kính 14,8 đơn vị; ở map 2, 12 ô đường trống không kề cạnh được chọn ngẫu nhiên, cảnh báo rồi phóng tia điện.

Khi bắt đầu mỗi lượt Bomber, 6 vật phẩm nâng cấp được đặt ngẫu nhiên trên các ô đường trống có thể tới được; phá thùng vẫn có thể nhận thêm vật phẩm.

Phá thùng có thể nhận vật phẩm **💪**: mỗi lần nhặt cộng 50% sát thương cơ bản của bom lên quái và Boss (36 → 54 → 72...). Nâng cấp giữ qua màn Boss, đặt lại khi bắt đầu lượt chơi mới và không tăng sát thương lên người chơi.

Thanh máu quái và Boss chia theo sát thương bom hiện tại và cập nhật ngay khi nhặt 💪. Các vạch tính từ máu tối đa; nếu không chia hết, phần nhỏ hơn nằm bên trái. Thanh Boss trên HUD cũng chia theo tổng máu tối đa.

Trong sảnh, mở **Skin & VIP → Vũ khí** để chọn miễn phí **Bom thường** hoặc **Máy chích điện** cho Bomber. Bom thường tạo khói; máy chích điện phóng tia sét, với cùng phạm vi và sát thương. Lựa chọn được lưu cùng tiến trình.

### Chỉnh bố cục map

Trong `index.html`, sửa `BOMBER_MAP_1`, `BOMBER_MAP_2` hoặc `HUNT_MAPS.map1` / `HUNT_MAPS.map2`: mỗi chuỗi là một hàng, `0` là đường đi và `1` là tường; riêng Bomber còn có `2` là nước chặn di chuyển nhưng không chặn vụ nổ. Có thể đặt cả tường và nước trong một map Bomber. Hàng đi từ bắc xuống nam, cột từ tây sang đông. Bomber có 11×11 ô bên trong viền nước, mỗi ô 4 đơn vị, dùng `0`/`1`/`2`; Săn bia có 49×49 ô bên trong tường viền, mỗi ô 2 đơn vị, chỉ dùng `0`/`1`.

Viền nước/tường, thùng Bomber, vị trí nhân vật, Boss, bia và các bục Săn bia được định nghĩa riêng, không nằm trong ma trận. Khi đổi tường, giữ các vị trí này thông thoáng, bảo đảm đường đi đủ rộng và các khu vực có thể tiếp cận; map Bomber cần ít nhất 6 ô trống có thể đến được ngoài vị trí người chơi và bot để đặt vật phẩm. Map 2 còn cần ít nhất 12 ô đường trống cùng màu trên bàn cờ ô xen kẽ để Boss chọn các ô nổ không kề cạnh, kể cả khi thùng chưa bị phá. Kiểm tra lại trong game.

---

## 👤 Skin & Cửa hàng

Có **12 skin** trong catalog:

| Skin | Tên | Loại | Giá (💎 Gem / 🎯 Mục tiêu) |
|------|-----|------|---------------------------|
| `chrome` | Classic Chrome | Mặc định (mở khóa) | 0 / 0 |
| `cyber` | Neon Cyber | Mặc định (mở khóa) | 0 / 0 |
| `gold` | Gold Edition | Standard | 10 / 15 |
| `pastel` | Pastel Panda | Standard | 15 / 20 |
| `lava` | Crimson Lava | Standard | 20 / 30 |
| `samurai` | Emerald Samurai | Standard | 28 / 40 |
| `galaxy` | Galaxy Void | Standard | 36 / 50 |
| `batman` | VIP Batman | VIP 3D | 36 / 50 |
| `thor` | VIP Thor | VIP 3D | 36 / 50 |
| `captain` | VIP Captain | VIP 3D | 36 / 50 |
| `spiderman` | VIP Spider-Man | VIP 3D | 36 / 50 |
| `maczt` | VIP Nhân Viên Maczt | VIP 3D | 0 / 0 |

Skin VIP cần đồng thời **36 💎 + 50 🎯** để mở khóa (trừ `maczt` mặc định mở). Skin mặc định: `chrome`, `cyber`, `maczt`.

---

## 🎹 Điều khiển

### Trên máy tính (PC)

| Hành động | Phím / Nút |
|-----------|------------|
| Tiến / Lùi | `W` / `S` (hoặc `↑` / `↓`) |
| Trái / Phải | `A` / `D` (hoặc `←` / `→`) |
| Nhảy | `Space` |
| Nhìn xung quanh | Chuột (khi Pointer Lock bật) |
| Bắn laser | `F` |
| Đổi loại đạn | `G` hoặc `Q` (`plasma` ↔ `lightning`) |
| Ném bom champir | Chuột phải giữ–ngắm, thả–ném |
| Đặt bom (bomber) | `E` hoặc `Space` |
| Developer Mode | Giữ `Ctrl` + `Shift`, nhấn `H` rồi `P` (trong 1,2 giây) |

### Trên thiết bị di động (Touch)

- Joystick ảo bên trái để di chuyển
- Kéo vùng ngoài joystick để xoay camera
- Nút `NHẢY`, `BẢN (F)`, `BOM` bên phải màn hình
- Nút bấm có hiệu ứng scale 0.92 khi nhấn

---

## 💾 Lưu tiến trình

- Dữ liệu lưu trong `localStorage` dưới key `roblox_bearbrick_3d_save_v4` (phiên bản 10)
- Tự động lưu (debounce) khi thay đổi tiến trình
- Có nút **Lưu ngay**, **Xuất JSON**, **Nhập JSON**, **Đặt lại dữ liệu**
- Lưu: theme nền, cấp độ/map đã đạt, skin đã mở, vũ khí săn bia và Boom, âm thanh, tổng gem/mục tiêu, completed maps

---

## 🛠️ Công nghệ

- **HTML5**, **CSS** (Tailwind CSS qua CDN), **JavaScript** (ES6 modules)
- **Three.js r160** (via unpkg CDN) cho đồ họa 3D
- **Web Audio API** tạo âm thanh thủ tục (không cần file âm thanh ngoài)
- **Google Fonts** (Inter)
- Chạy trực tiếp qua `file://`, không cần build tool hay máy chủ local

---

## 🚀 Cách chơi

1. Mở tệp `index.html` trên trình duyệt web hiện đại (Chrome, Firefox, Edge, Safari)
2. Đảm bảo kết nối internet để tải Tailwind CSS, Google Fonts, Three.js từ CDN
3. Mở **Chế độ** ở sảnh, chọn tab chế độ rồi chọn map để xem thông tin; nhấn **Vào game** để bắt đầu
4. Trong màn chơi, điều khiển nhân vật theo mục tiêu của từng chế độ
5. Nhấn **Thoát** để quay về màn hình chính (có hộp thoại xác nhận)

---

## ⚠️ Giới hạn

- Đây là bản chạy trên trình duyệt (web), **không phải** game Roblox thực sự sử dụng Lua/Roblox Studio
- Âm thanh được tạo thủ tục qua Web Audio API, cần tương tác người dùng đầu tiên (click/chạm) để khởi tạo `AudioContext`
- Dữ liệu lưu trữ cục bộ qua `localStorage`, có thể bị xóa nếu người dùng xóa dữ liệu duyệt web
- Một số tab trong cửa hàng sĩnh (phụ kiện, skin vũ khí) hiện là placeholder

---

*Tài liệu được sinh từ phân tích mã nguồn `index.html` của dự án Roblox_bearbrick.*
