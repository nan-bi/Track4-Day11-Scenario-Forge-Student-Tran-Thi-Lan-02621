MSSV: 2A202602621

# Lab 11 - Scenario Forge - bao cao

Thi tran cua em: `meta.json: _run.town = D`. Evidence duoc tao bang `LAB11_MSSV=2A202602621`, khong dung `--quick`.

## 1. Phat lai log co trung thuc khong?

`replay.json`: `n = 400`, `dropped_human_crash_logs = 0`, `n_used = 400`. Planner tren log phat lai co `replay.crashes = 91`, gom `rear = 84`, `side = 7`; trong the gioi phan ung chi co `reactive.crashes = 1`, `front = 1`; `crash_in_both = 0`.

Phan lon va cham replay la `rear` vi xe sau trong log khong biet planner se phanh som hon nguoi lai that, nen no van chay theo quy dao da ghi va dam duoi ego. Day la va cham gia cua replay, khong phai hanh vi trung thuc cua the gioi phan ung. Vi du `replay_only[0]`: scenario 0 bi `rear`, nhung khi chay reactive thi `min_ttc_reactive = 2.789 s`, tuc la tac nhan phia sau co phan ung va khong con dam nhu replay.

## 2. Kich ban bat kha thi

`is_feasible` loai 4 nhom: doi lan qua gap can gia toc ngang lon hon `mu*g`; phanh sau khi nhap lan `cutBrake > mu*g`; nguoi di bo chay nhanh hon `2.5 m/s`; xe cat lan co `cutV <= 0` hoac `cutV > 22 m/s`. Day deu la gioi han vat ly/hanh vi: ma sat duong gioi han phanh va gia toc ngang, nguoi di bo khong nen duoc mo phong nhu van dong vien chay, va xe cat lan khong duoc dung yen hoac chay qua nhanh ngoai bai toan.

`hunt.json`: bat kha thi nhieu nhat la `crosswalk_block` va `rear_ended`, moi hunt co `infeasible = 11`; `bus_bypass = 3`, `cut_in = 3`. CEM bi keo ve do vi cac diem nguy hiem cao thuong nam gan bien vat ly: doi lan qua nhanh, phanh qua muc, hoac hanh vi nguoi di bo/xe khong thuc. Neu dem cac case bat kha thi thanh va cham, uoc luong se bi thoi phong boi nhung tinh huong ngoai doi khong sinh ra duoc.

## 3. Monte Carlo cho biet gi

`mc.json`: `crashes = 3`, `n = 3000`, `p = 0.001`, Wilson 95% `[0.000340, 0.002936]`. Cac mode thay duoc: `crosswalk_block = 1`, `bus_bypass = 2`.

Muon Wilson hep con khoang +-20% cua p thi can xap xi `96 / p`, voi p = 0.001 la khoang 96,000 lan chay. Cac mode chua xuat hien trong 3000 lan la `cut_in`, `rear_ended`, `other`; khong duoc bao 0, ma bao can tren Wilson: `cut_in.hi = 0.001279`, `rear_ended.hi = 0.001279`, `other.hi = 0.001279`.

## 4. Importance sampling, ESS va suy bien

`estimate.json: budget = 16800`. Bang tom tat theo seed:

| seed | total | bus_bypass p/hits/ess/top | cut_in | crosswalk_block p/hits/ess/top | rear_ended |
|---|---:|---|---|---|---|
| 0 | 1.613e-3 | 1.613e-3 / 28 / 126.2 / 0.436 | 0 hit, upper 0.025 | 2.988e-7 / 28 / 120.2 / 0.331 | 0 hit, upper 0.025 |
| 1 | 5.550e-4 | 5.543e-4 / 29 / 134.5 / 0.578 | 0 hit, upper 0.025 | 7.146e-7 / 28 / 127.1 / 0.437 | 0 hit, upper 0.025 |
| 2 | 4.869e-4 | 4.865e-4 / 37 / 119.8 / 0.318 | 0 hit, upper 0.025 | 4.409e-7 / 41 / 124.1 / 0.420 | 0 hit, upper 0.025 |

Ba seed khong dong y hoan toan: seed 0 cao hon seed 1-2 khoang 3 lan, chu yeu do `bus_bypass`. MC cho `p = 1.0e-3`, nam trong cung bac do lon voi IS nhung MC rat thua mau. `top_share > 0.5` o `bus_bypass` seed 1, nen con so seed do bi chi phoi manh boi mot mau nang ky. `other` lay tu `mc.json` (`other.k = 0`, `hi = 0.001279`) vi khong co mot ham danger rieng de san tat ca va cham linh tinh.

## 5. Cac kieu that bai

`hunt.json`: hunt `bus_bypass` tim dung `bus_bypass = 105`; hunt `crosswalk_block` tim dung `crosswalk_block = 13`. Hunt `cut_in` lai tim `bus_bypass = 14`, va hunt `rear_ended` tim `crosswalk_block = 7`, nen hai hunt nay khong tim dung mode muc tieu trong budget nay.

Chuoi `bus_bypass`: C2 chen qua xe buyt dung gan vach, ego thay vat can/xe cat lan muon, khoang cach phanh con lai rat nho va cham o phia truoc. Chuoi `crosswalk_block`: nguoi di bo di ra tu vung bi xe buyt che; ego phanh nhung than xe da dung tren vach, chan duong nguoi di bo nen tinh la failure. Neu chi chay mot hunt chung, CEM se co xu huong co ve mode de tim nhat (`bus_bypass`) va lap lai no, nen can hunt rieng tung mode.

Anh 3D can chup tu `evidence/viewer.json` trong chapter 9; file export da tao `viewer.json` voi 11 run va 6000 cloud points.

## 6. Sua planner

`fix.json`: policy duoc chon la `{'cutinLat': 3.0}`. Tren TRAIN: `fail = 5.792e-6`, `fail_hi = 1.598e-5`, `progress = 217.82 m`, `harsh = 7.33`, `unseen = ['bus_bypass']`.

Tren TEST: chosen co `fail = 2.128e-7`, `fail_hi_seen = 3.830e-7`, `fail_hi = 3.878e-2`, `progress = 216.90 m`, `harsh = 7.51`, `unseen = ['bus_bypass']`. Baseline TEST co `fail = 2.094e-4`, `fail_hi_seen = 4.020e-4`, `fail_hi = 1.681e-3`, `progress = 216.94 m`, `harsh = 7.87`.

`fail_hi` cua chosen lon hon `fail` cua baseline vi mode `bus_bypass` khong co hit tren TEST cua policy moi, nen phai cong can tren zero-hit `upper_zero_hits = 0.0375`; do la can tren bao thu, khong phai diem uoc luong. Mot num nghe an toan hon nhung phan tac dung la `T = 2.0`: TRAIN `fail_hi = 1.504e-3`, cao hon baseline train, vi giu khoang cach lon hon co the lam tac nhan khac phan ung/chen lan khac di va tao failure moi.

## 7. Tu mo phong ra duong that

Ba gia dinh anh huong lon:

1. `cutGap` va hanh vi chap nhan khe cua xe cat lan: can du lieu camera/lidar tu xe that ve khoang cach xe chen lan chap nhan theo toc do va mat do giao thong. Neu gia dinh qua "hien", xac suat `bus_bypass/cut_in` bi thap.
2. `pedLook` va `pedGap` cua nguoi di bo bi xe buyt che: can log nguoi di bo tai tram xe buyt/vach qua duong co che khuat. Neu nguoi di bo it ngo hon gia dinh, rui ro `crosswalk_block` se cao hon.
3. `wet`/`mu` va phanh: can du lieu thoi tiet, mat duong, ABS/IMU ve gia toc phanh kha dung. Neu `mu` thuc te thap hon, ca ego va xe khac phanh kem hon, tang failure o cac mode gan va cham.

## 8. Gioi han

Xac suat nay chi noi ve mot the gioi mot con pho voi tac nhan IDM/civil don gian, khong bao quat tat ca do thi that. Cam bien gan nhu hoan hao ngoai che khuat boi xe buyt, nen khong tinh loi nhan thuc, false positive/false negative. Planner chi duoc sua bang vai num IDM/ped/cutin, khong danh gia logic moi nhu doi lan tranh xe dap hay du doan y dinh nhieu tac nhan.

Neu co them mot tuan, em se do nhieu scene co occlusion khac, them sensor noise/perception latency, va chay nhieu hon cho cac mode zero-hit de ha can tren.

## 9. Rac roi cua em

Rac roi: `hazard.json` them `bike` H0, xe dap o lan ngoai `E2` lang trai de tranh o ga/xe dung ven duong. Can cu: planner hien chu yeu phanh theo vat can phia truoc va nguoi di bo/vach che khuat, nen xe dap lang ngang tu mep lan co the la tinh huong kho.

Gia dinh: `space = {'h0_x':[90,150], 'h0_d':[0.8,1.4], 'h0_v':[4,7], 'h0_swerve':[0.8,1.5], 'h0_swDur':[1.6,2.5], 'h0_trig':[16,34]}`, `exposure = 0.03`. `mine.json: accepted = 0.9947`, `infeasible = 8`; sau khi loai overlap luc bat dau, moi policy co `n = 1155`.

Ket qua `mine.json`: baseline `crashes = 0`, `avoidable = 0`, `unavoidable = 0`, `unattributed = 0`, `low_speed = 0`, Wilson `p = 0`, `[lo, hi] = [0, 0.003315]`, `per_trip_hi = 9.945e-5`. Chosen cung giong baseline: `crashes = 0`, `avoidable = 0`, `unavoidable = 0`, `unattributed = 0`, `low_speed = 0`, `per_trip_hi = 9.945e-5`.

Voi range nay planner xu ly tot, nen khong co va cham de chup trong mine. Chuan `avoidable` la lac quan vi oracle phanh toi da ngay khi thay hazard, nhung cung bi han che vi chi thu phanh, khong thu danh lai. Neu muon lam planner manh hon cho tinh huong xe dap, can them logic du doan lateral swerve cho xe dap/xe hai banh va giam toc khi doi tuong mong manh o lan ben co xu huong cat vao lan ego; chi tang `cutinLat` khong du vi num do thiet ke cho xe C2, khong rieng xe dap.
