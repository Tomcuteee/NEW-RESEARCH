# PHÂN TÍCH NGUY CƠ BẢO MẬT TRONG HẠ TẦNG MẠNG ẢO HÓA (NFV) VÀ 5G
**English Title:** *Experimental Analysis of Security Threats in NFV Infrastructure: Supply Chain Attack Simulation and Defensive Mechanisms*

---

##  Tóm tắt (Abstract)
Network Functions Virtualization (NFV) là công nghệ nền tảng trong hạ tầng mạng 5G, cho phép chuyển đổi các chức năng mạng từ thiết bị phần cứng chuyên dụng sang phần mềm chạy trên máy chủ tiêu chuẩn. NFV mang lại sự linh hoạt, giảm chi phí và hỗ trợ triển khai dịch vụ nhanh chóng. Tuy nhiên, việc phụ thuộc vào các Virtual Network Functions (VNF) cũng mở rộng bề mặt tấn công, đặc biệt là nguy cơ **tấn công chuỗi cung ứng**.  
Bài báo này trình bày kiến thức cơ bản về NFV, kiến trúc chuẩn ETSI, cách triển khai thực tế trong môi trường mô phỏng, và phân tích các lỗ hổng bảo mật tiềm tàng khi ứng dụng công nghệ này.

**Keywords:** NFV, 5G Security, Supply Chain Attack, VNF Integrity, ETSI Architecture

---

## 1. Giới thiệu (Introduction)
Trong mạng viễn thông truyền thống, các chức năng như **Firewall, Router, Load Balancer** được triển khai dưới dạng thiết bị phần cứng chuyên dụng. Mô hình này có nhiều hạn chế: chi phí đầu tư cao, khó nâng cấp, và thiếu linh hoạt khi nhu cầu lưu lượng thay đổi.  

NFV ra đời để giải quyết vấn đề này bằng cách **phần mềm hóa hạ tầng mạng**. Thay vì mua thiết bị vật lý, nhà mạng chỉ cần triển khai phần mềm VNF trên máy chủ tiêu chuẩn. Đây là bước ngoặt công nghệ, đặc biệt trong mạng 5G, nơi NFV là nền tảng bắt buộc.

---

# 📘 NFV Background & Security Analysis

## 2. Kiến thức nền về NFV (Background)

### 2.1. Khái niệm NFV
**Network Functions Virtualization (NFV)** là công nghệ cho phép chuyển đổi các chức năng mạng truyền thống (Firewall, Router, IPS…) từ thiết bị phần cứng chuyên dụng sang phần mềm chạy trên máy chủ x86 tiêu chuẩn.  

- **Ý tưởng cốt lõi:** Tách phần mềm VNF khỏi phần cứng vật lý  
- **Lợi ích chính:**
  - Giảm chi phí CAPEX/OPEX  
  - Tăng khả năng mở rộng và triển khai nhanh  
  - Dễ dàng nâng cấp, thay thế hoặc mở rộng tính năng  
  - Tự động hóa bằng MANO  

---

### 2.2. NFV và SDN (mối quan hệ quan trọng)

NFV thường được triển khai kết hợp với **SDN (Software-Defined Networking):**

| Công nghệ | Mục tiêu | Vai trò |
|-----------|----------|---------|
| **NFV**  | Ảo hóa chức năng mạng | Xử lý dịch vụ ở tầng VNF |
| **SDN**  | Tách Control-plane và Data-plane | Điều khiển đường đi của luồng gói tin |

SDN điều khiển luồng, NFV xử lý luồng → tạo thành hệ sinh thái **5G linh hoạt**.

---

### 2.3. Kiến trúc ETSI NFV (chuẩn hóa)

Theo ETSI, NFV bao gồm **3 khối chính**:

1. **NFVI (NFV Infrastructure)**  
   - Phần cứng vật lý: server, storage, switch  
   - Lớp ảo hóa:  
     - Hypervisor: KVM, VMware ESXi  
     - Container: Docker, Kubernetes  
   - Cung cấp CPU/RAM/Network cho VNF  

2. **VNF (Virtual Network Functions)**  
   - Các chức năng mạng dưới dạng phần mềm: vRouter, vFirewall, vIDS, UPF…  
   - Có khả năng: triển khai – scale – update – terminate  

3. **MANO (Management and Orchestration)**  
   - **NFVO:** Điều phối dịch vụ mạng  
   - **VNFM:** Quản lý vòng đời VNF  
   - **VIM:** Quản lý tài nguyên NFVI (OpenStack, VMware vCloud)  

---

### 2.4. Vòng đời của VNF (ETSI VNFM Lifecycle)

Các giai đoạn chính của VNF:  
1. Initialization  
2. Instantiation  
3. Configuration  
4. Scaling (In/Out/Up/Down)  
5. Updating / Patching  
6. Termination  

Malware trong tấn công **Supply Chain** thường được cấy tại bước **1, 4 hoặc 5**.

---

### 2.5. Kiến trúc tổng quan NFV (ASCII Diagram)
User Traffic ↓ +--------------------------+ |     VNF Chain (SFC)      | | vFirewall → vIPS → vRouter | +--------------------------+ ↓ +--------------------------+ |          NFVI            | | Compute | Storage | Network | +--------------------------+ ↓ +--------------------------+ |           MANO           | | NFVO | VNFM | VIM        | +--------------------------+

---

### 2.6. Tính thực tiễn trong mạng 5G

- **5G Core là NFV-based**  
- Thành phần chính của 5G Core đều là VNF/CNF:  
  - **AMF:** Access Management Function  
  - **SMF:** Session Management Function  
  - **UPF:** User Plane Function (xử lý data plane tốc độ cao)  

**Control Plane vs Data Plane:**  
- Control-plane: SMF, AMF → điều khiển phiên kết nối  
- Data-plane: UPF → xử lý lưu lượng người dùng  

Tấn công vào **UPF** có thể nghe lén/chuyển hướng toàn bộ traffic.  

**Network Slicing:**  
- Cho phép tạo các slice độc lập cho từng dịch vụ  
- Ví dụ: slice cho xe tự hành, IoT, video streaming  

---

### 2.7. Cơ chế SFC (Service Function Chaining)

- **Điều khiển bởi:**  
  - SDN Controller (OpenDaylight, ONOS)  
  - NSH (Network Service Header) – gắn metadata để phân biệt từng luồng  

**Ví dụ chuỗi dịch vụ:**  
User → vFirewall → vIDS → vRouter → Internet

** Nếu hacker chiếm một VNF, họ có thể thay đổi thứ tự hoặc chèn VNF độc hại. **

---

### 2.8. Setup NFV trong môi trường mô phỏng

**Hạ tầng:**  
- 1 PC làm NFVI (VMware Workstation / ESXi)  
- 1 laptop làm MANO hoặc máy tấn công  
- 1 router vật lý  

**Các bước:**  
1. Tạo máy ảo và triển khai vRouter + vFirewall  
2. Cài OpenStack hoặc MANO đơn giản để điều phối  
3. Thiết lập Service Function Chain  
4. Mô phỏng tấn công Supply Chain bằng cách thay đổi file VNF  
5. Quan sát hành vi bất thường (reverse shell, log sai lệch…)  

---

## 3. Phân tích lỗ hổng bảo mật (Security Analysis)

### 3.1. Lỗ hổng tại NFVI
- VM Escape, tấn công hypervisor  
- Driver/firmware chưa được vá  
- Container escape (nếu dùng Kubernetes)  

### 3.2. Lỗ hổng tại MANO
- Lỗ hổng API NFVO/VNFM  
- Sai cấu hình phân quyền → chiếm quyền quản trị  
- Thao túng orchestration dẫn đến scale/instantiate VNF trái phép  

### 3.3. Lỗ hổng tại VNF
- VNF là file ảnh/VM tải từ bên ngoài  
- Dễ bị thay đổi hoặc cấy mã độc  
- Khó phát hiện nếu attacker chỉnh sửa nhẹ  
- Khi bị chiếm: có thể theo dõi hoặc chuyển hướng traffic toàn mạng  

### 3.4. Kịch bản tấn công chuỗi cung ứng (Supply Chain Attack)
1. **Xâm nhập nguồn phân phối VNF:** Hacker chiếm kho chứa hoặc chèn MITM khi tải file  
2. **Chỉnh sửa file VNF:** Gắn backdoor, Trojan hoặc reverse shell vào image  
3. **Đóng gói lại update hợp lệ:** Doanh nghiệp triển khai mà không hay biết  
4. **Kích hoạt:** VNF độc hại gửi traffic ngược về C2, vượt qua firewall vì nằm ngay trong chuỗi xử lý  

---
## 4. Demo (mô phỏng 1 cuộc tấn công)





--- 
## 5. Kết luận (Conclusion)
NFV là nền tảng bắt buộc trong hạ tầng 5G, mang lại nhiều lợi ích về chi phí và linh hoạt. Tuy nhiên, nó cũng mở rộng bề mặt tấn công, đặc biệt ở khâu chuỗi cung ứng VNF. Việc nghiên cứu, mô phỏng và kiểm thử các kịch bản tấn công là cần thiết để xây dựng cơ chế phòng vệ hiệu quả, đảm bảo tính toàn vẹn và an toàn cho hạ tầng viễn thông quốc gia.

---

## Tài liệu tham khảo (References)
1. ETSI NFV Architecture Overview.  
2. Schardong et al., “Survey on VNF Placement and Chaining,” 2021.  
3. Nguyễn Văn Quân, “Khóa luận tốt nghiệp về NFV Optimization,” 2023.  
4. 3GPP 5G Security Standards.  

---
