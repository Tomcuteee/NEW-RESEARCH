# PHÂN TÍCH NGUY CƠ BẢO MẬT TRONG HẠ TẦNG MẠNG ẢO HÓA (NFV) VÀ 5G  
*Experimental Analysis of Security Threats in NFV Infrastructure: Supply Chain Attack Simulation and Defensive Mechanisms*

**Tác giả:** Trần Minh Phúc  

---

## Tóm tắt (Abstract)

Sự chuyển dịch từ hạ tầng mạng phần cứng chuyên dụng sang hạ tầng ảo hóa dựa trên NFV (Network Functions Virtualization) đang trở thành xu hướng chủ đạo trong triển khai 5G Core. Tuy nhiên, kiến trúc nhiều lớp (NFVI, VNF/CNF, MANO, SDN) cùng với sự phụ thuộc vào phần mềm bên thứ ba và chuỗi cung ứng phức tạp đã làm gia tăng đáng kể bề mặt tấn công của hệ thống. Bài báo này tập trung phân tích có hệ thống các lỗ hổng bảo mật theo ba lớp chính của NFV: NFVI, MANO và VNF, với trọng tâm là các kịch bản tấn công chuỗi cung ứng và đường leo thang đặc quyền.

Về mặt thực nghiệm, một môi trường NFV quy mô nhỏ dựa trên Kubernetes (MicroK8s) được xây dựng để mô phỏng chuỗi tấn công nhiều bước. Chuỗi này bao gồm: (i) khai thác rò rỉ tệp cấu hình `kubeconfig` để chiếm quyền điều khiển Control Plane (MANO), (ii) triển khai VNF độc hại và lợi dụng lỗ hổng kernel (CVE-2022-0492) để thực hiện Container Escape và leo thang lên quyền root trên NFVI Host, và (iii) sử dụng quyền root để vô hiệu hóa toàn bộ Orchestrator, gây ra tấn công Từ chối Dịch vụ (DoS) trên cụm dịch vụ NFV đang chạy.

Kết quả thực nghiệm trên testbed cho thấy rằng, chỉ từ một lỗi cấu hình đơn lẻ ở lớp quản trị (rò rỉ `kubeconfig`), kẻ tấn công có thể kết hợp với một lỗ hổng kernel đã biết để vượt qua các lớp cô lập logic, chiếm quyền điều khiển NFVI Host và làm gián đoạn nghiêm trọng tính sẵn sàng của dịch vụ. Dựa trên phân tích này, bài báo đề xuất một bộ cơ chế phòng thủ đa lớp, bao gồm hardening NFVI, bảo vệ MANO theo mô hình Zero Trust và đảm bảo tính toàn vẹn chuỗi cung ứng VNF thông qua ký số và giám sát runtime. Những kết quả này góp phần làm rõ mức độ rủi ro của hạ tầng NFV trong bối cảnh triển khai 5G thực tế.

**Keywords:** NFV, 5G Security, Supply Chain Attack, VNF Integrity, ETSI Architecture

---

## 1. Giới thiệu (Introduction)

Trong mạng viễn thông truyền thống, các chức năng như **Firewall, Router, Load Balancer** được triển khai dưới dạng thiết bị phần cứng chuyên dụng. Mô hình này có độ ổn định cao nhưng thiếu linh hoạt, chi phí đầu tư (CAPEX) và vận hành (OPEX) lớn, quá trình triển khai/nâng cấp kéo dài và khó tự động hóa ở quy mô lớn.

**Network Functions Virtualization (NFV)** được đề xuất nhằm giải quyết những hạn chế nêu trên, bằng cách **phần mềm hóa hạ tầng mạng**. Thay vì sử dụng thiết bị vật lý độc quyền, nhà mạng triển khai các chức năng mạng dưới dạng **Virtual Network Functions (VNF/CNF)** chạy trên hạ tầng phần cứng dùng chung (COTS servers) với lớp ảo hóa. NFV đóng vai trò quan trọng trong kiến trúc 5G Core (5GC), cho phép triển khai nhanh, mở rộng linh hoạt, hỗ trợ network slicing và tích hợp chặt chẽ với SDN.

Tuy nhiên, việc chuyển từ kiến trúc “hardware-centric” sang “software-centric” cũng kéo theo việc mở rộng bề mặt tấn công: từ firmware/hệ điều hành chuyên dụng sang API, hypervisor, container runtime, VNF/CNF, MANO và chuỗi cung ứng phần mềm. Các nghiên cứu gần đây đã chỉ ra rằng các lỗ hổng tại NFVI, lỗi cấu hình MANO, cũng như rủi ro từ VNF image bên thứ ba có thể tạo thành đường tấn công phức tạp, khó phát hiện và có tác động diện rộng.

### 1.1. Đóng góp chính (Contributions)

Bài báo này có các đóng góp chính sau:

- **Mô hình hóa rủi ro bảo mật NFV theo ba lớp NFVI–MANO–VNF**, làm rõ bề mặt tấn công mở rộng của hạ tầng ảo hóa so với hạ tầng vật lý truyền thống và liên hệ với bối cảnh 5G.
- **Xây dựng một môi trường mô phỏng NFV dựa trên Kubernetes (MicroK8s)**, đủ để tái hiện một chuỗi tấn công kết hợp giữa lỗi cấu hình vận hành và lỗ hổng kernel đã biết.
- **Thực nghiệm một chuỗi tấn công nhiều bước**: từ rò rỉ `kubeconfig` (Broken Access Control ở MANO), triển khai VNF độc hại, khai thác **CVE-2022-0492** để Container Escape và leo thang lên root NFVI Host, cho đến việc vô hiệu hóa Orchestrator gây gián đoạn toàn bộ dịch vụ NFV.
- **Đề xuất một bộ cơ chế phòng thủ đa lớp**, gắn trực tiếp với từng bước trong chuỗi tấn công, nhấn mạnh vai trò của hardening NFVI, bảo vệ MANO theo mô hình Zero Trust và đảm bảo tính toàn vẹn VNF trong chuỗi cung ứng.

---

## 2. Kiến thức nền và công trình liên quan (Background and Related Work)

### 2.1. Khái niệm NFV

**Network Functions Virtualization (NFV)** là công nghệ cho phép chuyển đổi các chức năng mạng truyền thống (Firewall, Router, IPS…) từ thiết bị phần cứng chuyên dụng sang **phần mềm chạy trên hạ tầng ảo hóa dùng chung**.

- **Ý tưởng cốt lõi:** Tách phần mềm VNF khỏi phần cứng vật lý.
- **Lợi ích chính:**
  - Giảm chi phí CAPEX/OPEX.
  - Tăng khả năng mở rộng và triển khai nhanh.
  - Dễ dàng nâng cấp, thay thế hoặc mở rộng tính năng.
  - Hỗ trợ tự động hóa nhờ khối quản lý và điều phối (MANO).

### 2.2. NFV và SDN

NFV thường được triển khai kết hợp với **Software-Defined Networking (SDN)**:

| Công nghệ | Mục tiêu | Vai trò |
|----------|---------|--------|
| **NFV**  | Ảo hóa chức năng mạng | Xử lý dịch vụ ở tầng VNF |
| **SDN**  | Tách Control-plane và Data-plane | Điều khiển đường đi của luồng gói tin |

SDN điều khiển luồng, NFV xử lý luồng, qua đó tạo thành một hạ tầng 5G linh hoạt và dễ tự động hóa.

### 2.3. Kiến trúc ETSI NFV

Theo ETSI, NFV bao gồm ba khối chính:

1. **NFVI (NFV Infrastructure)**  
   - Phần cứng vật lý: server, storage, switch.  
   - Lớp ảo hóa: hypervisor (KVM, VMware ESXi), container (Docker, Kubernetes).  
   - Cung cấp tài nguyên CPU/RAM/Network cho VNF.

2. **VNF (Virtual Network Functions)**  
   - Các chức năng mạng dưới dạng phần mềm: vRouter, vFirewall, vIDS, UPF…  
   - Hỗ trợ triển khai, mở rộng, cập nhật, chấm dứt vòng đời.

3. **MANO (Management and Orchestration)**  
   - **NFVO:** Điều phối dịch vụ mạng (Network Service).  
   - **VNFM:** Quản lý vòng đời VNF.  
   - **VIM:** Quản lý tài nguyên NFVI (OpenStack, VMware, Kubernetes…).

### 2.4. Vòng đời của VNF

Các giai đoạn chính của vòng đời VNF gồm:

1. Initialization  
2. Instantiation  
3. Configuration  
4. Scaling (In/Out/Up/Down)  
5. Updating / Patching  
6. Termination  

Malware trong tấn công **Supply Chain** thường được cấy tại các giai đoạn khởi tạo, mở rộng hoặc cập nhật, khi image VNF được build, kéo về hoặc triển khai.

### 2.5. Hạ tầng vật lý và NFV: bề mặt tấn công

#### 2.5.1. Mô hình hạ tầng

| Mô hình | Đặc điểm | Bề mặt tấn công tập trung |
|--------|---------|---------------------------|
| **Máy chủ vật lý riêng (Bare Metal Server)** | Không có lớp ảo hóa giữa phần cứng và hệ điều hành (OS). | Firmware/BIOS, OS, cổng quản trị phần cứng. |
| **NFV (Network Function Virtualization)** | Kiến trúc ảo hóa, tách chức năng mạng (VNF/CNF) khỏi phần cứng độc quyền. | API, hypervisor/container runtime, MANO, chuỗi cung ứng phần mềm. |

#### 2.5.2. Lợi ích cốt lõi của NFV đối với 5G

NFV gần như là điều kiện cần để 5G Core vận hành hiệu quả, nhờ:

- **Tính linh hoạt & khả năng mở rộng:** Triển khai và mở rộng dịch vụ nhanh dựa trên nhu cầu.
- **Network Slicing:** Tạo các mạng logic độc lập (slice) trên cùng hạ tầng vật lý.
- **MEC (Multi-access Edge Computing):** Đặt chức năng như UPF gần người dùng để giảm độ trễ.

#### 2.5.3. So sánh bề mặt tấn công

| Đặc điểm | Mạng vật lý truyền thống | Mạng ảo hóa NFV/Cloud |
|---------|--------------------------|------------------------|
| **Bản chất** | Gần như hardware-centric. | Chủ yếu software-centric. |
| **Lớp tấn công** | Có thể xem tập trung vào ~3 lớp (Mạng, Thiết bị, Ứng dụng). | Trải rộng ra ít nhất 5 lớp (VNF, Container, NFVI, MANO, SDN). |
| **Điểm tập trung** | Từng thiết bị mạng riêng lẻ. | Điểm tập trung tại MANO/Kubernetes API. |
| **Phạm vi rủi ro** | Thường cục bộ, bị giới hạn bởi ranh giới vật lý. | Có thể lan rộng, phụ thuộc vào mức độ cô lập logic. |
| **Kỹ thuật tấn công** | DoS truyền thống, chiếm quyền qua cổng quản trị vật lý. | Tấn công chuỗi cung ứng, khai thác token API, Container Escape, tấn công Orchestrator. |

### 2.6. NFV trong triển khai 5G

- 5G Core thường được xây dựng trên nền NFV/Cloud-native.  
- Thành phần chính của 5G Core đều là VNF/CNF:  
  - **AMF:** Access Management Function.  
  - **SMF:** Session Management Function.  
  - **UPF:** User Plane Function.

**Control Plane vs Data Plane:**

- Control-plane: AMF, SMF → điều khiển phiên kết nối.  
- Data-plane: UPF → xử lý lưu lượng người dùng.

Tấn công vào UPF có thể dẫn đến nghe lén, thay đổi hoặc chuyển hướng traffic.

### 2.7. Service Function Chaining (SFC)

- Được điều khiển bởi:
  - SDN Controller (OpenDaylight, ONOS…).  
  - NSH (Network Service Header) để phân biệt luồng.

Ví dụ chuỗi dịch vụ:

> User → vFirewall → vIDS → vRouter → Internet

Nếu kẻ tấn công chiếm một VNF trong chuỗi và có khả năng thao túng cấu hình SFC, họ có thể thay đổi thứ tự hop hoặc chèn thêm VNF độc hại.

### 2.8. Công trình liên quan (Related Work)

Nhiều nghiên cứu đã phân tích bảo mật NFV và 5G, tập trung vào: (i) lỗ hổng trong hạ tầng ảo hóa và MANO, (ii) rủi ro container/Kubernetes trong 5G core cloud-native, và (iii) tấn công chuỗi cung ứng phần mềm. Một số survey đề cập chi tiết đến vấn đề đặt và xâu chuỗi VNF, cũng như các thách thức bảo mật khi triển khai trên môi trường multi-tenant.

Bài báo này kế thừa các khung phân tích đó, đồng thời tập trung cụ thể vào một chuỗi tấn công thực nghiệm điển hình kết hợp lỗi cấu hình (rò rỉ `kubeconfig`) và lỗ hổng kernel (CVE-2022-0492) trong bối cảnh NFV/5G.

---

## 3. Phân tích lỗ hổng bảo mật (Security Analysis)

### 3.1. Lỗ hổng tại NFVI

NFVI bao gồm phần cứng và lớp ảo hóa (Hypervisor/Container Runtime). Lỗ hổng tại đây có thể dẫn đến chiếm quyền toàn bộ hạ tầng:

- **Tấn công lớp ảo hóa (VM Escape):** Khai thác lỗi trong hypervisor (KVM, VMware ESXi) để thoát khỏi VM và chiếm quyền host.
- **Container Escape:** Khai thác lỗ hổng trong container runtime hoặc Linux kernel (ví dụ: **CVE-2022-0492**) để leo thang đặc quyền từ container lên host.
- **Lỗ hổng driver/firmware:** NIC, BIOS, hoặc module kernel I/O có thể có lỗ hổng chưa vá, cho phép thực thi mã độc ở mức thấp.

### 3.2. Lỗ hổng tại MANO

MANO (NFVO, VNFM, VIM) là Control Plane của NFV:

- **Lỗ hổng API:** Injection, lỗi xác thực/ủy quyền trong NFVO/VNFM/VIM APIs.
- **Broken Access Control:** Sai cấu hình RBAC, rò rỉ `kubeconfig`, service account token.
- **Thao túng Orchestration:** Can thiệp vào quyết định Instantiation, Scaling, Termination của VNF.

### 3.3. Lỗ hổng tại VNF

VNF là mục tiêu dễ bị tấn công:

- **Lỗ hổng ứng dụng:** Buffer overflow, logic bug, lỗi cấu hình dịch vụ.
- **Rủi ro tính toàn vẹn:** VNF image có thể bị chỉnh sửa, cấy backdoor trong pipeline build hoặc tại registry.
- **Hậu quả:** VNF bị chiếm có thể trở thành man-in-the-middle nội bộ, can thiệp traffic, hoặc làm bàn đạp tấn công ngược lên MANO và NFVI.

---

## 4. Phân tích rủi ro nâng cao (Advanced Threat Analysis)

### 4.1. Tấn công chuỗi cung ứng với VNF (VNF Integrity Attack)

Đặc trưng bởi:

- Nhắm vào khâu phát triển, lưu trữ, phân phối VNF image.
- Cấy mã độc (Trojan/Backdoor) vào image (`.qcow2`, container image) tại supply chain stage.
- VNF độc hại được triển khai qua MANO, đi vào trung tâm mạng mà không bị các lớp bảo vệ ngoại vi chặn.

### 4.2. Đường leo thang đặc quyền (Escalation Path)

Sự kết hợp lỗi cấu hình và lỗ hổng hệ thống tạo nên đường tấn công:

#### 4.2.1. Phá vỡ ranh giới cô lập

- Kiến trúc 5G SBA giả định các network functions nội bộ là tương đối tin cậy.
- Container Escape (như CVE-2022-0492) cho phép thoát ra khỏi ranh giới container khi hội đủ điều kiện, dẫn tới chiếm quyền host.

#### 4.2.2. Tác động khi chiếm quyền Control Plane

- Chiếm MANO (NFVO/VNFM/VIM) cho phép:
  - Terminate VNF hợp lệ → DoS.  
  - Deploy VNF độc hại hàng loạt → chiếm tài nguyên, tạo backdoor.  
  - Thao túng network slicing và SFC.

---

## 5. Thực nghiệm (Experimental Setup and Attack Simulation)

### 5.1. Thiết lập môi trường mô phỏng

Môi trường mô phỏng được triển khai như sau:

| Thành phần | Vai trò | Lỗ hổng cài đặt sẵn (giả lập) |
|-----------|---------|--------------------------------|
| **NFVI Host** (`192.168.1.10`) | Host OS, kernel, MicroK8s. | **Lỗi 1 (vận hành):** `kubeconfig` bị lộ trên web server `8080`. |
| **Attacker Host** (`192.168.1.21`) | Kali Linux (`kubectl`, `gobuster`, `netcat`, exploit PoC). | **Lỗi 2 (hệ thống):** kernel NFVI Host dễ bị khai thác **CVE-2022-0492**. |

### 5.2. Chuỗi tấn công nhiều bước

#### 5.2.1. Pha I: Xâm nhập ban đầu và chiếm Control Plane

- Quét web server và tải `kubeconfig`.
- Thiết lập biến môi trường `KUBECONFIG` để sử dụng Kubernetes API với quyền cluster-admin (theo giả lập).

```bash
curl http://192.168.1.10:8080/admin_kubeconfig_2025.txt -o ./hacked_config
export KUBECONFIG=./hacked_config
kubectl get nodes
```

#### 5.2.2. Pha II: Chiếm Data Plane và Container Escape

- Triển khai pod VNF độc hại với `privileged: true`.
- Chạy exploit PoC cho CVE-2022-0492 từ trong container, gửi reverse shell về Attacker Host.

| Thao tác | Lệnh (minh họa) | Kết quả |
|---------|------------------|---------|
| Deploy pod độc hại | `kubectl apply -f malicious_pod.yaml` | Pod được khởi tạo. |
| Chạy exploit CVE | `/tmp/exploit_2022.bin ... /dev/tcp/192.168.1.21/5555 ...` | Nhận reverse shell root từ NFVI Host. |
| Kiểm tra quyền | `whoami; cat /etc/os-release` | Output: `root` trên NFVI Host. |

#### 5.2.3. Pha III: Gây gián đoạn dịch vụ (DoS)

- Từ shell root trên NFVI Host, thực thi lệnh reset MicroK8s:

```bash
microk8s reset -y
```

- Kết quả: toàn bộ cluster Kubernetes và các VNF/CNF đang chạy bị dừng hoạt động, gây mất dịch vụ trên testbed.

### 5.3. Phân tích kết quả thực nghiệm

- Lỗ hổng cấu hình (`kubeconfig` lộ) cho phép vượt qua lớp bảo vệ API và chiếm quyền Control Plane.
- Lỗ hổng kernel (CVE-2022-0492) trong bối cảnh container privileged cho phép container escape lên NFVI Host.
- Khả năng thực thi lệnh ở mức root trên NFVI Host dẫn tới gián đoạn hoàn toàn dịch vụ NFV trong môi trường mô phỏng.

Kết quả này minh họa rằng, trong một cấu hình NFV dựa trên Kubernetes, sự kết hợp giữa lỗi vận hành và lỗ hổng hệ thống có thể tạo thành một chuỗi tấn công liên tầng với tác động đáng kể.

---

## 6. Cơ chế phòng thủ (Defense Mechanisms)

### 6.1. Bảo vệ NFVI

- Hardening kernel, vá lỗ hổng thường xuyên.
- Áp dụng SELinux/AppArmor.
- Sử dụng sandbox như gVisor, Kata Containers cho workload nhạy cảm.
- Hạn chế tối đa privileged containers, dùng PodSecurity Standards/Policies.

### 6.2. Bảo vệ MANO

- Áp dụng Zero Trust cho API nội bộ.
- Bảo vệ bí mật cấu hình (`kubeconfig`, tokens) bằng secrets management.
- Thiết kế RBAC chặt chẽ, nguyên tắc least privilege, tránh dùng cluster-admin cho tác vụ thông thường.

### 6.3. Bảo vệ chuỗi cung ứng VNF

- Ký số và xác thực VNF image (Notary, Sigstore/Cosign).
- Giám sát runtime hành vi VNF, phát hiện bất thường.
- Áp dụng quy trình CI/CD an toàn, kiểm soát mã nguồn và dependencies.

---

## 7. Kết luận (Conclusion)

Bài báo đã phân tích bề mặt tấn công của NFV theo ba lớp NFVI–MANO–VNF trong bối cảnh 5G, đồng thời xây dựng một testbed NFV dựa trên Kubernetes để mô phỏng chuỗi tấn công nhiều bước kết hợp lỗi cấu hình và lỗ hổng kernel. Thực nghiệm cho thấy, trong điều kiện cấu hình dễ bị tổn thương, kẻ tấn công có thể leo thang đặc quyền từ mức truy cập bên ngoài lên tới root trên NFVI Host và gây gián đoạn dịch vụ ở mức nghiêm trọng.

Dựa trên kết quả thu được, bài báo đề xuất một bộ cơ chế phòng thủ đa lớp nhằm giảm thiểu rủi ro, bao gồm hardening NFVI, bảo vệ MANO theo mô hình Zero Trust và tăng cường đảm bảo tính toàn vẹn cho chuỗi cung ứng VNF. Trong tương lai, hướng nghiên cứu có thể mở rộng sang môi trường multi-tenant, multi-slice quy mô lớn hơn, cũng như đánh giá định lượng tác động hiệu năng của các biện pháp phòng thủ được đề xuất.

---

## Tài liệu tham khảo (References)

1. ETSI NFV Architecture Overview.  
2. Schardong et al., “Survey on VNF Placement and Chaining,” 2021.  
3. Nguyễn Văn Quân, “Khóa luận tốt nghiệp về NFV Optimization,” 2023.  
4. 3GPP 5G Security Standards.  
5. (Có thể bổ sung thêm các bài báo NFV/5G security, container/Kubernetes security tùy yêu cầu cuộc thi/hội nghị.)  
