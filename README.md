# VPE-Ko

**한국어 멀티턴 대화 기반 Text-to-Video 프롬프트 생성 챗봇 시스템**

[![Paper](https://img.shields.io/badge/Paper-KoreaAI%202025-blue)](https://github.com/wndaasa/VPE-Ko/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📖 개요

VPE-Ko(Video Prompt Engineering based on Korean)는 한국어 대화를 통해 Text-to-Video 모델에 최적화된 영어 프롬프트를 생성하는 시스템입니다.

대부분의 T2V 모델은 영어 프롬프트를 전제로 설계되어 있어, 한국어 사용자들이 고품질 프롬프트를 작성하는 데 어려움을 겪습니다. VPE-Ko는 이러한 문제를 해결하기 위해 개발되었습니다.

### 주요 기능

- 🇰🇷 **한국어 대화 인터페이스**: 자연스러운 한국어 대화로 프롬프트 생성
- 🎬 **CogVideoX 최적화**: CogVideoX 스타일에 맞는 프롬프트 생성
- 💬 **멀티턴 대화**: 단계별 질문을 통해 상세한 정보 수집
- ⚖️ **가중치 조절**: 중요 요소에 자동 가중치 부여
- 🎥 **비디오 생성 연동**: 생성된 프롬프트로 바로 비디오 생성 가능

## 📁 프로젝트 구조

```
VPE-Ko/
├── checkpoints/              # 학습된 LoRA 어댑터
│   ├── adapter_config.json
│   ├── adapter_model.safetensors
│   └── README.md
├── dataset/                  # 데이터셋
│   └── qa_dataset.json       # 학습용 대화쌍 데이터
├── experience/               # 실험 결과
├── src/                      # 소스 코드
│   ├── create_dataset.py     # 데이터셋 생성
│   ├── fine_tune_qwen.py     # 모델 학습
│   └── chat_to_cogvideox.py  # 대화 및 비디오 생성
├── prompts/                  # 시스템 프롬프트 (선택)
├── output/                   # 생성된 비디오 및 프롬프트 저장
├── requirements.txt
└── README.md
```

## 🔧 설치

### 요구사항

- Python 3.10+
- CUDA 11.8+ (GPU 사용 시)
- 최소 24GB VRAM (14B 모델 기준)

### 설치 방법

```bash
# 1. 저장소 클론
git clone https://github.com/wndaasa/VPE-Ko.git
cd VPE-Ko

# 2. 가상환경 생성 (권장)
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows

# 3. 의존성 설치
pip install -r requirements.txt
```

### requirements.txt

```
torch>=2.0.0
transformers>=4.40.0
peft>=0.10.0
trl>=0.8.0
datasets>=2.0.0
openai>=1.0.0
diffusers>=0.27.0
accelerate>=0.20.0
bitsandbytes>=0.40.0
sentencepiece
```

> ⚠️ CogVideoX 비디오 생성 기능을 사용하려면 diffusers의 CogVideoX 지원 버전이 필요합니다.

## 🚀 사용법

### 1. 데이터셋 생성 (선택)

GPT API를 활용하여 학습용 대화 데이터셋을 생성합니다.

```bash
cd src
python create_dataset.py
```

> ⚠️ OpenAI API 키가 필요합니다. `create_dataset.py`에서 API 키를 설정하세요.

### 2. 모델 학습 (선택)

Qwen2 모델을 VPE-Ko 데이터셋으로 파인튜닝합니다.

```bash
python fine_tune_qwen.py
```

학습 설정:
- Base Model: `Qwen/Qwen2.5-14B-Instruct`
- LoRA rank: 16
- Learning rate: 1e-5
- Epochs: 1

### 3. 대화 및 비디오 생성

학습된 모델로 대화하며 프롬프트를 생성하고, 비디오를 생성합니다.

```bash
python chat_to_cogvideox.py
```

#### 대화 예시

```
비디오의 주제를 입력해 주세요: 뜨겁게 타오르는 화산

챗봇: 화산의 분화 장면을 어떤 시점에서 보여드릴까요? 예를 들어, 멀리서 전체 모습을 보여주거나, 가까이서 용암이 흐르는 모습을 보여줄 수 있어요.

사용자: 위에서 내려다보는 시점으로 해줘

챗봇: 카메라의 종류는 어떤 것을 원하시나요? 드론 촬영이나 헬리콥터에서 찍은 듯한 항공 촬영 등이 있을 수 있어요.

사용자: 드론으로 해줘

...

챗봇: 
### 최종 프롬프트:
"A scorching volcano erupts in a breathtaking display of fiery majesty, its (molten lava:2.0) flowing down the mountain's sides like a river of liquid fire. From above, the (drone camera:1.5) captures the scene in sweeping aerial shots..."

### 부정 프롬프트:
"low quality, blurry, static, cartoon style, 2D"
```

## 📊 실험 결과

VPE-Ko의 유효성을 검증하기 위해 A-B 테스트를 진행했습니다.

| 평가 항목 | 사용자 직접 작성 (A) | VPE-Ko 기반 (B) |
|----------|---------------------|-----------------|
| G-Eval | 3.6 | **4.9** |
| 프롬프트 만족도 | 3.6 | **4.5** |
| 영상 만족도 | 2.1 | **4.6** |

## 🎯 VPE-Ko 핵심 요소

프롬프트 생성 시 다음 5가지 핵심 요소를 수집합니다:

| 핵심 요소 | 예시 |
|----------|------|
| 주요 객체 | 건물, 사람, 동물 등 |
| 카메라 구도 | 전경, 클로즈업 등 |
| 카메라 종류 | 드론, 1인칭 카메라 등 |
| 영상의 속도 | 느린 속도, 빠른 속도 등 |
| 조명 | 자연광, 네온 조명 등 |

## 📝 Citation

본 연구가 도움이 되셨다면 인용 부탁드립니다:

```bibtex
@inproceedings{lim2025vpeko,
  title={한국어 기반 Text-to-Video 프롬프트 엔지니어링 시스템 연구: CogVideoX를 중점으로},
  author={임영기 and 최지희 and 엄태선 and 이우성 and 구영현},
  booktitle={한국인공지능학회},
  year={2025}
}
```

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- 본 연구는 2025년도 정부(과학기술정보통신부)의 재원으로 정보통신산업진흥원의 지원을 받아 수행되었습니다.
- Base Model: [Qwen2.5-14B-Instruct](https://huggingface.co/Qwen/Qwen2.5-14B-Instruct)
- Video Generation: [CogVideoX](https://github.com/THUDM/CogVideoX)
