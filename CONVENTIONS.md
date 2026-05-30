# 네이밍 & 구조 규칙 (Naming Conventions)

이 저장소의 모든 폴더/파일은 아래 규칙을 따른다.

## 1. 언어
- 폴더·파일 이름은 **영어만** 사용한다. (경로 깨짐/검색 문제 방지)
- 노트 **본문 내용**은 한국어 자유.
- 예외: 외부 고유명사는 원래 이름 유지 (`webhacking.kr`, `dreamhack.io`).

## 2. 이름 형식 — `Title_Snake_Case`
- 단어는 대문자로 시작, 단어 사이는 `_` (언더스코어).
- 공백·하이픈 금지.
- ✅ `Stack_Overflow`, `CLI_Essentials`, `SQL_Injection`
- ❌ `stack overflow`, `stack-overflow`, `stackOverflow`

## 3. 계층 규칙 — 도메인 → (그룹) → 리프 → 파일
- **도메인**(최상위): `NN_` 번호 + 이름. `01_Web`, `03_Pwn`
- **그룹/세부분야**(도메인 직속 자식): `NN_` 번호.
  - `00_` = 기반/개요(**Foundations**) 자리 — 전 도메인 통일.
  - `99_` = 로그/기타 맨 뒤.
- **리프(개념) 폴더**: 취약점·주제 단위, **번호 없이** 개념명. `XSS`, `SQL_Injection`, `ROP`
- ❌ 리프 아래로 더 쪼개지 말 것 (`XSS/01_Reflected/` 금지) → 파일/문서 제목으로 구분.

## 4. 리프 노드 내부 = 파일로 구분  ★핵심
취약점 폴더에 도달하면 폴더를 더 만들지 말고 한 주제의 *이론·공격·방어·실습*을 한 폴더에 응집시킨다.
```
SQL_Injection/
├── README.md              ← 원리 + 공격기법 + 방어/완화(시큐어코딩) 통합
├── exploit_blind_sqli.py  ← 익스플로잇/도구
├── poc_time_based.sh      ← PoC
└── lab/                   ← 테스트 환경(docker-compose 등) 격리
```
- **대표 노트는 항상 `README.md`**. 폴더명과 같은 이름의 노트 금지.
- **방어/완화는 별도 폴더 없이** README 안 한 섹션으로.
- 실습/PoC/Docker는 그 주제 폴더 안 `lab/`에. (도메인별 분리 X)

## 5. 확장자 / 파일 이름
- 이론·개념·방어: `README.md` (보조 노트는 `Title_Snake_Case.md`)
- 익스플로잇: `exploit_*.py` / `exploit_*.c`
- PoC·스크립트: `poc_*.sh`, `poc_*.py`
- 플래그/풀이 메모: `.md`, 단순 출력 로그: `.txt`

## 6. 전역 vs 도메인 자원
- `09_Resources_and_Tools` = **전역 재사용 자산만** (워드리스트·치트시트·공용 스크립트·환경).
- 특정 취약점 전용 익스플로잇은 해당 리프 폴더에. (중복 저장 금지)
- `99_CTF_and_Wargames` = 순수 대회/워게임 **풀이 기록**만 (학습자료와 분리).

## 7. 파일 형식 (자동 강제됨)
- 인코딩: **UTF-8** (`.editorconfig`)
- 줄바꿈: **LF** (`.gitattributes`, 커밋 시 자동 정규화)
- 마지막 줄 개행 포함, 끝 공백 제거 (마크다운 제외)

## 8. 경계 규칙 — 교차 주제(cross-cutting)  ★중복 방지
여러 도메인에 걸치는 주제는 **폴더를 또 만들지 말고**, 1차 정착지 한 곳에만 본문을 두고
나머지에서는 상대경로 마크다운 링크 `[제목](../../경로/README.md)`로 가리킨다.
(어떤 구조도 교차 주제를 100% 분리할 수 없음)

| 주제 | 1차 정착지(본문) | 교차참조 |
|------|------------------|----------|
| 어셈블리 (일반) | `00_Fundamentals/03_Programming` | RE/Pwn Foundations에서 링크 |
| 메모리·레지스터·호출규약 | `00_Fundamentals/04_Computer_Architecture` | RE/Pwn에서 링크 |
| 완화기법(ASLR/NX/PIE/Canary) | `03_Pwn/00_Foundations` | — |
| ELF/PE 포맷·디버거 셋업 | `02_Reverse_Engineering/00_Foundations` | Pwn에서 링크 |
| 네트워킹 이론 | `00_Fundamentals/02_Networking` | — |
| PCAP/패킷 분석 | `05_Forensics/03_Network_Forensics` | — |
| 네트워크 능동 공격 | `06_Infra_Pentesting` | — |
| 네트워크 방어 (방화벽·IDS/IPS·세그멘테이션) | `07_Defense/01_Network_Defense` | Infra 공격편에서 링크 |
| 침해사고 대응 (IR 프로세스) | `07_Defense/05_Incident_Response` | Forensics 아티팩트 분석에서 링크 |
| JWT/토큰 **공격 기법** | `01_Web/03_Access_Control_Auth/Authentication` | Crypto/Attacks 링크 |
| JWT/토큰 **암호 원리·취약점** | `04_Cryptography/05_Attacks` | Web 링크 |
| Mobile / Cloud | `08_Adjacent_Domains` / `06_Infra_Pentesting` (1차 홈) | 세부는 각 도메인 링크 |

**원칙:** "기법(어떻게 공격)"은 공격 도메인에, "원리(왜 뚫리나)"는 기반 도메인에. 한쪽만 본문.
