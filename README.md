# Post-Quantum Blockchain: Thử nghiệm chữ ký Dilithium / Falcon cho Smart Contract.
Students:
  - Nguyễn Trọng Nhân - 24521236. 
  - Lê Việt Hoàng - 24520546.

Lecturer: Nguyễn Ngọc Tự.
## Goals
Hiểu được cơ chế chữ ký hậu lượng-tử (Dilithium-Falcon), đánh giá tính khả thi của việc áp dụng các mô hình chữ ký trong môi trường blockchain (các rủi ro thực thi, ví dụ sampling, constant-time).

So sánh ba mô hình triển khai:
  - On-chain verification (hợp đồng thông minh trực tiếp kiểm tra chữ ký).
  - Off-chain verification + on-chain attestation (giảm gas bằng cách chỉ lưu bằng chứng trên chain).
  - Zk-proof assisted verification (dùng SNARK/Plonk để xác minh hiệu quả hơn).

Xây dựng prototype và đo đạc thực tế: gas fee, độ trễ, băng thông, lưu trữ.
Viết báo cáo khoa học, cung cấp mã nguồn reproducible và khuyến nghị cho áp dụng PQ signatures trong hệ thống blockchain.

