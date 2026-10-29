# Viterbi Decoder for Digital Communication Systems

## Tổng quan (Summary)

Repository này chứa toàn bộ mã nguồn, kịch bản mô phỏng và kết quả thực nghiệm cho đề tài **"Hardware Implementation and Performance Evaluation of Viterbi Decoder for Digital Communication Systems"**. 

Dự án bao gồm:
- Mô hình tham chiếu (MATLAB) cho toàn bộ hệ thống: Mã hóa tích chập, điều chế BPSK, kênh AWGN, giải mã Viterbi Hard/Soft.
- Thiết kế RTL (Verilog) cho bộ giải mã Viterbi (Soft và Hard) tối ưu hóa phần cứng.
- Hệ thống Testbench tự động (Self-checking) và quy trình kiểm chứng File-based Cosimulation (RTL vs MATLAB).
- Kết quả tổng hợp FPGA (Synthesis) trên dòng chip Cyclone V.

**Cấu hình mã:** Convolutional code tốc độ 1/2, K = 3, đa thức sinh (7,5)₈, 4 trạng thái, Traceback depth = 12.

## Mục lục
- [1. Cấu trúc Repository](#1-cấu-trúc-repository)
- [2. Yêu cầu hệ thống](#2-yêu-cầu-hệ-thống)
- [3. Hướng dẫn tái tạo kết quả (Reproducibility)](#3-hướng-dẫn-tái-tạo-kết-quả)
- [4. Kết quả thực nghiệm](#4-kết-quả-thực-nghiệm)
- [5. Đánh giá Trade-off phần cứng](#5-đánh-giá-trade-off-phần-cứng)

## 1. Cấu trúc Repository

```text
viterbi-decoder/
├── README.md
├── docs/
│   ├── midterm.pdf            # Báo cáo tiến độ
│   └── figures/               # Các hình ảnh trong báo cáo
│       ├── chapter1/          # Hình lý thuyết
│       ├── chapter2/          # Hình mô phỏng MATLAB (fig8_ber_comparison.png)
│       ├── chapter3/          # Hình kiến trúc RTL
│       └── chapter4/          # Hình kiểm chứng (waveform, BER plot)
├── matlab/
│   ├── plots/                 # Đồ thị BER xuất ra
│   └── Viterbi_Decoder_Project.m # Script mô phỏng BER 1 triệu bit (Chương 2)
├── simulink/
│   ├── plots/                 # Kết quả Cosim RTL
│   └── cosim_rtl_ber_sweep.m  # Script điều phối ModelSim chạy 1 triệu bit
├── verilog/
│   ├── viterbi_soft_decoder.v # RTL bộ giải mã Soft (PIPELINE=1, NORMALIZE=1)
│   ├── viterbi_hard_decoder.v # RTL bộ giải mã Hard
│   ├── tb_viterbi_soft_decoder.v # Testbench Unit Test (42 bit)
│   ├── tb_viterbi_hard_decoder.v # Testbench Unit Test (42 bit)
│   ├── tb_viterbi_soft_decoder_cosim.v # Testbench đọc/ghi file
│   └── tb_viterbi_hard_decoder_cosim.v # Testbench đọc/ghi file

## 2. Yêu cầu hệ thống
Công cụ	Mục đích sử dụng
MATLAB (R2025a)	Chạy mô hình tham chiếu, tính BER lý thuyết và điều phối Cosim.
Communications Toolbox	Sử dụng các hàm convenc, vitdec, poly2trellis.
ModelSim/QuestaSim	Mô phỏng RTL và chạy testbench.
Quartus Prime (Lite)	Tổng hợp thiết kế trên FPGA Cyclone V (5CGXFC9E7F35C8).

## 3. Hướng dẫn tái tạo kết quả
Để tái tạo lại 100% các số liệu và đồ thị trong báo cáo, hãy thực hiện theo 3 bước sau:

## 3.1. Mô phỏng mô hình tham chiếu MATLAB
-Mở MATLAB, di chuyển đến thư mục matlab/.
-Mở file Viterbi\_Decoder\_Project.m.
-Đảm bảo dòng cấu hình N\_bits = 1e6; (1.000.000 bit).
-Nhấn Run.

Kết quả: Cửa sổ Command Window in ra bảng BER tại các mức Eb/N0. Đồ thị BER được lưu tự động vào thư mục plots/.

## 3.2. Kiểm chứng chức năng RTL (Unit Test)
-Mở ModelSim/QuestaSim, trỏ đến thư mục verilog/.
-Trong cửa sổ Transcript, chạy các lệnh sau:
vlib work
vlog viterbi\_soft\_decoder.v tb\_viterbi\_soft\_decoder.v
vsim work.tb\_viterbi\_soft\_decoder
run -all

Kết quả: Console in ra dòng bits checked = 42, errors = 0 -> PASS. Mở cửa sổ Wave để xem waveform giao thức.

## 3.3. Kiểm chứng so sánh RTL vs MATLAB (File-based Cosim)
-Mở MATLAB, di chuyển đến thư mục simulink/.
-Mở file cosim\_rtl\_ber\_sweep.m (Đảm bảo cấu hình 1.000.000 bit/điểm).
-Nhấn Run.

Quá trình: Script tự động sinh file stimulus .txt, gọi ModelSim chạy ngầm (vsim -c), đọc file kết quả .txt và tính toán BER.

Kết quả: Bảng BER in ra Console, đồ thị BER lưu tại plots/cosim\_rtl\_ber.png.

## 4. Kết quả thực nghiệm
## 4.1. Kết quả Unit Test (42 bits)
Cả 2 phiên bản RTL đều vượt qua bài kiểm tra chức năng với 42 bit (bao gồm 40 bit dữ liệu + 2 bit kết thúc). Testbench tự động so sánh và xác nhận err\_count = 0.

## 4.2. So sánh BER RTL vs MATLAB (1.000.000 bit)
Dưới đây là kết quả đối chiếu BER tại các mốc Eb/N0 quan trọng, chạy trên luồng dữ liệu liên tục 1 triệu bit:

Eb/N0 (dB)	BER (MATLAB - Soft)	BER (RTL - Soft)	BER (MATLAB - Hard)	BER (RTL - Hard)
0.0		1.132e-1		9.54e-2			1.974e-1		1.98e-1
2.0		1.965e-2		1.55e-2			7.385e-2		7.42e-2
4.0		8.690e-4		6.71e-4			1.177e-2		1.20e-2
6.0		1.700e-5		8.00e-6			6.440e-4		6.90e-4

Nhận xét: RTL bám sát mô hình tham chiếu. Tại SNR cao, RTL Soft có BER tốt hơn một chút do nhận tín hiệu 8-bit (RTL) so với lượng tử hóa 3-bit (MATLAB).

## 5. Đánh giá Trade-off phần cứng
Kết quả tổng hợp (Synthesis) trên chip Cyclone V (Target 100 MHz, góc Slow 85°C):

Bộ giải mã	ALMs	Registers	Fmax (MHz)	Coding Gain (tại BER 1e-4)
Hard Decoder	346	192		23.64 MHz	1.55 dB
Soft Decoder	410	211		32.32 MHz	3.35 dB

-Diện tích: Bản Soft tốn thêm 64 ALMs (\~18%) do datapath rộng hơn.
-Hiệu năng: Bản Soft mang lại lợi thế \~1.8 dB so với bản Hard.
-Timing: Cả hai đều chưa đạt 100 MHz do giới hạn của vòng hồi tiếp ACS (critical path). Hướng khắc phục: Pipelining sâu hơn hoặc chuyển sang kiến trúc Register-Exchange.

## 6. Tiến độ hiện tại
Hạng mục						Trạng thái
Cơ sở lý thuyết						Hoàn thành
Mô hình MATLAB (Encoder, BPSK, AWGN, Hard/Soft Viterbi)	Hoàn thành
Khảo sát BER (1 triệu bit)				Hoàn thành
RTL Soft/Hard Decoder \& Testbench			Hoàn thành
Kiểm chứng RTL vs MATLAB (Cosim)			Hoàn thành (Bit-exact)
Tổng hợp FPGA \& Đo Fmax/ALMs				Hoàn thành
Đánh giá Traceback depth \& Quantization		Đã thảo luận lý thuyết (mở rộng trong tương lai)

