---
layout: post
title: WDSS (Web Show Drone Simulator)
---

기획부터 검증까지 한 화면에서 끝내는 드론 쇼 웹 제작 서비스

- **깃허브** : [Github](https://github.com/MW-1st) / **노션** : [Notion](https://www.notion.so/WDSS-Project-DroneFactory-2519421e3b718077a932da9270ab6944?source=copy_link)
- **개발 인원** : 6명
- **프로젝트 진행 일자** : 2025. 08. 21. ~ 2025. 09. 27.
- **맡은 역할** : C#을 통해, Web과 Unity, 실제 드론(ESP32 Drone)과 통신하는 **중앙 제어 서버(GCS) 구축 및 구현**

---

## 프로젝트 배경

- **기획** : 드론쇼 제작은 설계–변환–시뮬–수정 과정이 단절되어 준비 기간이 길고(통상 4주+), 전문 인력 의존과 인건비 부담이 큼. 비전공자가 접근하기 어려움.
- **기술** : 이미지 → 점군 변환과 실시간 미리보기를 단일 파이프라인에서 처리하고, 중앙 제어 로직을 서버로 분리(ASP.NET Core)해 Web·Unity·실드론(ESP32)까지 일관된 제어가 필요.
- **목표** : 웹 기반 에디터–GCS–뷰어(Unity WebGL)를 잇는 엔드투엔드 파이프라인으로 *빠른 제작·저비용·실기 확장*을 검증.

---

## 프로젝트 문제

- **모놀리식 구조**: Unity 내부 스크립트로 GCS 기능(상태/세션/통신/이벤트/시나리오)을 모두 구현 → 시뮬레이션 내에서만 동작, 실 드론 적용 불가.
- **성능 병목**: 렌더링과 제어 로직이 한 프로세스라 CPU 사용률 급증(평균 85% 이상), FPS 변동 심화.
- **운영 난이도**: 서버/클라이언트 강결합으로 CI/CD·테스트·배포 효율 저하.

---

## 해결과정

- **서버 분리**: 중앙 제어 로직을 Unity에서 완전히 분리 → **ASP.NET Core 기반 GCS 서버**를 AWS EC2에 독립 배포.
- **Unity 역할 단순화**: Unity는 **렌더링 전담(Viewer)** 으로 축소.

- **결과**:
  - Unity CPU 점유율 **85% → 25% 감소**
  - 서버 리소스 사용량 **70% 절감**
  - **실제 ESP32 드론 적용 가능성 확보**, 시뮬과 현실 간 일관된 아키텍처 달성

---

## 기술적 의사결정 과정

- **ASP.NET Core(C#)** : 기존 C# 코드 재사용 + 고성능 비동기 통신 지원 → 빠른 전환 및 안정성 확보.
- **FastAPI(OpenCV)** : 이미지 변환·전처리를 비동기로 처리, JSON 기반 결과물을 GCS 서버와 공유.
- **통신 방식** :
  - WebSocket → 시뮬레이터와 서버 간 양방향 상태 동기화
  - UDP/TCP → 드론 실기 제어(저지연, 경량화)
- **인프라** : AWS EC2로 서버 구성, DB는 PostgreSQL+Redis 혼합 운용(영속/세션 분리).

---

## 아키텍쳐 설명

**Frontend–Backend–DB**(FastAPI/OpenCV + PostgreSQL/Redis)와 **ASP.NET GCS 서버**를 AWS EC2에 독립 구성.

- 사용자는 **Web Editor(React·Vite·Fabric.js)** 로 장면을 설계 → **FastAPI(OpenCV)** 가 점군 변환/JSON 생성 → **PostgreSQL/Redis** 에 저장.
- **ASP.NET GCS 서버**는 중앙 제어 허브로서, Unity(WebGL)에는 **상태 스트림을 전송**, ESP32 드론에는 **UDP/TCP 제어 신호를 송신**.
- Unity는 렌더링 전담 시뮬레이터, ESP32는 실기 드론으로 현실 세계 테스트가 가능.
