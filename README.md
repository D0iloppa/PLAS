# PLAS (Personal Language Acquisition System)

> 언어 학습이 아닌, 언어 습득을 위한 개인 프레임워크

## 🎯 Why This Project?

나는 엔지니어로서 4년 넘게 일했지만, 영어는 여전히 어렵다.
전산 용어가 대부분 영어로 이루어졌음에도 말이다.

**문제의 본질**:
- 기존 앱들은 "언어 학습(Learning)"에 집중
- 하지만 언어는 "습득(Acquisition)"되어야 한다

**Krashen의 Input Hypothesis**:
1. Comprehensible Input (이해 가능한 입력)
2. Low Affective Filter (심리적 장벽 최소화)

이 이론을 소프트웨어로 구현하면?

→ **이 프로젝트는 그 가설에 대한 실험이며, 내가 알파테스터이자 PoC다.**

---

## 🏗️ Architecture
```
┌─────────────────────────────────────┐
│         User Interface               │
│   (Content Player + Chat)           │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│       Input Engine                   │
│  Content → Comprehensible Input     │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│       Shared System                  │
│  User Vocab + Acquisition Metrics   │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│       Mirror Engine                  │
│  Linguistic Parent AI                │
└─────────────────────────────────────┘
```

**Core Concepts**:
- **Input Engine**: 내가 좋아하는 콘텐츠를 학습 자료로 변환
- **Mirror Engine**: 문법 교정 없이, 자연스럽게 재표현하는 AI
- **Shared System**: 나의 어휘/습득 수준을 추적

상세 아키텍처: [docs/architecture.md](docs/architecture.md)

---

## 🚀 Current Status

**Phase**: Prototype (MVP 구축 중)

- [x] 아키텍처 설계
- [ ] Input Engine (YouTube subtitle extraction)
- [ ] Mirror Engine (GPT-4 integration)
- [ ] Basic Web UI
- [ ] Personal usage starts (Target: 2024-12-01)

---

## 🛠️ Tech Stack

**Backend**:
- Java Spring Boot (Orchestration)
- Python (STT/NLP utilities)

**AI/ML**:
- OpenAI GPT-4 API (Mirror Engine)
- Whisper API (STT)

**Storage**:
- PostgreSQL (User data)
- Redis (Cache)

**Frontend**:
- React
- shadcn/ui

---

## 📖 Documentation

- [Architecture Blueprint](docs/architecture.md) - 시스템 설계 상세
- [Technical Whitepaper](docs/whitepaper.md) - 이론적 배경
- [Development Log](docs/devlog.md) - 개발 일지

---

## 🎓 Research Background

이 프로젝트는 다음 연구/이론에 기반합니다:

- **Stephen Krashen** - Input Hypothesis
- **Chris Lonsdale** - "How to learn any language in 6 months" (TEDx)
- **Paul Nation** - Vocabulary acquisition research

---

## 🧪 The Experiment

**가설**: 
이 프레임워크를 6개월 사용하면, 
나는 영어로 자연스럽게 구사할 수 있다.

**측정 지표**:
- 발화 복잡도 (words per sentence)
- 응답 지연 시간 (reaction time)
- 자발적 표현 비율 (spontaneous vs memorized)

**검증 방법**:
- Before/After 비디오 비교
- 매주 대화 로그 분석
- 6개월 후 실제 외국인과 대화 테스트

---

## 🤝 Contributing

현재는 개인 실험 프로젝트이지만,
관심 있으신 분들은 Issue/Discussion 환영합니다.

---

## 📝 License

MIT License - 자유롭게 사용하세요

---

## 👤 Author

**권도일**
- Blog: [https://blog.naver.com/kdi3939](https://blog.naver.com/kdi3939)
- Email: kdi3939@gmail.com

---

## 📚 Blog Series

개발 과정을 블로그에 연재 중:
1. [언어 습득 vs 언어 학습: Krashen 이론 정리](https://blog.naver.com/kdi3939/224080263020)

---

_Last updated: 2025-11-18
