Mong muốn xây dựng KIRA – Trợ lý NMD
Mục 1 —  thời tiết qua Open-Meteo (Đang hoàn thiện)
Vẫn còn lỗi hỏi thông tin thì chạy sang Groq (vẫn trả lời độ ẩm ở Sài Gòn) chứ không phải Open Meteo
Mục 2 — Tra thông tin (giá + tin tức)
Mong muốn:
•	2.1 Giá: tỷ giá ngoại tệ chỉ lấy từ Vietcombank (XML chính thức); giá vàng chỉ từ BTMC (API chính thức); các loại giá khác (tiền ảo, xăng dầu, lãi suất tiết kiệm...) lấy từ webgia.com qua JSON-LD nhúng, không parse bảng HTML.
•	2.2 Tin tức khác (xã hội, pháp luật, kinh tế...): dùng RSS các trang do người dùng cung cấp theo mục trong LIST.yaml; KIRA phải tìm thông tin sát nhất với câu hỏi.
Giải pháp:
•	2.1: ba handler bypass-Groq độc lập — mỗi handler gọi trực tiếp đúng nguồn theo domain câu hỏi (tỷ giá → Vietcombank XML; vàng → BTMC API; giá khác → webgia.com (JSON-LD)).
•	2.2: lấy tiêu đề/tóm tắt các bài RSS mới nhất trong danh mục liên quan (tra từ LIST.yaml) → gửi kèm câu hỏi lên Groq để chọn bài khớp nhất/tổng hợp trả lời (kiểu RAG đơn giản). Đây là nội dung công khai, không phải Memory riêng của nhà, nên gửi lên Cloud AI không vi phạm nguyên tắc "Memory local không gửi Cloud".
Mục 3 — Lưu thông tin (Memory local)
Mong muốn:
•	Khi người dùng nói "KIRA, lưu lại/ghi nhớ/lưu" → KIRA hỏi lại "Bạn muốn KIRA lưu gì?" → lưu câu trả lời vào Memory local (VD "Phòng khách dùng đèn Rạng Đông").
•	Nếu 2 tháng sau người dùng nói thông tin khác cho cùng chủ đề (VD "Phòng khách dùng đèn Philips") → KIRA phải hỏi lại xác nhận ("Đèn Rạng Đông hay đèn Philips?") trước khi ghi đè giá trị cũ.
•	Không liên quan đến mục 4 (nhắc việc) — câu không có yếu tố thời gian.
Giải pháp:
•	Tách câu bằng rule-based tự viết, dùng thư viện parse (MIT, thuần Python, nhẹ, ổn định) thay cho regex thô — viết mẫu kiểu "{subject} dùng {value}" để tách chủ thể/giá trị.
•	Danh sách từ khóa nối (dùng/là/có/để/đặt ở...) để trong 1 file config riêng (VD linking_words.yaml), phân loại theo attribute_type (VD "thiết bị", "cài đặt/thông số") — dễ bổ sung khi gặp câu không khớp mẫu.
•	Khóa so sánh trùng lặp: (subject, attribute_type), VD ("phòng khách", "thiết bị"). Khi câu mới trùng khóa nhưng value khác → kích hoạt hỏi xác nhận đè.
•	Lưu trong SQLite (đã có sẵn) với cột: subject, attribute_type, value, raw_text, updated_at.
•	Quản lý luồng hỏi-đáp nhiều lượt (chờ nội dung cần lưu → chờ xác nhận đè) bằng thư viện transitions (state machine nhẹ, MIT, đang được bảo trì tích cực) để tránh nhận nhầm câu nói khác là nội dung cần lưu.
Mục 4 — Nhắc việc
Mong muốn:
•	Cùng câu kích hoạt với mục 3 ("KIRA, lưu lại/ghi nhớ/lưu") → KIRA hỏi "Bạn muốn KIRA lưu gì?" → người trả lời có yếu tố thời gian (VD "5 ngày nữa, lúc 9h nhắc tôi nộp tiền điện") → đúng thời điểm, KIRA nhắc.
•	Giờ không nói rõ sáng/tối → hiểu là AM nếu nhỏ hơn 12 (VD "lúc 9h" = 9h sáng).
•	Nếu quá giờ nhắc mà không ai ở nhà nghe (endpoint không active) → KIRA nhắc lại ngay khi được mở/tương tác lần đầu, miễn còn trong 24h kể từ giờ hẹn; quá 24h thì bỏ qua.
Giải pháp:
•	Phân nhánh dựa vào việc câu trả lời có/không có yếu tố thời gian (nhận diện bằng mẫu "X ngày/giờ nữa", "lúc Y giờ" qua parse/regex) để tách case này khỏi mục 3 (lưu kiến thức tĩnh, không thời gian) — dùng chung câu kích hoạt nhưng tự động phân loại ý định sau khi nghe nội dung.
•	Lưu trong SQLite (memory local) với các trường: nội dung nhắc, due_at (thời điểm), delivered (đã nhắc chưa), created_at.
•	Logic kiểm tra khi KIRA khởi động/có tương tác: quét các bản ghi due_at đã qua nhưng delivered=false và còn trong 24h → phát nhắc; quá 24h → đánh dấu bỏ qua, không phát nữa.
Mục 5 — Phát nhạc online
Mong muốn:
•	"KIRA mở/phát/bật nhạc [tên]" → tìm playlist YouTube theo tên trong LIST.yaml → phát trên endpoint của phòng ra lệnh (mỗi phòng độc lập, tự chạy iframe YouTube riêng trong browser thiết bị đó).
•	KIRA nhớ vị trí (giây) đang dừng ở playlist đó để lần sau mở lại phát tiếp.
•	Lệnh điều khiển khi đang mở: "KIRA tắt nhạc" → stopVideo(); "KIRA chuyển bài"/"bài tiếp theo" → nextVideo(); "bài trước" → previousVideo().
•	Khi đang nghe nhạc, endpoint đó chỉ nhận lệnh liên quan tới nhạc, không xử lý lệnh khác.
Giải pháp:
•	LIST.yaml chứa tên playlist ↔ link YouTube playlist.
•	Gateway không thể gọi trực tiếp hàm JS (nextVideo()...) của YouTube iframe vì hàm đó chỉ tồn tại trong trình duyệt của endpoint — cần một kênh real-time (VD WebSocket) để Gateway đẩy lệnh play/pause/seek/next/previous xuống đúng endpoint đang phát; endpoint báo lại vị trí giây hiện tại theo định kỳ hoặc khi dừng, để Gateway lưu resume.
•	"Music mode": khi 1 endpoint đang phát nhạc, router lệnh của endpoint đó chỉ cho tập lệnh liên quan nhạc đi qua — quản lý bằng state machine (transitions), cùng cơ chế dùng cho luồng hỏi-đáp ở mục 3/4.
•	Resume lưu dạng: tên playlist, video hiện tại, giây dừng — theo từng endpoint/phòng (vì mỗi phòng độc lập).
Mục 6 — Phát sách truyện
Mong muốn:
•	"KIRA mở/phát/bật sách truyện" → KIRA hỏi "Bạn muốn nghe truyện gì?" → mở thư mục tương ứng (thư mục riêng từng loại, lưu trên HP Box) và chạy file mp3, theo ánh xạ tên trong LIST.yaml.
•	Có resume: biết lần trước nghe dở ở đâu trong thư mục để phát tiếp.
•	Lệnh điều khiển giống mục 5: "KIRA tắt nhạc" → stop(); "KIRA chuyển bài"/"bài tiếp theo" → next(); "bài trước" → previous().
•	Nếu SSD không đủ dung lượng, gỡ Windows 10 khỏi HP Box (đang dual-boot song song Debian 13 XFCE) để giải phóng chỗ.
Giải pháp:
•	LIST.yaml thêm mục ánh xạ tên loại truyện ↔ đường dẫn thư mục mp3 trên HP Box.
•	Mỗi thư mục có 1 file resume.yaml riêng, lưu file đang nghe + vị trí giây, để KIRA phát tiếp đúng chỗ.
•	Vì file nằm sẵn trên HP Box (khác nguồn ngoài của mục 5), Gateway có thể tự stream file qua HTTP tới endpoint, đơn giản hơn cơ chế iframe YouTube của mục 5.
•	Cấu trúc resume tương tự mục 5 (thư mục/playlist, item, timestamp) — có thể dùng chung 1 schema cho cả 2 mục để tránh trùng lặp logic.
•	Việc gỡ Windows 10 là hành động hạ tầng (giải phóng SSD), thực hiện khi cần thêm dung lượng, không phải logic phần mềm.
Mục 7 — HASS + Tuya Cloud
Mong muốn:
•	Thiết bị hiện có trong nhà (Tuya Zigbee): công tắc, cảm biến chuyển động, cảm biến nhiệt độ + độ ẩm, cảm biến mở cửa/cửa sổ.
•	Bật/tắt, nhận trạng thái, thông số cảm biến của các thiết bị qua HASS và Tuya Cloud.
•	Có thể ra lệnh giọng nói bật/tắt thiết bị, hỏi/trả lời thông tin từ cảm biến (VD "cửa phòng khách đang mở hay đóng", "nhiệt độ phòng ngủ bao nhiêu").
•	KIRA phải đọc kết quả bằng giọng nói qua TTS khi được hỏi, không chỉ trả dữ liệu thô.
•	Nếu chạy song song nặng máy thì làm sau (tạm gác lại).
Giải pháp:
•	Dùng Home Assistant Container (bản Core chạy qua Docker, không phải HAOS đầy đủ có Supervisor/Add-on Store) — với quy mô thiết bị nhỏ của nhà (công tắc, cảm biến chuyển động, nhiệt độ/độ ẩm, mở cửa — vài chục thiết bị), ước tính chỉ tốn thêm khoảng 500MB-1GB RAM (HA Core ~300-600MB + Zigbee2MQTT ~150-300MB + Mosquitto ~50-100MB) — Kiểm tra xem có phù hợp với 7.7GB RAM hiện có của HP Box không.
•	Qua tích hợp Tuya Cloud có sẵn trong HASS làm lớp trung gian quản lý trạng thái thiết bị.
•	KIRA Gateway gọi REST API/WebSocket API của HASS: bật/tắt qua service call (VD light.turn_on/switch.turn_on), đọc trạng thái/cảm biến qua GET /api/states/<entity_id> (gọi khi người dùng hỏi, không cần subscribe liên tục trừ khi cần cảnh báo chủ động).
•	Ánh xạ tên gọi tiếng Việt ↔ entity_id lưu trong LIST.yaml.
•	Luồng đầy đủ: câu hỏi giọng nói → nhận diện ý định (điều khiển/hỏi cảm biến) → map tên ↔ entity_id → gọi HASS API → nhận kết quả → ghép câu trả lời → TTS (Google Cloud TTS, pipeline đã có) phát ra loa endpoint — giống luồng đang dùng cho thời tiết/giá, chỉ thêm bước gọi HASS.
•	Với quy mô thiết bị hiện tại, chạy HASS Container chung HP Box (cùng Gateway) là khả thi; vẫn cần theo dõi CPU 2-core khi nhiều service chạy song song.

