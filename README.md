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
2.  Dilithium (ML-DSA, FIPS 204):
    - Lattice-based, Fiat–Shamir with aborts.
    - Dễ triển khai an toàn, không yêu cầu floating-point.
    - Nhược điểm: chữ ký lớn (2.4 KB), public key ~1.3 KB.
3. Falcon (FN-DSA draft):
    - Lattice-based, Gaussian sampling (FFT).
    - Ưu điểm: chữ ký nhỏ (~666 B), public key ~897 B, verify nhanh.
    - Nhược điểm: implement phức tạp, dễ side-channel nếu dùng floating-point.
4. Thách thức:
    - Blockchain hạn chế bởi gas, calldata, storage.
    - Chữ ký PQ có thể gây tăng chi phí gấp nhiều lần so với ECDSA.
    - Cần tìm phương án kết hợp: off-chain + zk-proof.
## Research Questions. 
1. Chi phí (gas, bytes lưu trữ, latency) để xác thực Dilithium/Falcon on-chain so với ECDSA là bao nhiêu trên EVM/Solidity và trên nền tảng WASM (Substrate/CosmWasm)?
2. Có những thiết kế hybrid/auxiliary (off-chain verification + on-chain attestation, hoặc zk-proof of verification) nào giúp giảm chi phí on-chain mà vẫn giữ được tính bảo mật/ khả năng audit không?
3. Có sự khác biệt thực tiễn trong việc triển khai Dilithium với Falcon (cân nhắc: kích thước chữ ký, công đoạn sampling/FFT, constant-time difficulty) ảnh hưởng tới lựa chọn cho blockchain hay không?
* Giả thuyết: Triển khai verify trực tiếp trên EVM sẽ tốn gas và có thể không thực tế; WASM-based smart contract hoặc off-chain + attestation/zk-proof là các hướng thực nghiệm khả thi hơn. Falcon có chữ ký nhỏ hơn thi an toàn hơn nhưng chữ ký có thể lớn hơn. 
