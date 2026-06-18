---
artifact: 02 — JTBD Project Analysis
bai-tap: Lab 2 — Dùng JTBD để soi lại dự án nhóm
format: Theo nhóm dự án → share trong bàn → chốt hypothesis cuối
time: 25 phút trên lớp
nop-cuoi: Có — đây là file nộp cuối của Lab 2
companion-reference: Strategyn_JTBD_Playbook.pdf (giảng viên gửi kèm)
---

# Lab 2 — JTBD Project Analysis / Dùng JTBD để soi lại dự án nhóm

**Tên dự án / sản phẩm:** AI20K-170 v2 — Synthetic-to-Real YOLO Dataset Pipeline

> Đây là **file duy nhất** của Lab 2.  
> File này đồng thời đóng vai trò:
>
> - guide từng bước,
> - worksheet để điền trực tiếp,
> - và file nộp cuối cho người chấm.

Mục tiêu của bài này không phải brainstorm thêm thật nhiều tính năng AI.
Mục tiêu là:

1. **xác định người dùng thực sự đang cố hoàn thành job gì**
2. **hiểu họ đang dùng giải pháp nào để hoàn thành job đó hôm nay**
3. **chỉ ra AI nên chen vào đúng bước nào trong workflow**
4. **viết ra product hypothesis và assumption còn phải validate**

Quy tắc xuyên suốt: **không rõ job thì đừng bàn feature.**

---

## Bước 0 — Khoanh đúng 1 lát cắt của dự án

### Điền nhanh trước khi làm

- **Dự án của nhóm tôi là:** AI20K-170 v2 — pipeline tạo YOLO dataset synthetic-to-real cho tabletop robot object detection, dùng PyBullet sim + generative realism + Grounding DINO QA gate
- **Lát cắt tôi chọn để phân tích hôm nay là:** Robotics Perception Engineer cần tạo labeled dataset cho tabletop object detection khi không có ngân sách/thời gian để chụp-gán nhãn thủ công toàn bộ
- **Vì sao tôi chọn lát cắt này:**  
  > Đây là pain point lõi và đã có bằng chứng học thuật xác nhận (YCB-Video dataset 133,827 frame được tạo ra chính xác vì chụp ảnh thật quá tốn kém). Nếu lát cắt này không đủ đau hoặc không xảy ra thường xuyên, cả pipeline sẽ không có value.

---

## Bước 1 — Viết `Project Snapshot`

### Tóm tắt dự án trong 3 dòng

1. **Nhóm tôi đang nghĩ mình đang giải quyết vấn đề gì?**  
   > Sim-to-real gap: detector train từ dữ liệu simulator thường kém hiệu năng trên camera thật vì khác biệt texture, lighting, background — và thu thập/gán nhãn ảnh thật tốn quá nhiều thời gian để lặp lại mỗi khi đổi object hay scene.

2. **Người dùng chính hiện nhóm đang nhắm tới là ai?**  
   > Robotics Perception Engineer tại startup hoặc lab nhỏ đang build detector tabletop pick-and-place/sorting, dùng YOLO, có simulator nhưng không có data team riêng.

3. **Hiện tại người dùng đó đang giải quyết vấn đề này bằng cách nào?**  
   > Hoặc chịu đựng sim-only data (hiệu năng kém), hoặc tự chụp ảnh thật + gán nhãn bằng tay (tốn 2–5 ngày/object), hoặc dùng classic domain randomization đơn thuần mà không có QA kiểm tra label preservation.

---

## Bước 2 — Viết `Market Context`

### Trả lời 4 câu ngắn

1. **Ai đang gặp vấn đề này?**  
   > Robotics Perception Engineer tại startup robotics nhỏ (3–15 người), lab nghiên cứu đại học, hoặc freelancer build demo tabletop perception cho investor/khách hàng. Người trực tiếp bị kẹt ở khâu data là engineer, không phải founder.

2. **Vấn đề xuất hiện trong hoàn cảnh nào?**  
   > Khi team cần train/retrain detector sau khi đổi vật thể, thay đổi background setup, hoặc chuyển sang camera mới — tức là bất kỳ lúc nào họ cần dataset mới và không thể chờ 2–5 ngày chụp-gán nhãn thủ công. Deadline demo với mentor/nhà đầu tư làm áp lực tăng cao.

3. **Hiện tại họ đang dùng giải pháp thay thế nào?**  
   > (1) Sim-only renders + domain randomization thủ công; (2) Chụp ảnh thật + LabelImg/CVAT gán nhãn bằng tay; (3) Dùng dataset công khai như YCB-Video nhưng không khớp camera/scene setup thực tế của họ; (4) Không validate — ship model sim-only, accept performance drop.

4. **Vì sao đây là thời điểm đáng giải?**  
   > Generative AI (SD1.5, ControlNet, Qwen-Image-Edit) giờ đã tốt đến mức có thể "reality-fy" ảnh sim ra ảnh gần thật trong vài giây/ảnh với chi phí thấp. Cùng lúc, zero-shot detection (Grounding DINO) cho phép kiểm tra tự động xem label có còn đúng sau realism pass không — 2 khả năng này cùng lúc mới mở ra pipeline không tồn tại 2–3 năm trước.

### Tóm tắt market context trong 3-4 dòng

> Robotics startup nhỏ và lab nghiên cứu bị kẹt ở data bottleneck mỗi lần cần deploy detector trên real camera: không có data team riêng, chụp ảnh thật tốn 2–5 ngày/object, gán nhãn dễ sai và không lặp lại được. Sim-only data là shortcut nhưng thường dẫn đến performance drop trên camera thật do sim-to-real gap. Classic domain randomization giảm gap một phần nhưng không có QA gate để xác nhận label vẫn đúng sau khi apply realism. AI20K-170 nhắm đúng vào điểm này: giảm manual labeling nhưng bắt buộc có real-camera validation để chứng minh value.

---

## Bước 3 — Xác định `Job Executor`

### Điền

- **Job executor của dự án này là:** Robotics Perception Engineer tại startup hoặc lab nhỏ — người trực tiếp build, train, và validate detector tabletop object detection
- **Vì sao tôi tin đây là người trực tiếp "thuê" giải pháp để làm job:**  
  > Founder/CTO có thể là người mua, nhưng họ không trực tiếp chạy pipeline data, không ngồi label ảnh, không debug mAP regression. Người bị kẹt và người "hire" tool để thoát khỏi bottleneck là Perception Engineer — người đó trực tiếp cảm nhận pain khi phải lặp lại vòng chụp-label mỗi lần đổi object hay scene.

---

## Bước 4 — Viết `Core JTBD`

### Bản nháp 1

**Core JTBD bản nháp:**  
> Tạo dataset labeled đủ tin cậy cho known tabletop objects bằng AI-assisted pipeline để train detector chạy tốt trên real camera mà không cần lặp lại vòng chụp-gán nhãn thủ công

### Gạch bỏ từ solution nếu có

- Các từ solution tôi đang lỡ nhét vào câu: *"AI-assisted pipeline"* — bỏ đi; *"synthetic-to-real"* — bỏ đi; đây là how, không phải job

### Bản chốt

**Core JTBD cuối cùng:**  
> Có được dataset object detection đủ đa dạng và đủ tin cậy về label để detector chạy đúng trên camera thật — mà không mất nhiều ngày gán nhãn thủ công mỗi lần đổi vật thể hoặc scene

---

## Bước 5 — Viết 3 `Job Stories`

### Bảng 3 job stories

| # | Trigger / When | Motivation / I want to | Outcome / so I can | Điều story này cho thấy |
|---|---|---|---|---|
| JS1 | Khi tôi cần retrain detector cho một vật thể mới trước buổi demo với mentor 3 ngày nữa | Tôi muốn có dataset labeled đủ tốt trong vòng vài giờ thay vì mấy ngày | Để train model kịp và có số mAP trình bày được, không phải demo "sẽ làm" | Deadline áp lực là trigger chính — 2–5 ngày chụp-label thủ công không còn khả thi. Job không phải "làm dataset đẹp", job là "có số liệu kịp deadline" |
| JS2 | Khi tôi đổi từ background trắng sang background thật (bàn gỗ, context nhà xưởng) mà không muốn chụp lại từ đầu | Tôi muốn adapt dataset cũ sang distribution mới một cách nhanh chóng và giữ label vẫn đúng | Để kiểm tra xem model có generalize sang background mới không mà không tốn thêm 2 ngày chụp ảnh | Flexibility là giá trị lớn: user cần test nhiều scene configurations nhanh, không phải commit vào một setup cố định |
| JS3 | Khi tôi nộp model cho reviewer/mentor và bị hỏi "dataset này đến từ đâu, label có đáng tin không?" | Tôi muốn có evidence package rõ ràng — reject rate, IoU gate, golden benchmark — để trả lời câu hỏi đó | Để review pass được và không bị nghi ngờ về chất lượng data | Accountability quan trọng hơn tưởng: perception engineer không chỉ cần dataset chạy được mà cần chứng minh được data quality cho stakeholder |

---

## Bước 6 — Liệt kê `Current Alternatives`

### Bảng alternatives

| Alternative hiện tại | User đang thuê nó để làm gì? | Nó làm tốt gì? | Nó fail ở đâu? | Switching cost hiện tại cao hay thấp? |
|---|---|---|---|---|
| Chụp ảnh thật + gán nhãn bằng tay (LabelImg, CVAT) | Tạo dataset có label chính xác, đúng real distribution | Label chính xác tuyệt đối; real distribution không có sim gap | Tốn 2–5 ngày/object; không scale khi đổi scene/object; tốn nhân lực | **Cao** — chi phí thời gian quá cao để dễ dàng bỏ |
| Sim-only domain randomization (PyBullet + classical augmentation) | Sinh label tự động, nhanh, không tốn công | Nhanh, reproducible, không tốn tiền | Sim-to-real gap làm mAP drop khi test trên camera thật; không có cách verify label vẫn đúng sau augmentation mạnh | **Thấp** — engineer đã quen dùng, ít tốn công nhưng biết kết quả sẽ kém |
| Dataset công khai (YCB-Video, COCO) | Bỏ qua khâu data, train thẳng | Không tốn công; label đã có; cộng đồng đã validate | Không match camera/scene setup thực tế; không thể customize object set; không thể retrain khi scene đổi | **Thấp** — nhưng bị giới hạn nghiêm trọng về flexibility |
| Không validate — ship sim-only model | Tiết kiệm thời gian tối đa, demo nhanh | Nhanh nhất | Không biết model có chạy trên real camera không; bị bắt trong review khi reviewer hỏi evidence | **Thấp** — nhưng rủi ro bị reject cao |

### Kết luận nhanh

**Nếu project của tôi biến mất hôm nay, user nhiều khả năng sẽ quay về:**  
> Sim-only domain randomization + chịu đựng performance drop, hoặc chụp ảnh thật cho vật thể quan trọng nhất và bỏ qua các vật thể còn lại. Phần lớn sẽ tiếp tục "live with the gap" vì không có thời gian làm tốt hơn.

---

## Bước 7 — Điền `JTBD Lite Map`

### Bảng JTBD Lite Map

| Step | Trong workflow này user đang cố làm gì? | Hôm nay họ đang dùng gì? | Friction / pain hiện tại | Mức đau |
|---|---|---|---|---|
| Define | Xác định vật thể cần detect, yêu cầu mAP, camera setup | Tự spec trong đầu hoặc doc nháp; không có template chuẩn | Không rõ "đủ tốt" là gì; không có benchmark để so | Med |
| Locate | Tìm 3D mesh của vật thể + setup simulator (PyBullet/Blender) | Tự download URDF/mesh từ Internet; đôi khi phải tự model | URDF không đúng COM hoặc scale → sim không realistic ngay từ đầu | Med |
| Prepare | Sinh ảnh sim + gán nhãn + apply domain randomization **+ chụp ảnh thật nếu sim không đủ** | Script tự viết + LabelImg hoặc CVAT cho ảnh thật | **Đây là bước đau nhất**: chụp-label thủ công tốn 2–5 ngày/object; không reproducible khi đổi scene | **High** |
| Confirm | Kiểm tra xem label sau augmentation/realism có còn đúng không | Visual inspection thủ công (nhìn bbox overlay) | Không có automated QA — engineer phải xem từng ảnh; dễ miss lỗi hệ thống như label drift sau realism | **High** |
| Execute | Train YOLO trên dataset, tune hyperparams | Ultralytics YOLO CLI; notebook Colab/RunPod | Slow feedback loop: train 1–2 giờ mới biết dataset có vấn đề không | Med |
| Monitor | Theo dõi mAP trong quá trình train; phát hiện regression | Tensorboard / wandb / YOLO CLI output | Khó phân biệt "data quality vấn đề" vs "hyperparams vấn đề" khi mAP thấp | Med |
| Modify | Điều chỉnh dataset (thêm ảnh, fix label, đổi augmentation) rồi retrain | Quay lại từ bước Prepare — tốn thêm 1–2 ngày nếu cần ảnh thật | **Vòng lặp rất chậm**: mỗi lần sửa dataset cần lặp lại phần Prepare — ảnh thật tốn quá nhiều thời gian | **High** |
| Conclude | Benchmark trên real-camera test set; viết evidence report cho reviewer | Tự chụp test images; viết số bằng tay vào doc | Không có "golden test set" chuẩn để compare; khó thuyết phục reviewer về data quality | Med |

### Chốt 2 bước đau nhất

**Bước đau nhất #1:** Prepare — sinh và gán nhãn dataset, đặc biệt khi cần ảnh thật  
**Bước đau nhất #2:** Confirm — kiểm tra xem label sau realism/augmentation có còn đúng không

**Vì sao đây là nơi đáng chú ý nhất:**  
> **Prepare** là bottleneck thời gian: chụp ảnh thật + gán nhãn thủ công là lý do duy nhất khiến vòng lặp retrain mất ngày thay vì giờ. Nếu giải quyết được bước này mà vẫn giữ label tin cậy, toàn bộ vòng lặp nhanh lên 5–10x.  
> **Confirm** là bottleneck tin cậy: nếu không có automated QA, engineer không biết liệu dataset có tốt hơn hay đang train trên noise. Đây là lý do nhiều team abandon sim-to-real approach vì "không biết có đáng không".

---

## Bước 8 — Chỉ ra `AI Leverage Point`

### Bảng leverage point

| Step | AI nên giúp bằng cách nào? | Vì sao AI hợp ở đây? | Rủi ro chính nếu dùng AI |
|---|---|---|---|
| Prepare | SD1.5 + IP-Adapter + ControlNet Depth+Canny biến ảnh sim thành ảnh trông gần thật (texture, material, lighting realistic hơn) trong vài giây/ảnh, với label tự động từ sim | AI generative làm được "reality-fy" ảnh mà không đòi hỏi camera thật; cost/image thấp; reproducible từ seed | Generative render có thể làm lệch geometry vật thể → label sai âm thầm; cần QA gate để phát hiện |
| Confirm | Grounding DINO zero-shot re-detect trên ảnh sau realism → so IoU với bbox sim gốc → tự động accept/flag/reject | Zero-shot detection không cần train; có thể chạy automated batch QA trên toàn bộ dataset sau mỗi realism pass | DINO có thể fail trên vật thể bị che khuất nhiều hoặc có texture giống background → false accept |

### Kết luận nhanh

**AI leverage point quan trọng nhất của dự án tôi là:**  
> Bước **Prepare**: dùng generative AI (SD1.5 + ControlNet) để biến ảnh sim thành ảnh gần-thật, kết hợp với Grounding DINO tự động kiểm tra label preservation — xóa bỏ sự cần thiết của phần lớn chụp ảnh thật thủ công trong vòng lặp đầu tiên.

**Vì sao không phải ở bước khác:**  
> Bước Define và Execute không phải là nơi AI tạo ra giá trị khác biệt — engineer đã có tool tốt cho train/monitor (YOLO CLI, wandb). Giá trị không thể thay thế của AI nằm ở chỗ nó tạo được ảnh gần-thật từ ảnh sim (không thể làm bằng classical augmentation) và tự động kiểm tra label (không thể làm bằng tay trên scale).

---

## Bước 9 — Viết `Product Hypothesis`

### Bản hypothesis của tôi

> Nếu chúng ta giúp **Robotics Perception Engineer** tạo dataset labeled cho tabletop YCB objects tốt hơn ở bước **Prepare + Confirm**,  
> bằng cách **sinh ảnh sim → generative realism (SD1.5 + ControlNet) → Grounding DINO QA gate tự động**,  
> thì họ sẽ chuyển từ **chụp ảnh thật + gán nhãn thủ công** sang **pipeline AI-assisted này**,  
> vì **giảm từ 2–5 ngày xuống còn vài giờ để có dataset mới, với bằng chứng label quality có thể trình bày với reviewer**.

### Tín hiệu sớm nếu hypothesis này đúng

1. Engineer chạy pipeline lần đầu và có dataset QA-passed trong cùng buổi, không phải cùng tuần
2. mAP trên Golden Dataset của model train từ gen_realism dataset cao hơn sim_only baseline một cách nhất quán (xác nhận real-camera value, không chỉ "ảnh đẹp hơn")

---

## Bước 10 — Liệt kê `Assumptions to Validate`

### Bảng assumptions

| Assumption | Vì sao assumption này rủi ro? | Tôi đang có bằng chứng gì? | Cần validate bằng cách nào tiếp theo? |
|---|---|---|---|
| A1 | Generative realism thực sự giúp mAP tăng trên real-camera test, không chỉ làm ảnh đẹp hơn | Rủi ro cao: "đẹp" và "useful for training" là hai thứ khác nhau | Benchmark đã chạy: gen_realism +0.044 vs sim_only trên Golden; nhưng sample size còn nhỏ (245 ảnh/7 objects) | Chạy thêm với more objects + varied backgrounds; tách riêng benchmark cho từng class |
| A2 | Grounding DINO IoU gate đủ reliable để phát hiện label drift sau realism — không bỏ sót false accept | Nếu DINO miss nhiều false accept, dataset bị nhiễu âm thầm và mAP sẽ không ổn định | Threshold 0.85/0.70 được chọn empirically; reject rate 15–20% nghe hợp lý nhưng chưa validate edge cases | Test trên intentionally corrupted images (bbox shifted ±20%) để đo DINO sensitivity/specificity |
| A3 | Perception Engineer thực sự bị kẹt ở data bottleneck đủ thường xuyên để cần một tool riêng cho việc này | Nếu họ chỉ setup scene một lần và không đổi, pain không đủ lớn | Bằng chứng học thuật (YCB-Video paper, Tobin 2017) xác nhận pain; nhưng chưa interview user thật | Phỏng vấn 3–5 Perception Engineer thật — hỏi "lần gần nhất bạn cần tạo/update dataset là khi nào và mất bao lâu?" |
| A4 | Engineer sẵn sàng chấp nhận automated QA thay vì manual review cho dataset training | Nếu họ không tin QA gate, sẽ vẫn phải visual inspect tất cả → pipeline chỉ tiết kiệm được bước Prepare, không tiết kiệm Confirm | Chưa có user feedback thật; chỉ có team assumption | Cho engineer thật chạy pipeline và observe xem họ có re-inspect sau QA gate không |
| A5 | Lát cắt 7 YCB objects đủ represent use case thật của target user | YCB là dataset chuẩn học thuật; startup thật có thể cần objects rất khác (custom parts, industrial components) | YCB cho phép benchmark nghiêm ngặt (có mesh + ảnh thật chuẩn); nhưng generalizability chưa test | Test pipeline với 1–2 non-YCB objects; đo xem reject rate và mAP pattern có giữ hay không |

### Assumption nguy hiểm nhất nếu tôi đang sai

> **A3** — nếu Perception Engineer không thực sự lặp lại vòng tạo dataset thường xuyên (chỉ setup một lần rồi thôi), thì toàn bộ hypothesis sụp đổ: pipeline này không tiết kiệm đủ thời gian để đáng dùng. Pain point cần phải lặp lại và đủ thường xuyên thì tool mới có value.

---

## Bước 11 — Share trong bàn (3')

### Mỗi người / mỗi nhóm chỉ nói 4 thứ

1. **Job executor của bạn là ai**
2. **Core JTBD của bạn là gì**
3. **Step đau nhất đang nằm ở đâu**
4. **AI leverage point + assumption rủi ro nhất là gì**

### Ghi nhanh sau khi nghe bàn phản biện

| Ý phản biện tôi nghe được | Nó chạm vào phần nào? | Tôi sẽ giữ / sửa gì? |
|---|---|---|
| | | |
| | | |
| | | |

---

## Bước 12 — Chốt version cuối sau thảo luận

### Sau khi nghe phản biện, tôi thay đổi gì?

- [ ] Giữ nguyên `job executor`
- [ ] Sửa `job executor`
- [ ] Giữ nguyên `core JTBD`
- [ ] Sửa `core JTBD`
- [ ] Giữ nguyên `AI leverage point`
- [ ] Sửa `AI leverage point`
- [ ] Giữ nguyên `product hypothesis`
- [ ] Sửa `product hypothesis`

### Vì sao tôi giữ / sửa?

> _______________________________________________  
> _______________________________________________

### Version cuối cùng tôi nộp

**Job executor:**  
> Robotics Perception Engineer tại startup hoặc lab nhỏ — người trực tiếp build, train, và validate YOLO detector cho tabletop object detection, không có data team riêng.

**Core JTBD:**  
> Có được dataset object detection đủ đa dạng và đủ tin cậy về label để detector chạy đúng trên camera thật — mà không mất nhiều ngày gán nhãn thủ công mỗi lần đổi vật thể hoặc scene.

**2 bước đau nhất trong workflow:**  
> (1) **Prepare** — chụp ảnh thật + gán nhãn thủ công là bottleneck thời gian chính, mất 2–5 ngày/object; (2) **Confirm** — không có automated QA để verify label vẫn đúng sau augmentation mạnh, khiến engineer không tin kết quả pipeline.

**AI leverage point chính:**  
> Bước Prepare: generative realism (SD1.5 + ControlNet Depth+Canny) biến ảnh sim thành ảnh gần-thật; kết hợp Grounding DINO QA gate tự động kiểm tra label preservation — xóa bỏ phần lớn nhu cầu chụp ảnh thật thủ công mà vẫn có evidence quality có thể trình bày.

**Product hypothesis:**  
> Nếu chúng ta giúp Robotics Perception Engineer tạo dataset labeled tốt hơn ở bước Prepare + Confirm bằng cách kết hợp generative realism (SD1.5 + ControlNet) với Grounding DINO QA gate tự động, thì họ sẽ chuyển từ chụp-label thủ công sang pipeline này — vì giảm từ 2–5 ngày xuống vài giờ trong khi vẫn có bằng chứng label quality đủ thuyết phục reviewer.

**Assumption cần validate đầu tiên:**  
> A3 — Perception Engineer thực sự lặp lại vòng tạo dataset đủ thường xuyên để một pipeline tiết kiệm thời gian tạo ra value đáng kể. Cần phỏng vấn 3–5 engineer thật để xác nhận frequency và severity của pain này trước khi đầu tư thêm vào pipeline.

---

## Checklist trước khi nộp

- [x] Tôi đã khoanh đúng 1 lát cắt cụ thể của dự án.
- [x] Tôi đã phân biệt được `job executor` với buyer / influencer.
- [x] `Core JTBD` của tôi không nhét solution vào câu.
- [x] Tôi đã viết đủ 3 `job stories`.
- [x] Tôi đã điền `JTBD lite map` và khoanh ra 2 bước đau nhất.
- [x] Tôi đã chỉ ra `AI leverage point` thay vì nhảy thẳng vào feature list.
- [x] Tôi đã ghi rõ `assumptions to validate`.
- [ ] Tôi đã sửa version cuối sau khi share trong bàn.

---

## Nếu còn thời gian / làm về nhà

- Phỏng vấn nhanh 1 Robotics Perception Engineer thật để kiểm xem JS1 (deadline pressure → cần dataset trong giờ) hay JS2 (scene flexibility) là story sát nhất với pain thực tế.
- So sánh pipeline với current alternatives theo 3 tiêu chí: nhanh hơn (2–5 ngày → vài giờ ✓), rẻ hơn (pod cost/ảnh vs human labeler cost ✓ với caveats), tin hơn (DINO IoU gate vs visual inspection — chưa chắc user tin ngay).
- Câu hỏi khó: nếu không có generative realism, pipeline này — chỉ sim + domain randomization + DINO QA — có đủ giá trị không? Nếu câu trả lời là "có", thì AI contribution là incremental improvement, không phải enabler. Nếu "không", thì generative realism là core moat và cần bảo vệ quality của nó.
