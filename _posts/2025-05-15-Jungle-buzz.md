---
layout: post
title: Jungle Buzz
date: 2025-05-15
---

실시간 채팅 기반 감정 공유 플랫폼

- **깃허브** : [Github](https://github.com/suinkimme/jungle-buzz.git)
- **개발 인원** : 3인
- **프로젝트 진행 일자** : 2025. 05. 12. ~ 2025. 05. 15.
- **맡은 역할** : **Python Flask 기반 백엔드 API 서버 및 실시간 WebSocket 통신 서버 구축**

---

## 프로젝트 배경

- **기획** : 기존 채팅 플랫폼은 단순 메시지 교환에만 집중하여, 사용자들의 감정 상태나 커뮤니티 분위기를 파악하기 어려움. 실시간 소통과 감정 분석을 통한 새로운 소셜 경험 필요.
- **기술** : 실시간 채팅과 AI 감정 분석을 단일 플랫폼에서 처리하고, 성능 최적화를 위한 버퍼링 시스템과 이중 서버 구조로 안정적인 서비스 제공이 필요.
- **목표** : Flask 기반 REST API–WebSocket–AI 분석을 잇는 통합 백엔드로 *실시간 소통·감정 인사이트·확장 가능한 아키텍처*를 구현.

---

## 프로젝트 문제

- **단일 서버 구조**: 하나의 서버에서 REST API와 실시간 통신을 모두 처리 → 높은 부하시 성능 저하, 메시지 손실 위험.
- **DB 쓰기 병목**: 채팅 메시지를 실시간으로 개별 저장 → 빈번한 DB 접근으로 성능 저하.
- **확장성 한계**: 모놀리식 구조로 인해 기능별 독립적인 스케일링 어려움.

---

## 해결과정

- **이중 서버 분리**: REST API 서버(`app.py`)와 WebSocket 서버(`ws_server.py`)를 완전히 분리 → **역할별 최적화**.
- **버퍼링 시스템 도입**: 채팅 메시지를 메모리 버퍼에 쌓아두고 **배치 처리**로 DB 저장 → 쓰기 성능 향상.
- **JWT 기반 인증**: 토큰 기반 무상태 인증으로 **서버 간 세션 공유** 문제 해결.

- **결과**:
  - DB 쓰기 요청 **70% 감소** (개별 저장 → 배치 처리)
  - 실시간 메시지 전송 지연 **50ms 이하** 유지
  - **AI 감정 분석** 기능으로 커뮤니티 인사이트 제공

---

## 기술적 의사결정 과정

- **Flask + Socket.IO** : Python 생태계 활용 + 빠른 프로토타이핑 → 개발 속도와 안정성 확보.
- **MongoDB** : 채팅 데이터의 스키마 유연성 + 높은 쓰기 성능 → NoSQL의 장점 활용.
- **통신 방식** :
  - REST API → 사용자 인증, 데이터 조회 (HTTP 표준)
  - WebSocket → 실시간 채팅 (양방향, 저지연)
- **AI 분석** : OpenAI GPT-4o-mini → 24시간 주기 감정 분석 및 인사이트 생성.

---

## 아키텍쳐 설명

**이중 서버 구조**로 REST API와 실시간 통신을 분리하여 성능과 확장성을 확보.

- 사용자는 **REST API 서버**를 통해 인증/회원가입/채팅 히스토리 조회를 수행.
- **WebSocket 서버**는 실시간 채팅 전담으로, JWT 검증 후 룸 기반 메시지 브로드캐스트.
- **버퍼링 시스템**으로 채팅을 메모리에 임시 저장 → 일정 주기/크기마다 MongoDB에 배치 저장.
- **AI 스케줄러**가 매일 오전 9:50에 24시간 채팅 데이터를 분석하여 감정 인사이트 생성.

---

## 핵심 구현 기능

### JWT 기반 인증 시스템

```python
@token_required
def decorated(*args, **kwargs):
    decoded = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
```

- Bearer 토큰 방식으로 API 보안 강화
- 무상태 인증으로 서버 간 세션 공유 문제 해결

### 버퍼링 기반 성능 최적화

```python
chat_buffer = []
MAX_BUFFER_SIZE = 10
FLUSH_INTERVAL = 2  # 초

def flush_chat_buffer():
    # 비동기 배치 저장 로직
```

- 채팅 메시지를 메모리 버퍼에 임시 저장
- 버퍼 크기 초과 또는 시간 초과시 자동 플러시

### 실시간 WebSocket 통신

```python
@socketio.on("send_chat")
def handle_send_chat(data):
    # JWT 검증 후 실시간 브로드캐스트
    emit("new_chat", {...}, to="main")
```

- Socket.IO 기반 양방향 실시간 통신
- 룸 기반 채팅방 관리 및 메시지 브로드캐스트

### 페이지네이션 API (app.py)

```python
@app.route('/api/chat-logs', methods=['GET'])
@token_required
def get_chat_logs(user):
    skip = (page - 1) * page_size
    logs_cursor = chat_col.find({'username': user}).sort('timestamp', -1).skip(skip).limit(page_size)
```

- MongoDB 기반 효율적인 페이징 처리
- 사용자별 채팅 히스토리 관리

### AI 감정 분석 시스템

```python
def analyze_chat():
    chat_logs = get_recent_chat_logs(1000 * 60 * 60 * 24)  # 24시간
    response = openaiClient.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "system", "content": "감정 분석 프롬프트"}]
    )
```

- GPT-4o-mini 기반 일일 감정 분석
- 마크다운 테이블 형식으로 인사이트 제공

---

## 엔드 포인트

| 엔드포인트        | 기능      | 특징                          |
| ----------------- | --------- | ----------------------------- |
| /api/register     | 회원가입  | 비밀번호 해싱, 중복 검사      |
| /api/login        | 로그인    | JWT 토큰 발급                 |
| /api/send-chat    | 채팅 전송 | 버퍼링 저장, 토큰 검증        |
| /api/chat-logs    | 채팅 조회 | 페이지네이션, 사용자별 필터링 |
| /api/recent-chats | 최근 채팅 | 실시간 초기 로딩용            |

---

## 설계의 특장점

| 구분        | 내용                                                                     |
| ----------- | ------------------------------------------------------------------------ |
| 성능 최적화 | 버퍼링으로 DB 쓰기 부하 분산 <br> 별도 WebSocket 서버로 실시간 통신 전담 |
| 확장성      | REST API와 WebSocket 서버 분리 <br> 마이크로서비스 아키텍처 준비         |
| 사용자 경험 | 실시간 채팅 + AI 감정 분석 <br> 개인 히스토리 관리                       |
