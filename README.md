# Post-Quantum Blockchain: Thử nghiệm chữ ký Dilithium / Falcon cho Smart Contract.
Students:
  - Nguyễn Trọng Nhân - 24521236. 
  - Lê Việt Hoàng - 24520546.

Lecturer: Nguyễn Ngọc Tự.

Thời gian: 12 tuần (Semester 1, 2025-2026).
## Goals
Hiểu được cơ chế chữ ký hậu lượng-tử (Dilithium-Falcon), đánh giá tính khả thi của việc áp dụng các mô hình chữ ký trong môi trường blockchain (các rủi ro thực thi, ví dụ sampling, constant-time).

So sánh ba mô hình triển khai:
  - On-chain verification (hợp đồng thông minh trực tiếp kiểm tra chữ ký).
  - Off-chain verification + on-chain attestation (giảm gas bằng cách chỉ lưu bằng chứng trên chain).
  - Zk-proof assisted verification (dùng SNARK/Plonk để xác minh hiệu quả hơn).

Xây dựng prototype và đo đạc thực tế: gas fee, độ trễ, băng thông, lưu trữ.

Viết báo cáo khoa học, cung cấp mã nguồn reproducible và khuyến nghị cho áp dụng PQ signatures trong hệ thống blockchain.
## Senario.

## Assets to Protect. 
1. Tính toàn vẹn và xác thực giao dịch trên blockchain.
2. Chi phí vận hành hợp đồng thông minh (gas, storage).
3. Khóa riêng tư (private key) của người dùng.
## Stakeholders.
1. Nhà phát triển blockchain/ smart contract: cần giải pháp triển khai hiệu quả, an toàn.
2. Người dùng blockchain: mong muốn giao dịch chi phí thấp, độ trễ nhỏ.
3. Tổ chức tài chính/ quản lý tài sản số: cần tính bền vững lâu dài trước nguy cơ lượng tử.
## Motivation & Background.
1. Thực trạng:
    - Ethereum, Bitcoin: dùng ECDSA (secp256k1).
    - Polkadot/Substrate: dùng sr25519 (EdDSA trên Ed25519).
    - Cosmos: hỗ trợ secp256k1, Ed25519.
        * Tất cả đều dựa trên elliptic curve signatures, vốn sẽ bị phá bởi thuật toán Shor trên máy tính lượng tử quy mô lớn.


