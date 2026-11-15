# Roblox Bearbrick 3D World — Code Treemap

Tài liệu định hướng nhanh cho agent/contributor trước khi sửa game. Mã nguồn runtime hiện tập trung trong một file: [`index.html`](./index.html). Dùng tên section hoặc hàm để tìm trong source; vị trí dòng thay đổi theo các lần cập nhật.

## 1. Cấu trúc repository

```text
Roblox_bearbrick/
├── index.html    # UI, stylesheet, Three.js scene và toàn bộ logic game
├── README.md     # Tên dự án và tác giả
├── rulegame.md   # Mô tả gameplay/controls
└── treemap.md    # Bản đồ cấu trúc mã nguồn
```

`index.html` chạy trực tiếp trên trình duyệt; không có bundler hay thư mục `src/`. Trang tải Tailwind CSS, Inter và Three.js r160 qua CDN/import map. JavaScript nằm trong một `<script type="module">`.

## 2. UI và runtime

### HTML/CSS

- `<head>`: CSS toàn cục, responsive/mobile controls, glass-panel và scrollbar theme.
- `#canvas-container`: WebGL canvas do Three.js tạo.
- Backdrop Galaxy/Noel được tạo riêng trong `scene` (không thuộc `worldGroup`), nên vẫn xuất hiện ở cả sảnh và mọi map. Noel thêm mây đêm, tuyết và xe tuần lộc.
- `#main-menu-overlay`: sảnh, điểm/thành tích, cửa hàng skin, tab phụ kiện/vũ khí, chọn chế độ và bản đồ.
- `#gameplay-hud`: HP/objective, avatar drawer, pointer lock, radar, bàn phím hướng dẫn và nút cảm ứng.
- `#result-modal`, `#exit-confirm-modal`: kết quả trận đấu và xác nhận thoát có pause.
- `#minimap-canvas`: radar 2D; `#xray-markers-layer`: chỉ báo bia nguy hiểm xuyên tường.

### Khởi tạo và vòng lặp

- Tạo `scene`, `camera`, `renderer`, ánh sáng và nền chủ đề ở section 2.
- `createSpaceBackdrop()` dựng các model nền dùng chung; `updateSpaceBackdrop()` quay hành tinh/mặt trăng, cho tàu bay theo quỹ đạo, sinh sao băng hoặc cập nhật tuyết/mây/xe tuần lộc Noel.
- `setBackgroundTheme()` đổi theme từ panel Background ở sảnh và lưu lựa chọn.
- `SoundEngine` phát BGM procedural theo theme Galaxy/Noel và mode sảnh/trận; audio bắt đầu sau tương tác đầu tiên để tuân thủ autoplay policy của trình duyệt.
- Đọc save, dựng sảnh qua `enterMainMenu()`, rồi gọi `animate()` khi trang tải.
- Mỗi frame, nếu không pause: cập nhật obstacle, vật lý, animation nhân vật, projectile/collectible, camera và radar; cuối cùng render Three.js.
- `startGameMode(mode, mapId)` dựng world và reset state cho trận mới. Hàm này được gọi từ nút chọn mode/map và nút chơi lại; yêu cầu Pointer Lock ngay trong thao tác bắt đầu.
- `pauseForExitConfirmation()` bật `state.isPaused`; vòng lặp tiếp tục render nhưng bỏ qua update gameplay. `resumeFromExitConfirmation()` tiếp tục trận, còn xác nhận thoát gọi `enterMainMenu()`.

## 3. Bản đồ section JavaScript trong `index.html`

| Section trong source | Trách nhiệm và điểm vào chính |
|---|---|
| 1. Pure Code Web Audio API Sound Engine | `SoundEngine`: SFX và BGM procedural theo theme/mode |
| 2. Three.js Scene, Space-Obby Sky & Lighting | Scene/camera/renderer, Galaxy và Noel backdrop |
| 3. Procedural Textures | Studs, cỏ, nhãn shampoo, logo VIP |
| 4. Bearbrick Character Builder & Skin Catalog | `SKINS`, `BearbrickCharacter`, dựng/animation nhân vật |
| 5. World Builders | World registries, collision helpers, map builders và trang trí lobby theo theme |
| 6. Weapons | Mục tiêu hunt (bia kẹo khi Noel), súng, bom, hiệu ứng và projectile |
| 7. Player State, HUD, Skin Shop & Save System | `state`, theme selection, HUD, skin shop, `LocalSaveManager`, chuyển màn hình |
| Input & UI Event Handlers | Keyboard, mouse/pointer lock, touch/joystick và button bindings |
| 8. Obstacle Updates, AABB Physics & HP Regen | Obstacle animation, tuần lộc map/lobby, physics, HP và camera |
| 9. Camera-Relative Rotating 2D Radar Mini-Map | `renderMiniMap()`: tường, gem, bia và mũi tên chỉ bia ngoài phạm vi |
| 10. Main Animation Loop & Initialization | `animate()`, backdrop update, load save và khởi chạy |

## 4. Chức năng theo subsystem

### World và va chạm

- `worldGroup`: nhóm mesh của world hiện tại; `clearWorld()` dọn world và các registry trước khi dựng map khác.
- `addPlatformBlock(...)`: tạo platform/tường, mesh và AABB collider. Với tường maze cần `recordMiniMapWall = true` để đăng ký radar và camera occlusion.
- Registry chính: `colliders`, `platformMeshes`, `stars`, `targets`, `rollingRails`, `sweeperSpinners`, `miniMapWalls`, `cameraOccluders`, `huntSpawnPoints`, `healingPools`.
- Builders: `buildMenuShowcaseWorld()`, `buildClimbMap1()`, `buildClimbMap2()`, `buildHuntArenaMap()`.
- `addClimbDifficultyExtension()` mở rộng tháp theo độ khó bằng platform trượt/vòng xoay xen kẽ; Map 2 giữ ray dẫn ngang, chỉ mặt trượt nghiêng 15° và collider tạo lực trượt xuống dốc.
- `buildChristmasMapDecorations()` thêm cây thông, mảng tuyết và tuần lộc chuyển động cho hai map climb khi theme Noel được chọn.
- `buildLobbyThemeDecorations()` thêm UFO/người ngoài hành tinh cho Galaxy hoặc cây thông, tuần lộc, quà, gậy kẹo và tất chân cho Noel.
- `updateCameraOcclusion()` làm mờ tường chắn camera. Khi bắn, các tường đang mờ được bỏ qua trong chọn điểm ngắm và kiểm tra va chạm projectile.

### Gameplay và input

- `state.gameMode`: `climb` hoặc `hunt`; `state.climbMapId`: `map1`/`map2`.
- Climb: `updatePhysics()`, `updateDynamicObstacles()`, `updateCollectibles()`.
- Hunt: `spawnRandomHuntTarget()`, `getCrosshairWorldTarget()`, `shootLaser()`, `throwShampooBomb()`, `updateProjectiles()`.
- Hunt có `map1` (bia đứng yên) và `map2` (bia di chuyển chậm); `state.huntMapLevels` lưu cấp riêng từng map. Mỗi lần thắng tăng level map đó và thêm 5 bia mục tiêu.
- Climb có `state.climbMapLevels` độc lập cho hai map; `addClimbDifficultyExtension()` thêm 4 bậc xen kẽ ray trượt/vòng xoay (ô vòng xoay rộng 7.5 đơn vị) và gem cho mỗi ⭐ tăng thêm theo đường vòng lục giác bán kính cố định, không có trần level trong code.
- `menu-maps-cleared` hiển thị tổng ⭐ của cả bốn map; badge hoàn thành cũ đã được bỏ.
- PC: WASD/arrows di chuyển, Space nhảy; hunt dùng F bắn, G/Q đổi đạn, chuột phải giữ-ngắm/thả-ném bom.
- Trong trận, Ctrl+Shift rồi nhấn H và P bật/tắt `state.isDevMode`; damage/knockback bị chặn, còn ArrowUp nhảy sang bậc kế tiếp hoặc hạ bia sắp nổ nhất.
- Touch input gồm virtual joystick, camera drag và các nút nhảy/bắn/bom.
- Nút **Thoát** pause trận và hỏi xác nhận trước khi về sảnh.

### Skin và cửa hàng

- `SKINS` là catalog; `drawSkinAvatarIcon()` vẽ icon skin trên HTML canvas.
- `getSkinCostLabel()`, `canAffordSkin()`, `handleSkinClick()` quản lý giá/mở khóa; `renderSkinShopUI()` dựng danh sách icon ở sảnh và avatar drawer.
- Skin mặc định: `chrome`, `cyber`, `maczt`. Các VIP khác cần đồng thời `36 💎 + 50 🎯`.
- Cửa hàng sảnh có tab skin, phụ kiện và skin vũ khí; hai tab sau hiện là placeholder.
- `spawnRandomHuntTarget()` dựng bia kẹo khi `state.backgroundTheme === 'noel'`.

### Lưu tiến trình

- `LocalSaveManager` lưu trong `localStorage` key `roblox_bearbrick_3d_save_v4`; payload hiện ở version 8, chứa background theme và tiến trình difficulty/level của từng map.
- `scheduleSave()` debounce autosave; có lưu ngay, reset, import và export JSON.
- Khi thay đổi danh mục/unlock skin, kiểm tra `DEFAULT_UNLOCKED_SKINS` và logic migration trong `applyLoadedData()`.

## 5. Hướng dẫn mở rộng

- **Thêm map climb:** tạo builder cạnh `buildClimbMap1/2()`, nối vào `startGameMode()`, nút sảnh, spawn/respawn và `completedMaps`.
- **Thêm mode:** cập nhật menu button + binding, nhánh `startGameMode()`, HUD/objective, update loop, completion/result, minimap và save nếu có tiến trình mới.
- **Thêm skin:** thêm catalog entry vào `SKINS`, xử lý chi tiết icon trong `drawSkinAvatarIcon()` nếu cần, rồi kiểm tra default unlock, giá và migration save.
- **Thêm phụ kiện/skin vũ khí:** bắt đầu từ các panel tab placeholder và thêm riêng state, catalog, equip UI, render/asset logic và persistence.
- **Sửa platform/tường:** dùng builder chung để mesh/collider/radar/camera-occlusion được đăng ký nhất quán.

Không có test runner riêng trong repository; sau sửa JavaScript có thể kiểm tra cú pháp bằng Node trên nội dung module và dùng `git diff --check`.
