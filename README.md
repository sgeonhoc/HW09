# HW09 — 멀티플레이 숫자야구 (Number Baseball)

언리얼 엔진 5.5 기반 **멀티플레이 채팅 + 숫자야구** 게임입니다.
서버가 1~9 중 중복 없는 3자리 정답을 생성하고, 접속한 플레이어들이 채팅으로 숫자를 입력해
**S(Strike) / B(Ball) / OUT** 판정을 받으며 먼저 3S(정답)를 맞추는 사람이 승리합니다.

> 본 과제는 복습 + 심화 목적이며, 클래스 접두사는 `CX-`를 사용하지 않고 `HW-` 접두사로 작성했습니다.

---

## 🎮 게임 규칙

- 게임 시작 시 **서버**가 1~9 사이 **중복되지 않는 3자리 숫자**를 생성합니다. (예: `386`, `594`)
- 각 플레이어는 **기본 3회**의 기회를 가집니다.
- 채팅창에 3자리 숫자를 입력하면 정답과 비교해 결과를 반환합니다.
  - **S (Strike):** 숫자와 자리가 모두 일치
  - **B (Ball):** 숫자는 있지만 자리가 다름
  - **OUT:** 일치하는 숫자가 하나도 없음
  - 결과 예시: `1S1B`, `0S2B`, `OUT`
- 입력값은 엄격하게 검사합니다.
  - 3자리가 아니거나, 숫자가 아니거나, 중복된 숫자가 있으면 **무효 처리**되며 **기회를 소진하지 않습니다.**
- **승리:** 누구든 3회 안에 먼저 `3S`를 달성하면 즉시 승자 판정 → 공지 위젯 표시 → 게임 리셋
- **무승부:** 모든 플레이어가 기회를 다 썼는데도 정답을 못 맞추면 무승부 → 공지 위젯 표시 → 게임 리셋

---

## ✅ 구현 기능 (과제 평가 기준)

| 항목 | 구현 위치 |
|------|-----------|
| GameMode 서버 로직 | `AHWGameModeBase` |
| 난수(정답) 생성 로직 | `AHWGameModeBase::GenerateSecretNumber()` |
| 입력 유효성 검사 | `AHWGameModeBase::IsGuessNumberString()` |
| 판정 로직 (S/B/OUT) | `AHWGameModeBase::JudgeResult()` |
| 시도 횟수 / 상태 관리 | `AHWPlayerState` (`CurrentGuessCount` / `MaxGuessCount`, `GetPlayerInfoString()`) |
| 승리 / 무승부 / 게임 리셋 | `AHWGameModeBase::JudgeGame()`, `ResetGame()` |
| 멀티플레이 채팅 | `AHWPlayerController` + `UHWChatInput` (Server/Client RPC) |
| C++ 라이브러리 | `ChatXFunctionLibrary` (`Source/ChatX/ChatX.h`) |

---

## 🗂 클래스 구조

```
Source/ChatX/
├── ChatX.h / ChatX.cpp          # 모듈 + ChatXFunctionLibrary (PrintString, NetMode/Role 유틸)
├── Game/
│   ├── HWGameModeBase.*         # 정답 생성, 입력 검사, 판정, 승/무/리셋 (서버 권한)
│   └── HWGameStateBase.*        # 접속 알림 멀티캐스트 RPC
├── Player/
│   ├── HWPlayerController.*     # 채팅 입출력, Server/Client RPC, 공지 텍스트(Replicated)
│   ├── HWPlayerState.*          # 플레이어 이름 / 시도 횟수 상태 (Replicated)
│   └── HWPawn.*                 # 디버그용 폰
└── UI/
    └── HWChatInput.*            # 채팅 입력 위젯 바인딩
```

블루프린트(`Content/ChatX/Blueprint/`)는 위 C++ 클래스를 부모로 하는 BP / 위젯입니다.

---

## ▶ 실행 방법

1. `ChatX.uproject` 우클릭 → **Generate Visual Studio project files**
2. 솔루션 빌드 (Development Editor / Win64) 또는 에디터에서 빌드
3. 에디터 상단 **Play** 드롭다운에서 **Number of Players ≥ 2**, **Net Mode: Play As Listen Server** 설정
4. 플레이 후 각 클라이언트 채팅창에 3자리 숫자 입력
5. 서버 정답은 `Output Log`(LogTemp, Error)에서 확인 가능

---

## 🛠 개발 환경

- Unreal Engine **5.5**
- Visual Studio 2022 / C++17
- 모듈명: `ChatX`
