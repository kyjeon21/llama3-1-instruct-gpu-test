`nvidia-smi`가 작동하지 않는 문제는 주로 NVIDIA 드라이버 문제에서 기인하며, 드라이버와 커널 또는 GPU와 드라이버 간의 호환성 문제일 가능성이 높습니다. H100과 같은 최신 GPU는 특정 드라이버와 CUDA 버전이 요구되므로, 환경에 맞는 설정이 필요합니다. 아래에서 문제 해결 과정을 안내합니다.

---

## **1. 환경 분석**

1. **GPU: NVIDIA H100 SXM5 80GB**
   - 최신 GPU 모델이므로 드라이버와 CUDA Toolkit의 최신 버전이 필요.
   - H100은 NVIDIA CUDA 12.0 이상을 지원합니다.

2. **OS: Linux 4.18.0-477.10.1.el8_8.x86_64 (RHEL 8 기반)**
   - CentOS, RHEL, AlmaLinux 등의 RHEL 계열 OS에서는 특정 드라이버 버전이 필요합니다.
   - RHEL 계열은 드라이버 설치 시 추가적인 설정이 필요할 수 있습니다.

---

## **2. 문제 해결 절차**

### **2.1. 드라이버 상태 확인**

1. **현재 설치된 드라이버 확인**
   ```bash
   rpm -qa | grep -i nvidia
   ```
   - 설치된 NVIDIA 드라이버가 표시되지 않으면 드라이버가 설치되지 않았거나 문제가 있을 가능성이 높습니다.

2. **GPU가 시스템에 인식되는지 확인**
   ```bash
   lspci | grep -i nvidia
   ```
   - 출력 예시:
     ```
     01:00.0 VGA compatible controller: NVIDIA Corporation H100 SXM5 (rev a1)
     ```

3. **커널 모듈 로드 상태 확인**
   ```bash
   lsmod | grep nvidia
   ```
   - 출력이 없다면 NVIDIA 드라이버가 커널 모듈로 로드되지 않은 상태입니다.

4. **드라이버 로그 확인**
   ```bash
   dmesg | grep -i nvidia
   ```
   - 드라이버 로드 실패 시 관련 로그가 표시됩니다.

---

### **2.2. NVIDIA 드라이버 재설치**

1. **NVIDIA 드라이버 다운로드**
   - NVIDIA 드라이버 다운로드 페이지에서 H100과 RHEL 8에 맞는 최신 드라이버를 다운로드:  
     [NVIDIA 드라이버 다운로드](https://www.nvidia.com/Download/index.aspx)

   - 드라이버 버전 예시: **535.104.05** 또는 최신 버전.

2. **기존 드라이버 제거**
   ```bash
   sudo yum remove -y '*nvidia*'
   sudo yum autoremove
   ```

3. **필요 패키지 설치**
   ```bash
   sudo yum install -y epel-release
   sudo yum install -y dkms gcc kernel-devel kernel-headers
   ```

4. **드라이버 설치**
   - 다운로드한 `.run` 파일 실행:
     ```bash
     sudo bash NVIDIA-Linux-x86_64-535.104.05.run
     ```
   - 설치 옵션:
     - DKMS 모듈 설치 여부를 확인하여 드라이버가 커널 업데이트 후에도 유지되도록 설정.

5. **NVIDIA 모듈 강제 로드**
   ```bash
   sudo modprobe nvidia
   ```

6. **시스템 재부팅**
   ```bash
   sudo reboot
   ```

---

### **2.3. NVIDIA Docker 사용 시 설정**
H100과 같은 GPU를 사용하는 경우, Docker 환경에서도 드라이버가 필요할 수 있습니다.

1. **NVIDIA Container Toolkit 설치**
   ```bash
   curl -s -L https://nvidia.github.io/nvidia-docker/rhel8.8/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo
   sudo yum install -y nvidia-container-toolkit
   sudo systemctl restart docker
   ```

2. **Docker에서 GPU 테스트**
   ```bash
   docker run --rm --gpus all nvidia/cuda:12.2.0-base nvidia-smi
   ```

---

### **2.4. 커널 업데이트 및 드라이버 호환성 확인**

1. **커널 버전 확인**
   ```bash
   uname -r
   ```

2. **RHEL 8 최신 커널 설치**
   - 최신 커널 업데이트:
     ```bash
     sudo yum update kernel kernel-headers kernel-devel
     sudo reboot
     ```

3. **NVIDIA 드라이버와 커널 모듈 재설치**
   - 커널 업데이트 후 드라이버를 재설치하여 호환성을 확인합니다.

---

### **2.5. CUDA 설치**
H100은 CUDA 12.0 이상이 필요합니다.

1. **CUDA Toolkit 다운로드**
   - [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)에서 RHEL 8 및 H100 지원 버전 선택.

2. **CUDA 설치**
   ```bash
   sudo rpm -i cuda-repo-rhel8-12-2-local-12.2.0-1.x86_64.rpm
   sudo yum clean all
   sudo yum install -y cuda
   ```

3. **환경 변수 설정**
   ```bash
   echo 'export PATH=/usr/local/cuda-12.2/bin:$PATH' >> ~/.bashrc
   echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.2/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
   source ~/.bashrc
   ```

---

### **2.6. 확인 작업**

1. **nvidia-smi 테스트**
   ```bash
   nvidia-smi
   ```
   - 정상적으로 GPU 정보가 출력되면 성공.

2. **CUDA 확인**
   ```bash
   nvcc --version
   ```
   - CUDA 버전 정보가 출력되면 정상 설치.

3. **샘플 테스트**
   ```bash
   /usr/local/cuda-12.2/samples/1_Utilities/deviceQuery/deviceQuery
   ```

---

### **3. 예상 문제 및 해결**

1. **`nvidia-smi` 여전히 작동하지 않음**
   - NVIDIA 커널 모듈이 충돌 중일 수 있습니다:
     ```bash
     sudo rmmod nvidia
     sudo modprobe nvidia
     ```

2. **드라이버와 CUDA 버전 불일치**
   - H100은 CUDA 12.0 이상 및 최신 드라이버가 필요합니다.
   - [NVIDIA Compatibility Matrix](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)에서 호환성을 다시 확인하세요.

3. **커널 업데이트 후 드라이버 작동 실패**
   - `dkms`를 이용해 드라이버를 다시 빌드:
     ```bash
     sudo dkms build -m nvidia -v 535.104.05
     sudo dkms install -m nvidia -v 535.104.05
     ```

---

이 과정을 통해 NVIDIA 드라이버와 CUDA를 설치하고 H100 GPU를 정상적으로 사용할 수 있을 것입니다. 추가 문제가 발생하면 에러 로그를 공유해주세요!