### **1. 가상환경(venv) 생성 및 활성화**

1. **원하는 폴더로 이동**
   ```bash
   cd /path/to/your/project
   ```

2. **가상환경 생성**
   ```bash
   python -m venv venv
   ```

3. **가상환경 활성화**
   - **Linux/Mac**:
     ```bash
     source venv/bin/activate
     ```
   - **Windows**:
     ```bash
     venv\Scripts\activate
     ```

4. **활성화 확인**  
   `(venv)` 표시가 터미널에 나타나는지 확인.

---

### **2. 라이브러리 설치 (nexus 셋업되어 있을 경우): 지금 환경에선 이건 무시하시면 될 것 같아요**

가상환경 활성화 상태에서 아래 명령어를 실행합니다:

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip install transformers
pip install accelerate
```

---

### **2. 라이브러리 설치 (nexus 셋업 안되어 있을 경우 - 폐쇄망)**

1. **압축 파일 복사**
   - `pytorch_packages.tar.gz` 파일을 현재 작업 디렉터리로 복사합니다.

2. **압축 해제**
   ```bash
   tar -xzvf pytorch_packages.tar.gz -C ./pytorch_packages
   ```

   - 압축 해제 후, `pytorch_packages` 디렉터리에 모든 `.whl` 파일이 저장됩니다.

3. **로컬 `.whl` 파일로 설치**
   - 아래 명령어를 통해 로컬 디렉터리에서 패키지를 설치합니다:
     ```bash
     pip install --no-index --find-links=./pytorch_packages torch torchvision torchaudio
     pip install --no-index --find-links=./pytorch_packages transformers accelerate
     ```

---


### **3. GPU 활성화 확인**

아래 명령어를 실행하여 GPU가 정상적으로 사용 가능한지 확인합니다:

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

- **출력**: `True`가 나오면 성공.

---

### **4. 모델 테스트 코드 작성**

아래 코드를 `test.py` 파일로 저장합니다:

```python
import transformers
import torch

# 로컬에 저장된 모델 경로 지정
model_path = "/path/to/your/model"  # 모델 경로

# 파이프라인 생성
pipeline = transformers.pipeline(
    "text-generation",
    model=model_path,  # 모델 로컬 경로
    model_kwargs={"torch_dtype": torch.bfloat16},
    device_map="auto",  # GPU 활용
)

# 입력 메시지
messages = [
    {"role": "system", "content": "You are a pirate chatbot who always responds in pirate speak!"},
    {"role": "user", "content": "Who are you?"},
]

# 메시지 입력 및 출력
outputs = pipeline(
    messages,
    max_new_tokens=256,
)
print(outputs[0]["generated_text"])
```

---

### **5. 테스트 실행**

아래 명령어로 실행합니다:

```bash
python test.py
```

- **출력**: 모델이 입력 메시지에 대한 결과를 반환하면 성공입니다.

---

### **참고 사항**
- 모델 경로(`/path/to/your/model`)를 적절히 설정해야 합니다.
- GPU가 정상적으로 활성화되지 않으면 `device_map="auto"` 대신 `device="cpu"`를 사용하여 CPU로 테스트 가능합니다.