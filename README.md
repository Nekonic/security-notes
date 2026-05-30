# Security Study Notes

해킹·보안 공부를 도메인별로 정리하는 개인 저장소.
CTF(Web·Pwn·Rev·Crypto·Forensics)와 실무 모의해킹(Infra/OSCP)을 함께 다룬다.

> 폴더/파일 네이밍과 구조 규칙은 [CONVENTIONS.md](CONVENTIONS.md) 참고.

---

## 디렉터리 맵

| # | 도메인 | 내용 |
|---|--------|------|
| `00` | [Fundamentals](00_Fundamentals/) | Linux · Networking · Programming · Computer Architecture · Security Concepts |
| `01` | [Web](01_Web/) | Client-Side · Server-Side · Access Control/Auth · Business Logic · Protocol/Infra |
| `02` | [Reverse Engineering](02_Reverse_Engineering/) | Static · Dynamic · Hooking · Anti-Reversing · Malware Analysis |
| `03` | [Pwn](03_Pwn/) | Shellcoding · Stack · Format String · Heap · Integer/Type · Kernel |
| `04` | [Cryptography](04_Cryptography/) | Symmetric · Asymmetric · Hashing/MAC/KDF · Protocols · Attacks |
| `05` | [Forensics](05_Forensics/) | Disk · Memory · Network · Steganography |
| `06` | [Infra Pentesting](06_Infra_Pentesting/) | Recon · Initial Access · PrivEsc · Active Directory · Pivoting · Cloud |
| `07` | [Defense](07_Defense/) | Network Defense · Endpoint Hardening · Logging/Monitoring · Detection Engineering · Incident Response |
| `08` | [Adjacent Domains](08_Adjacent_Domains/) | OSINT · Mobile · Hardware/IoT |
| `09` | [Resources & Tools](09_Resources_and_Tools/) | Wordlists · Cheatsheets · Scripts · Environments |
| `99` | [CTF & Wargames](99_CTF_and_Wargames/) | Platforms · Writeups · Templates |

---

## 진행 현황

`- [x]` 정리 완료 · `- [ ]` 미완

### 00. Fundamentals
- [ ] Linux
- [ ] Networking
- [ ] Programming (C · Python · Bash · Assembly)
- [ ] Computer Architecture
- [ ] Security Concepts (Compliance Frameworks 작성 중)

### 01. Web
- [ ] Foundations (HTTP · Encoding · SOP/CORS · Automation)
- [ ] Client-Side (XSS · CSRF · Clickjacking · Open Redirect · Prototype Pollution · CORS Misconfig)
- [ ] Server-Side (SQLi · NoSQLi · Cmd Injection · Path Traversal · File Upload · SSRF · SSTI · XXE · Deserialization)
- [ ] Access Control / Auth (IDOR · Authentication)
- [ ] Business Logic (Logic Flaws · Race Condition)
- [ ] Protocol / Infra (Request Smuggling · Web Cache Poisoning · WebSockets · API/GraphQL · Info Disclosure)

### 02. Reverse Engineering
- [ ] Foundations
- [ ] Static Analysis
- [ ] Dynamic Analysis
- [ ] Hooking / Instrumentation
- [ ] Anti-Reversing
- [ ] Malware Analysis

### 03. Pwn
- [ ] Foundations (메모리 레이아웃 · 완화기법 ASLR/NX/PIE/Canary)
- [ ] Shellcoding
- [ ] Stack
- [ ] Format String
- [ ] Heap
- [ ] Integer / Type
- [ ] Kernel

### 04. Cryptography
- [ ] Foundations
- [ ] Symmetric
- [ ] Asymmetric
- [ ] Hashing / MAC / KDF
- [ ] Protocols
- [ ] Attacks

### 05. Forensics
- [ ] Foundations
- [ ] Disk
- [ ] Memory
- [ ] Network Forensics
- [ ] Steganography

### 06. Infra Pentesting
- [ ] Methodology
- [ ] Recon / Enumeration
- [ ] Initial Access
- [ ] Privilege Escalation
- [ ] Active Directory
- [ ] Pivoting / Tunneling
- [ ] Cloud Security (OpenStack 하드닝 · Compliance/CSPM 작성 중)

### 07. Defense
- [ ] Foundations
- [ ] Network Defense (방화벽 · IDS/IPS · 세그멘테이션 · 제로트러스트)
- [ ] Endpoint Hardening
- [ ] Logging / Monitoring (SIEM)
- [ ] Detection Engineering (탐지 룰 · Sigma/YARA)
- [ ] Incident Response

### 08. Adjacent Domains
- [ ] OSINT
- [ ] Mobile
- [ ] Hardware / IoT

### 99. CTF & Wargames
- [ ] Platforms (bandit · webhacking.kr · dreamhack.io)
- [ ] Writeups
- [ ] Templates

---

## 사용 규칙 (요약)

- **1개념 = 1폴더**, 그 안은 폴더를 더 쪼개지 않고 파일로 구분한다.
  - `README.md` — 원리 + 공격 기법 + 방어/완화 통합
  - `exploit_*.py` / `poc_*.sh` — 익스플로잇·PoC
  - `lab/` — 실습 환경(Docker 등)
- 여러 도메인에 걸치는 주제는 **1차 정착지 한 곳**에만 본문을 두고 상대경로 링크로 교차참조. ([CONVENTIONS.md](CONVENTIONS.md)의 "경계 규칙" 참고)
- 전역 도구·워드리스트는 `09_Resources_and_Tools`에만, 대회 풀이는 `99_CTF_and_Wargames`에만.

---

## 환경

- 인코딩 **UTF-8**, 줄바꿈 **LF** 강제 (`.editorconfig`, `.gitattributes`)
- `.venv/` 등은 `.gitignore`로 제외 — 노트와 코드만 버전 관리
