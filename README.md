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

Giả thuyết: Triển khai verify trực tiếp trên EVM sẽ tốn gas và có thể không thực tế; WASM-based smart contract hoặc off-chain + attestation/zk-proof là các hướng thực nghiệm khả thi hơn. Falcon có chữ ký nhỏ hơn thi an toàn hơn nhưng chữ ký có thể lớn hơn. 
## Methodology. 
1. Thiết kế kịch bản triển khai
  - On-chain verification (EVM/Solidity): port trực tiếp thuật toán verify.
  - On-chain verification (WASM/Polkadot): compile Rust → WASM, chạy trên Substrate.
  - Off-chain verification + on-chain attestation: chỉ đưa Merkle root/hash lên chain.
  - zk-proof assisted verification: tạo zk-SNARK proof ngoài chuỗi, verify proof nhỏ trên chain.
2. Implementation plan.
  - Libraries: PQClean, liboqs, pqcrypto (Rust).
  - Blockchain platforms: Hardhat/Ganache (EVM), Substrate dev node, CosmWasm.
  - zk tools: Circom/snakjs, Halo2.
  - Bechmark: đo gas, calldata size, storage, latency, proof gen time.
## Assets & Security Requirements. 
1. Assets cần bảo vệ.
  - Khóa riêng tư người dùng.
  - Tính toàn vẹn giao dịch blockchain.
  - Chi phí gas & storage (tài nguyên on-chain).
2. Yêu cầu bảo mật khi triển khai.
  - Constant-time implementation (không branch theo secret).
  - Randomness chuẩn CSPRNG.
  - Side-channel hardened (đặc biệt là Falcon).
  - Input validation: kiểm tra signature/public key length.
  - Replay protection cho attestation (nonce/timestamp/chain-id).
3. Checklist an toàn.
  - Dilithium: rejection sampling, SHAKE-128/256 chuẩn.
  - Falcon: integer-based sampler, tránh floating point không constant-time.
  - Blockchain: hash thay vì lưu trực tiếp chữ ký.
## Experiment Setup.
1. Key generation: tạo keypairs Dilithium & Falcon theo chuẩn Nist.
2. Test vectors: verify với bộ chuẩn NIST PQC.
3. Blockchain testnest: Ethereum local, Substrate dev chain.
4. Benchmark metrics:
   - Gas per verify.
   - Transaction size (bytes).
   - Storage overhead.
   - Proof generation time (zk).
## Deliverables.
1. Báo cáo chi tiết + so sánh Dilithium vs Falcon.
2. Prototype contracts (EVM, WASM).
3. Off-chain attestation server.
4. zk-proof circuit (demo).
5. Bộ dữ liệu benchmark (CSV, plots).
6. Demo video + repoducible repo (Docker).
## Riks & Mitigation.
1. Falcon side-channel → chỉ dùng implementation đã hardened.
2. Gas EVM quá cao → fallback sang attestation/zk.
3. zk circuit quá phức tạp → thử nghiệm sub-circuit hoặc batching.
4. Thay đổi chuẩn NIST → ghi rõ commit hash, version.

## References.
1. [NIST FIPS 204 — ML-DSA (Dilithium).](https://nvlpubs.nist.gov/nistpubs/fips/nist.fips.204.pdf)
2. [Cloudflare (2024) — A look at the latest post-quantum signature standardization candidates](https://blog.cloudflare.com/another-look-at-pq-signatures/)
3. [Benchmarking and Analysing NIST PQC Lattice-Based Signature Scheme Standards on the ARM Cortex M7](https://csrc.nist.gov/csrc/media/Events/2022/fourth-pqc-standardization-conference/documents/papers/benchmarking-and-analysiing-nist-pqc-lattice-based-pqc2022.pdf)
4. [CRYSTALS-Dilithium Algorithm Specifications and Supporting Documentation](https://pq-crystals.org/dilithium/data/dilithium-specification-round3.pdf)
