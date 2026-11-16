# RULEGAME.MD - Roblox Bearbrick 3D World

Tài liệu mô tả toàn bộ cơ chế gameplay, cơ chế code, chi tiết từng nút bấm và style game của dự án `Roblox_bearbrick`.

---

## 1. GIOI THIEU GAME

- **Tên game**: Roblox Bearbrick 3D World
- **Tác giả**: thuc.hoang MACZT
- **Loại**: Game web đơn tệp (single-file HTML), đồ họa 3D, kiểu obby / leo thang (climb) kết hợp bắn súng và bom champir. Có 4 chế độ: climb, hunt, runner, bomber.
- **Ngôn ngữ / công nghệ**: HTML5, CSS (Tailwind CSS qua CDN), JavaScript (ES6 modules), Three.js r160 (via unpkg CDN), Web Audio API
- **Màn hình**: Chạy trực tiếp trên trình duyệt, hỗ trợ cả máy tính (PC) và thiết bị di động (touch)
- **Ngôn ngữ giao diện**: Tiếng Việt

---

## 2. STYLE GAME (NGU VI & THIET KE)

### 2.1 Phong thẩm mỹ (Visual Style)

| Thành phần | Chi tiết |
|------------|----------|
| **Màu nền chính** | `#090d16` (den muy sâm), fog màu `#132043` |
| **Cảnh backdrop** | Không gian vũ trụ với 1400 sao hạt (star particles), màu `#e0f2fe` |
| **Kiểu khối** | Phong cách Roblox Stud: khối có nút bumpy trên bề mặt, kết hợp cỏ (grass), champir (shampoo), văn phong Bearbrick (mảnh đồ chơi gỗ/vitse verein bloc) |
| **Chất liệu vật liệu** | `MeshStandardMaterial` với metalness/roughness cao, bóng mềm (PCFSoftShadowMap), tone mapping `ACESFilmicToneMapping`, exposure 1.12 |
| **Giao diện (UI)** | Glassmorphism: nền mờ `rgba(15, 23, 42, 0.78)`, backdrop-filter blur 14px, viền trắng mờ, bóng Drop shadow |
| **Phông chữ** | Inter (Google Fonts), trọng số 400/600/700/800/900 |
| **Nút bấm (KBD style)** | Lớp `.kbd`: màu trắng mờ, border dưới dày, radius 6px, font-weight 700 |
| **Nút chạm di động (touch-btn)** | Hiệu ứng scale 0.92 khi nhấn, transition transform 0.08s |

### 2.2 Âm thanh (Sound Engine)

Sử dụng **Web Audio API** tạo âm thanh thủ tục (procedural synthesis), không cần tệp âm thanh ngoài:

| Hàm âm thanh | Mô tả |
|-------------|-------|
| `playLaser(isTargetKill)` | Tiếng nổ laser, tần số rơi dần, nếu trúng mục tiêu có tiếng chuông square wave 440→660 Hz |
| `playStarCollect()` | 4 nốt triangle wave ascending (587.33, 739.99, 880, 1174.66 Hz) |
| `playWarningBeep()` | Tiếng bip square wave 920 Hz |
| `playDamage()` | Tiếng sawtooth rơi từ 190 Hz xuống 55 Hz |
| `playVictory()` | Tiếng chiến thắng |

---

## 3. CO CHE TRONG GAME (GAMEPLAY MECHANICS)

### 3.1 Man hinh & luong chay

Game có 2 màn hình chính:
1. **`menu`** - Màn hình chính: hiển thị showcase Bearbrick 3D, chọn skin, chọn chế độ chơi, cài đặt âm thanh, lưu/nạp dữ liệu
2. **`playing`** - Màn chơi: nhân vật xuất hiện trên bản đồ, điều khiển di chuyển, bắn, ném bom, thu thập gem, leo thang

Chuyển màn hình qua hàm:
- `enterMainMenu()` - quay về menu
- `startGameMode(mode, mapId)` - bắt đầu chơi (mặc định mode = `'climb'`, mapId = `'map1'`)

Mỗi lần vào màn chơi, chữ **START** xuất hiện giữa màn hình rồi mờ dần trong 0,5 giây.

### 3.2 Che do choi (Game Mode)

Hiện có **4 chế độ chính**:
- **`climb`** (Leo thang): Leo platform, tránh chướng ngại vật và thu thập gem trên hai map.
- **`hunt`** (Săn bia): Bắn và ném bom vào bia trong mê cung.
- **`runner`** (Chạy tự động): Nhân vật chạy tự động về phía trước, người chơi đổi làn (A/D hoặc joystick) và nhảy (Space) để tránh chướng ngại vật.
- **`bomber`** (Đặt bom): Di chuyển theo tâm lối với tốc độ mặc định 9.6, bắt đầu đặt được 2 bom đồng thời, nổ dài 2 ô theo bốn hướng; phá thùng để tìm nâng cấp, gồm vật phẩm hồi 20% HP tối đa và khiên dạng vành cầu xoay quanh thân. Level 1 có 20 bot (36 HP mỗi bot) và Boss 300 HP. Mỗi chiến thắng tăng 1⭐ riêng cho map; mỗi level thêm 2 bot và 18 HP/bot, tăng 100 HP/Boss. Mỗi màn có đồng hồ riêng: level 1 là 2 phút, mỗi level sau thêm 1 phút, và đồng hồ đặt lại khi vào màn Boss. Từ Level 4 trở đi có 2 Boss ở hai góc đối diện, ra chiêu lệch nhịp. Bot đuổi theo nhân vật và không coi bom là vật cản.

Khi hoàn thành mục tiêu, Bearbrick nhảy mừng, chữ **VICTORY** bay lên cùng hiệu ứng pháo hoa trong 3 giây rồi mới hiện bảng kết quả. Khi thua ở Runner hoặc Bomber, chữ **LOSE** bay lên 2 giây trước bảng chơi lại/thoát; các thời điểm thắng/thua có âm báo riêng và Bomber có nhạc nền riêng.

Bản đồ:
- **Climb Map 1:** Tháp Không Gian, 18 tầng.
- **Climb Map 2:** Pháo Đài Không Gian, 20 tầng.
- **Hunt Map 1:** Mê Cung Cổ Điển; bia đứng yên.
- **Hunt Map 2:** Mê Cung Chuyển Động; bố cục tường khác Map 1, có lối đi vào các khu vực và bia di chuyển chậm quanh vị trí spawn.
- **Runner World:** Địa hình chạy tự động theo làn.
- **Bomber Map 1:** Bản đồ đặt bom, phá thùng tìm nâng cấp; viền sông liền mạch có sóng chuyển động và không thể đi qua, tường đá bên trong được hạ thấp, thùng nước nhiều chi tiết và bụi cỏ dạng khối đung đưa.
- **Bomber Map 2:** Cùng bố cục ô như map 1, nhưng thay tường đá bằng vũng nước không thể đi qua; vùng nổ vẫn xuyên qua nước.
- **Vật phẩm đầu lượt:** 6 vật phẩm nâng cấp ngẫu nhiên xuất hiện trên các ô đường trống có thể tới được, ngoài vật phẩm có thể rơi từ thùng.
- **Vật phẩm 💪:** Mỗi lần nhặt cộng 50% sát thương cơ bản (18 HP) lên bot và Boss cho cả bom thường lẫn máy chích điện. Nâng cấp giữ qua màn Boss, đặt lại khi bắt đầu lượt mới và không tăng sát thương tự gây cho người chơi.

Các map Bomber được khai báo bằng ma trận `0` (đường) / `1` (tường đá) / `2` (nước), còn Hunt dùng `0` (đường) / `1` (tường) trong `index.html`; xem phần chỉnh bố cục map trong README để biết kích thước và các vị trí cần giữ thông thoáng. Shop vũ khí ở sảnh có bom thường và máy chích điện miễn phí cho Bomber, cùng sát thương và tầm nổ nhưng hiệu ứng khác nhau.

Mỗi climb map bắt đầu ở **độ khó 1⭐**. Thắng map sẽ lưu độ khó kế tiếp riêng cho map đó và thêm **4 tầng** vào đường leo, xen kẽ ray trượt và vòng xoay. Các tầng mở rộng được bố trí theo quy tắc vòng lục giác bán kính 9.5 đơn vị, mỗi bước lệch 60° (không random); ô vòng xoay rộng 7.5 đơn vị để có thêm chỗ đứng. Không có giới hạn level được đặt trong code. Ở Map 2, vòng xoay 1–2 có 2 cánh, vòng 3–4 có 3 cánh; ray trượt nghiêng 15° và đẩy Bearbrick trượt chậm xuống dốc.

Mỗi hunt map có level riêng, lưu trong `state.huntMapLevels`. Map bắt đầu ở cấp 1 với 15 bia; mỗi lần thắng, cấp của map đó tăng 1 và mục tiêu tăng thêm 5 bia (`15 + 5 × (level - 1)`).

Để dễ tiếp cận hơn, Climb Map 1 giảm tốc độ các vòng xoay còn 20% mức cũ; vòng 1–2 có 2 cánh, vòng 3–4 có 3 cánh (vòng 5 giữ 4 cánh). Tốc độ bia tuần tra ở Hunt Map 2 tăng 20% so với cấu hình ban đầu.

Kho thành tích ở sảnh hiển thị tổng độ khó ⭐ của cả bốn map; các map không còn badge hoàn thành vì có thể nâng độ khó vô hạn.

### 3.3 Nhan vat & State

**Trạng thái người chơi (`state`)** bao gồm:
- `pos` (THREE.Vector3): Vị trí hiện tại
- `hp`: Máu (health points), tối đa 100
- `score`: Điểm số hiện tại
- `totalStarsAllTime`: Tổng gem đã thu thập suốt đời
- `totalTargetsAllTime`: Tổng mục tiêu bắn trúng
- `weaponMode`: `'plasma'` hoặc `'lightning'`
- `skin`: Key skin hiện tại (mặc định `'chrome'`)
- `unlockedSkins`: Danh sách skin đã mở khóa
- `keys`: `{w, a, s, d}` - trạng thái phím bàn phím
- `joyVec`: `{x, y}` - vector joystick ảo di động
- `matchEnded`: Boolean, trận đấu đã kết thúc chưa
- `isDragging`: Boolean, đang kéo chuột trên menu (xoay Bearbrick showcase)

### 3.4 Vat ly & Di chuyen

- **Vật lý đơn giản (AABB)**: Kiểm tra va chạm giữa hộp người chơi và các khối bản đồ
- **Regen máu**: Nếu đứng trong `healingPool` (hồ nước shampoo), hồi +5 HP/s
- **Respawn**: Khi máu về 0 hoặc nhấn nút Reset, nhân vật quay lại vị trí spawn
- **Giới hạn bản đồ**: `MAP_BOUNDS = { halfSize: 50 }` (vùng chơi vuông 100×100)
- **Camera**: PerspectiveCamera 60° theo nhân vật, có cuộn chuột để zoom, kéo chuột để xoay trên menu. Bomber dùng camera orthographic nghiêng 42°, tự fit toàn bộ map 52×52 theo tỷ lệ viewport và giữ kích thước nhân vật ổn định theo chiều sâu.
- **Pointer Lock**: Bấm `Shift` hoặc nút "Shift-Lock" để khóa chuột, nhìn tự do như FPS
- Trong chế độ Săn Bia, độ nhạy camera theo trục dọc đã giảm khoảng 50%; tường chắn đường nhìn giữa camera và nhân vật sẽ mờ đi để vẫn nhìn thấy Bearbrick.

### 3.5 Vu khi & Ban

| Vũ khí | Mô tả |
|--------|-------|
| **Plasma** | Bắn tia laser plasma, hình ảnh tiếng nổ laser |
| **Lightning** | Bắn tia sét, hiệu ứng tần số khác |

**Cơ chế bắn:**
- Nhấn `F` (PC) hoặc nút **🔥 Fire** (di động) -> `shootLaser()`
- Nhấn `G` hoặc `Q` để đổi giữa Plasma và Lightning
- Tia bắn Raycaster từ camera hướng về phía trước
- Nếu trúng mục tiêu: phát âm thanh `playLaser(true)`, tăng `totalTargetsAllTime`, hiển thị toast
- Nếu không trúng: phát `playLaser(false)`

### 3.6 Bom Champir (Shampoo Bomb)

- Giữ chuột phải hoặc nút **💣 Bomb** (di động) để định hướng ném (`state.isAimingBomb = true`)
- Thả chuột phải / nhả nút -> `throwShampooBomb()` ném bom theo hướng Camera
- Bom tạo vụ nổ vùng (AOE), gây sát thương hoặc hiệu ứng theo thiết kế
- Phát âm thanh tương ứng khi ném

### 3.7 Vat pham & Thu thuap

| Vật phẩm | Hình dạng | Hiệu ứng |
|----------|-----------|----------|
| **Gem** | OctahedronGeometry (bức 8 mặt), kích thước 0.7 | +1 điểm score, +1 `totalStarsAllTime`, phát `playStarCollect()`, lưu tự động |
| **Healing Pool** | Hồ nước shampoo tròn | Hồi +5 HP/s khi nhân vật đứng trong |

### 3.8 Chuong ngai vat

| Chướng ngại vật | Hàm tạo | Mô tả |
|-----------------|---------|-------|
| **Rolling Rail** (Đường ray lăn) | `addRollingRailPlatform()` | Nền platform lăn chuyển động theo trục xác định, có phạm vi và tốc độ |
| **4-Blade Sweeper** (Vòng quay 4 thanh) | `addFourBladeSweeper()` | Cục quay 4 thanh đẩy, quay liên tục, làm mất máu nếu va chạm |
| **Platform Block** | `addPlatformBlock()` | Khối nền cơ bản, có nút Stud (Roblox stud), màu sắc tùy chỉnh |

---

## 4. CO CHE CODE (CODE MECHANICS / ARCHITECTURE)

### 4.1 Cau truc file

```
index.html
├── <head>
│   ├── Tailwind CSS CDN
│   ├── Google Fonts (Inter)
│   ├── CSS tùy chỉnh (glass-panel, kbd, touch-btn, responsive)
│   └── Importmap cho Three.js r160
├── <body>
│   ├── #canvas-container (Viewport Three.js)
│   ├── Overlay Menu (main menu, skin shop, settings)
│   ├── Overlay HUD (in-game: HP, score, ammo, buttons)
│   └── Overlay Result (kết thúc trận)
└── <script>
    ├── SoundManager (Web Audio API procedural)
    ├── Three.js Scene / Camera / Renderer
    ├── Texture generators (procedural canvas textures)
    ├── Mesh builders (stud, grass, shampoo label, VIP logo)
    ├── World builders (menu showcase, climb maps)
    ├── Player / Bearbrick avatar builder (skins)
    ├── Game state object
    ├── Input handlers (keyboard, touch, joystick, pointer lock)
    ├── Combat (shootLaser, throwShampooBomb, applyDamage)
    ├── Physics (updatePhysics, AABB collision, healing regen)
    ├── Obstacle updates (updateDynamicObstacles)
    ├── Collectibles (updateCollectibles)
    ├── Minimap renderer (renderMiniMap)
    ├── SaveManager (localStorage, JSON export/import)
    └── Animation loop (animate -> requestAnimationFrame)
```

### 4.2 Vòng lặp chính (Main Loop)

```javascript
function animate() {
    requestAnimationFrame(animate);          // Lặp ~60 FPS
    const dt = Math.min(clock.getDelta(), 0.05);  // Delta time, giới hạn tối đa 50ms
    const elapsedTime = clock.getElapsedTime();
    
    // Cập nhật chướng ngại vật, nước hồ, vật phẩm thu thập
    updateDynamicObstacles(dt, elapsedTime);
    updateCollectibles(elapsedTime);
    
    // Cập nhật vật lý người chơi (di chuyển, va chạm, máu)
    updatePhysics(dt);
    
    // Cập nhật camera
    updateCamera();
    
    // Vẽ minimap
    renderMiniMap(elapsedTime);
    
    // Render cảnh Three.js
    renderer.render(scene, camera);
}
```

### 4.3 He thong Texture thu tuc (Procedural Textures)

Tất cả texture được vẽ trên `<canvas>` 2D rồi chuyển thành `THREE.CanvasTexture`:

| Hàm | Kích th.canvas | Mô tả |
|-----|---------------|-------|
| `createRobloxStudTexture()` | 128×128 | Texture nút stud kiểu Roblox |
| `createGrassStudTexture()` | 128×128 | Texture cỏ có stud |
| `createShampooLabelTexture()` | 128×128 | Nhãn champir |
| `createVipLogoTexture(type)` | 256×256 | Logo biểu tượng nhân vật VIP / biển 표시 |

### 4.4 He thong Skin (Bearbrick Avatar)

Xây dựng nhân vật Bearbrick dạng nhóm `THREE.Group` với các bộ phận: head, body, eyes, chest plate, glasses, visor, headset...

| Skin Key | Tên hiển thị | Loại | Giá (💎 / 🎯) | Mô tả |
|----------|-------------|------|--------------|-------|
| `chrome` | Classic Chrome | Mặc định (mở khóa) | 0 / 0 | Màu crom kim loại cơ bản |
| `cyber` | Neon Cyber | Mặc định (mở khóa) | 0 / 0 | Màu neon cyberpunk |
| `gold` | Gold Edition | Standard | 10 / 15 | Màu vàng kim loại |
| `pastel` | Pastel Panda | Standard | 15 / 20 | Màu pastel hồng trắng |
| `lava` | Crimson Lava | Standard | 20 / 30 | Màu đỏ lava |
| `samurai` | Emerald Samurai | Standard | 28 / 40 | Màu xanh ngọc samurai |
| `galaxy` | Galaxy Void | Standard | 36 / 50 | Màu vũ trụ của không gian |
| `batman` | VIP Batman | VIP 3D | 36 / 50 | Áo balo Batman, mặt nạ, khoá hiệu |
| `thor` | VIP Thor | VIP 3D | 36 / 50 | Áo Thor, sấm sét |
| `captain` | VIP Captain | VIP 3D | 36 / 50 | Áo Captain America |
| `spiderman` | VIP Spider-Man | VIP 3D | 36 / 50 | Áo khoác lưới nhện, mắt trắng |
| `maczt` | VIP Nhân Viên Maczt | VIP 3D | 0 / 0 | Kính tech visor màu xanh, headsetMic, áo đồng phục MACZT |

Mỗi skin có:
- `gemCost`: Giá mua bằng gem
- `targetCost`: Giá mua bằng số mục tiêu bắn trúng
- `shortName`: Tên viết tắt hiển thị trên nút skin

Mở khóa skin: Hàm `applySkinSelection(skinKey)` thay đổi `state.skin`, rebuild mô hình Bearbrick, phát `playVictory()`, lưu tự động.

### 4.5 He thong Luu du lieu (SaveManager)

- **Tự động lưu**: `saveManager.scheduleSave()` debounce lưu vào `localStorage`
- **Nút Lưu (Lưu ngay)**: `btn-save-local` -> `saveManager.saveNow(false)`
- **Đặt lại**: `menu-btn-reset` -> `saveManager.resetAllData()` (xóa toàn bộ tiến trình)

---

## 5. CHI TIET TUNG NUT BAM (BUTTONS & CONTROLS)

### 5.1 Menu chinh (Main Menu)

| Nút / Phím | ID / Mã | Chức năng |
|------------|---------|-----------|
| Âm thanh | `menu-btn-audio` | Bật/tắt âm thanh (hieu luc icon 🔊/🔇) |
| Xuất JSON | `menu-btn-export` | Tải dữ liệu lưu về tệp JSON |
| Nhập JSON | `menu-btn-import` | Chọn tệp JSON để nhập dữ liệu |
| Đặt lại dữ liệu | `menu-btn-reset` | Xóa toàn bộ tiến trình lưu (gem, skin, điểm) |
| Bắt đầu Climb Map 1 | `btn-start-climb-1` | Chơi Tháp Không Gian |
| Bắt đầu Climb Map 2 | `btn-start-climb-2` | Chơi Pháo Đài Không Gian |
| Bắt đầu Hunt Map 1 | `btn-start-hunt-map1` | Săn bia đứng yên ở mê cung cổ điển |
| Bắt đầu Hunt Map 2 | `btn-start-hunt-map2` | Săn bia di chuyển chậm ở mê cung mới |

### 5.2 Man choi / HUD (In-Game)

| Nút / Phím | ID / Mã | Chức năng |
|------------|---------|-----------|
| Bắn laser / Fire | `btn-fire` (PC: `F`) | `shootLaser()` - bắn tia plasma/lightning |
| Ném bom champir | `btn-bomb` / chuột phải | Giữ để ngắm, thả để ném bom shampoo |
| Đổi loại đạn | `btn-switch-ammo` (PC: `G` hoặc `Q`) | Chuyển đổi giữa `plasma` ↔ `lightning` |
| Nhảy (di động) | `touch-jump` | Lệnh nhảy trên thiết bị cảm ứng |
| Reset nhân vật | `btn-reset` | `respawnPlayer()` - hồi sinh tại điểm xuất phát |
| Khóa chuột (Pointer Lock) | `btn-pointer-lock` | Bật/tắt khóa con trỏ chuột (Shift-Lock); Bomber không tự khóa và ẩn nút này |
| Developer Mode | Giữ `Ctrl` + `Shift`, nhấn `H` rồi `P` trong vòng 1,2 giây | Bật/tắt không mất máu và không bị vòng xoay đánh văng; trong climb, `ArrowUp` tới bậc kế tiếp; trong hunt, `ArrowUp` hạ bia sắp nổ nhất |
| MởDrawer Avatar | `btn-avatar-toggle` | Mở/đóng bảng thông tin nhân vật và skin |
| Lưu game (trong chơi) | `btn-save-local` | Lưu tiến trình ngay lập tức |
| Xuất JSON (trong chơi) | `btn-export-json` | Tải file JSON |
| Replay (màn hình kết quả) | `btn-result-replay` | Chơi lại chế độ/bản đồ hiện tại |
| Đặt bom (bomber) | `btn-place-bomber-bomb` | Đặt bom trong chế độ bomber |

Trong Bomber, bộ đếm giờ kèm thanh tiến trình và thanh máu Boss hiển thị trong bảng thông tin phía trên; nhân vật và bot có thanh máu phía trên đầu, đồng thời có ô vuông cam nhạt mờ 40% dưới chân nhân vật để chỉ vị trí. Vụ nổ hiển thị sát thương và trúng khi ít nhất 1/3 diện tích thân nhân vật hoặc bot giao với ô lửa. Nửa giây trước khi nổ, các ô trong vùng ảnh hưởng nhấp nháy; khói tỏa nhanh theo bốn hướng. Bot có vệt chém khi đánh thường và lắc lư khi bước; nếu không tìm được đường tới nhân vật, bot lùi khoảng 3 ô rồi tìm lại. Nhân vật chớp sáng khi trúng bom, còn bot, Boss và nhân vật lắc lư rồi mờ dần khi bị hạ. Ban đầu nhân vật và 4 bot mờ dần xuất hiện trong 0,5 giây; 4 bot còn lại xuất hiện tuần tự trong 0,5 giây tiếp theo.

### 5.3 Dieu khien di chuyen (Movement)

| Bộ điều khiển | PC (Bàn phím) | Di động (Touch) |
|--------------|---------------|-----------------|
| Tiến / Lùi | `W` / `S` (hoặc `ArrowUp` / `ArrowDown`) | Joystick ảo bên trái (drag nút joystick-knob) |
| Trái / Phải | `A` / `D` (hoặc `ArrowLeft` / `ArrowRight`) | Joystick ảo bên trái |
| Nhảy | (Xử lý trong `updatePhysics`) | Nút `touch-jump` |
| Nhìn xung quanh | Chuột (khi Pointer Lock bật; trục dọc giảm 50%) | Kéo khu vực ngoài joystick (camera touch) |
| Bắn | `F` | Nút `btn-fire` (vùng bên phải màn hình) |
| Đặt bom (bomber) | `E` / `Space` | Nút `btn-place-bomber-bomb` (chỉ ở chế độ bomber) |

Tốc độ xoay theo hướng di chuyển tăng 4 lần. Trong Bomber, khi ít nhất 3/4 bề ngang nhân vật đã vào làn, hệ thống căn dần về tâm làn để tránh mắc ở góc tường.

### 5.4 Joystick ảo (Mobile)

- Vùng: `joystick-zone` (trái dưới màn hình, chỉ hiển thị khi `max-width: 768px` và độ dọc)
- Nút: `joystick-knob` (gradient cyan→trắng, bóng effectively)
- Khi chạm vào joystick: ghi nhận `joyTouchId`, tính vector chuẩn hóa `state.joyVec = {x, y}` trong khoảng [-1, 1]
- Khi nhả: reset `joyVec` về `{0, 0}`, knob về vị trí trung tâm
- Được xử lý bởi `updateJoystickFromTouch(touch)`, `updatePhysics(dt)` đọc `state.joyVec`
- **Xuất JSON**: `btn-export-json` / `menu-btn-export` -> tải tệp `.json`
- **Nhập JSON**: `menu-btn-import` -> chọn tệp JSON -> `saveManager.importFromJsonFile(file)`
- **Đặt lại**: `menu-btn-reset` -> `saveManager.resetAllData()` (xóa toàn bộ tiến trình)

---

## 6. CHI TIET CAC HAM CODE CHINH (KEY FUNCTIONS)

Dưới đây là danh sách các hàm quan trọng trong `index.html` (theo thứ tự xuất hiện):

| Vùng | Hàm | Dòng (khoảng) | Mô tả |
|------|-----|---------------|-------|
| Âm thanh | `playLaser(isTargetKill)` | ~660 | Phát âm thanh bắn laser |
| Âm thanh | `playStarCollect()` | ~694 | Phát âm thanh thu thập gem |
| Âm thanh | `playWarningBeep()` | ~715 | Phát âm thanh cảnh báo |
| Âm thanh | `playDamage()` | ~732 | Phát âm thanh bị thương |
| Âm thanh | `playVictory()` | ~750 | Phát âm thanh thắng |
| Cảnh | `createSpaceBackdrop()` | ~793 | Tạo backdrop sao vũ trụ |
| Texture | `createRobloxStudTexture()` | ~852 | Tạo texture stud kiểu Roblox |
| Texture | `createGrassStudTexture()` | ~882 | Tạo texture cỏ |
| Texture | `createShampooLabelTexture()` | ~918 | Tạo nhãn champir |
| Texture | `createVipLogoTexture(type)` | ~941 | Tạo texture logo VIP |
| Mesh | `createShampooBottleMesh()` | ~1083 | Tạo lọ champir 3D |
| World | `clearWorld()` | ~1846 | Xóa toàn bộ đối tượng bản đồ cũ |
| World | `addPlatformBlock(...)` | ~1870 | Thêm khối nền có stud |
| World | `addHealingWaterPool()` | ~1913 | Thêm hồ máu hồi +5 HP/s |
| Obstacle | `addRollingRailPlatform()` | ~2001 | Thêm platform lăn |
| Obstacle | `addFourBladeSweeper()` | ~2075 | Thêm vòng quay 4 thanh |
| Collectible | `spawnCollectibleGem()` | ~2131 | Tạo gem octahedron |
| Menu | `buildMenuShowcaseWorld()` | ~2156 | Xây dựng cảnh showcase menu |
| Map | `buildClimbMap1()` | ~2192 | Xây dựng bản đồ Rainbow Sky Tower (18 bậc) |
| UI | `enterMainMenu()` | ~3517 | Quay về màn hình chính |
| UI | `setCombatUIVisible(visible)` | ~3540 | Hiển thị/ẩn HUD chiến đấu |
| Game | `startGameMode(mode, mapId)` | ~3551 | Bắt đầu chơi |
| Combat | `setWeaponMode(mode)` | ~3896 | Chuyển plasma / lightning |
| Skin | `applySkinSelection(skinKey)` | ~3951 | Áp dụng skin Bearbrick |
| Physics | `updateDynamicObstacles(dt, elapsedTime)` | ~4016 | Cập nhật chướng ngại động, hồ máu |
| Physics | `updatePhysics(dt)` | ~4083 | Vật lý nhân vật, va chạm, máu |
| Camera | `updateCamera()` | ~4316 | Cập nhật camera theo người chơi |
| Collectible | `updateCollectibles(elapsedTime)` | ~4358 | Kiểm tra va chạm thu thập gem |
| Minimap | `renderMiniMap(elapsedTime)` | ~4391 | Vẽ bản đồ nhỏ (minimap) 2D |
| Loop | `animate()` | ~4554 | Vòng lặp render chính (requestAnimationFrame) |
| Map | `buildClimbMap2()` | ~4712 | Xây dựng bản đồ Pháo Đài Không Gian (20 bậc) |
| Map | `buildHuntArenaMap(mapId)` | ~4776 | Xây dựng mê cung săn bia |
| World | `buildMenuShowcaseWorld()` | ~2156 | Xây dựng cảnh showcase menu |
| Gameplay | `spawnRandomHuntTarget()` | — | Sinh bia mục tiêu ngẫu nhiên (hunt) |
| Combat | `shootLaser()` | — | Bắn tia laser (plasma/lightning) |
| Combat | `throwShampooBomb()` | — | Ném bom champir |
| Combat | `updateProjectiles()` | — | Cập nhật dasan và hiệu ứng |
| Runner | `buildRunnerWorld()` | — | Xây dựng.world chạy tự động |
| Bomber | `startBomberGame()` | — | Khởi động trò chơi đặt bom |
| Save | `LocalSaveManager` | ~6231 | Quản lý lưu/nạp/export/import JSON |
| Save | `saveManager.resetAllData()` | — | Xóa toàn bộ tiến trình lưu |
| Skin | `drawSkinAvatarIcon()` | — | Vẽ icon skin trên canvas HTML |
| Dev | `updateDeveloperModeBadge()` | — | Cập nhật badge Dev Mode trên HUD |

---

## 7. CAU TRUC DU LIEU STATE (GAME STATE)

```javascript
state = {
    screen: 'menu' | 'playing',
    gameMode: 'climb' | 'hunt' | 'runner' | 'bomber',
    climbMapId: 'map1',
    climbLevel: 1,
    climbMapLevels: { map1: 1, map2: 1 },
    huntMapId: 'map1' | 'map2',
    huntLevel: 1,
    huntMapLevels: { map1: 1, map2: 1 },
    matchEnded: false,
    isPaused: false,          // Trạng thái tạm dừng (thoát xác nhận)
    isDevMode: false,         // Chế độ lập trình viên (Ctrl+Shift+H+P)
    backgroundTheme: 'galaxy' | 'noel',
    pos: {x, y, z},           // Vị trí người chơi
    hp: 100,                  // Máu tối đa 100
    maxHp: 100,
    score: 0,
    totalStarsAllTime: 0,     // Tổng gem tích lũy
    totalTargetsAllTime: 0,   // Tổng mục tiêu bắn trúng
    huntGoal: 15,             // Số bia cần hạ trong hunt
    huntDestroyed: 0,         // Số bia đã hạ
    weaponMode: 'plasma',     // 'plasma' | 'lightning'
    skin: 'chrome',           // Key skin hiện tại
    unlockedSkins: ['chrome', 'cyber', 'maczt'], // Danh sách skin đã mở
    completedMaps: { climb_map1: false, climb_map2: false, hunt_mode: false, bomber_map1: false },
    keys: {w:false, a:false, s:false, d:false, space: false},
    joyVec: {x:0, y:0},       // Vector joystick di động
    isDragging: false,        // Kéo chuột trên menu
    isAimingBomb: false,      // Đang định hướng ném bom
    muted: false,             // Tắt âm thanh
    // Chế độ runner
    runnerLane: 1,
    runnerDistance: 0,
    runnerCoinsCollected: 0,
    runnerSpeed: 17,
    runnerLaneCooldown: 0,
    // Tiến độ leo thang
    currentStep: 0,
    highestStep: 0,
    highestGroundedY: 0,
    matchStartTime: 0
};
```

---

## 8. RESPONSIVE & THIET BI DI DONG

- Sử dụng `@media (orientation: portrait) and (max-width: 768px)` để điều chỉnh kích thước joystick và nút hành động
- Joystick ảo chỉ hiển thị trên màn hình nhỏ (`md:hidden`)
- Nút hành động (`btn-fire`, `btn-bomb`, `touch-jump`) được căn lề phải, kích thước lớn hơn trên di động
- `#canvas-container` chiếm toàn bộ màn hình, `overflow: hidden`, `user-select: none`, `touch-action: none` để tránh chọn văn bản và cuộn trang vô tình khi chơi

---

## 9. CACH CHAY GAME

1. Mở tệp `index.html` trên trình duyệt web hiện đại (Chrome, Firefox, Edge, Safari)
2. Đảm bảo kết nối internet để tải:
   - Tailwind CSS CDN
   - Google Fonts (Inter)
   - Three.js r160 từ unpkg
3. Không cần cài đặt build tool hay máy chủ local (có thể mở trực tiếp qua `file://`)

---

## 10. GIOI HAN & DIEU KIEN

- Đây là bản chạy trên trình duyệt (web), **không phải** game Roblox thực sự sử dụng Lua/Roblox Studio. Nó chỉ mô phỏng phong cách Roblox/Bearbrick bằng Three.js.
- Âm thanh được tạo thủ tục qua Web Audio API, cần tương tác người dùng đầu tiên (click/chạm) để khởi tạo `AudioContext` (tránh bị chặn bởi trình duyệt).
- Dữ liệu lưu trữ cục bộ qua `localStorage`, có thể bị xóa nếu người dùng xóa dữ liệu duyệt web.
- Hiện có **2 bản đồ climb** (Tháp Không Gian 18 tầng, Pháo Đài Không Gian 20 tầng), **2 bản đồ hunt** (Mê Cung Cổ Điển, Mê Cung Chuyển Động), **1 world runner** và **2 bản đồ bomber**. Các bản đồ/chế độ khác có thể mở rộng bằng cách viết thêm hàm builder và gọi trong `startGameMode()`.

---

*Tài liệu này được tự động sinh từ phân tích mã nguồn `index.html` của dự án Roblox_bearbrick. Nếu có thay đổi mã nguồn, tài liệu cần được cập nhật lại tương ứng.*