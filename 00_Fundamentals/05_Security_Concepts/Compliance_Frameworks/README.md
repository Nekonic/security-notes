# 보안 프레임워크 & 컴플라이언스

보안을 "체계적으로" 하기 위한 표준·프레임워크·벤치마크 모음.
**무엇을 지켜야 하는가(거버넌스)** 의 기준선이 된다. 특정 도메인 적용은 각 도메인에서 링크로 참조한다.

> 클라우드 특화 적용(CSPM, ISO 27017, CIS AWS 등)은
> [06_Infra_Pentesting/06_Cloud_Security/Compliance_CSPM](../../../06_Infra_Pentesting/06_Cloud_Security/Compliance_CSPM/) 참고.

---

## 1. 분류 — 프레임워크는 "성격"이 다르다

세 가지를 헷갈리면 안 된다:

| 유형 | 질문 | 예시 |
|------|------|------|
| **거버넌스 프레임워크** | 보안을 *어떻게 관리/운영* 할까 (방향성) | NIST CSF, ISO 27001 |
| **컨트롤 카탈로그** | *무슨 통제 항목* 을 둘까 (목록) | ISO 27002, CSA CCM |
| **벤치마크/체크리스트** | *구체적으로 어떤 설정값* 으로 (실행) | CIS Benchmark |

→ CSF로 방향 잡고 → 27002/CCM에서 통제 고르고 → CIS로 실제 설정에 적용.

---

## 2. NIST CSF (Cybersecurity Framework)

미국 NIST가 만든 **위험 기반 거버넌스 프레임워크**. 업종 무관 범용. 현재 **2.0**(2024).

- **6개 핵심 기능** (2.0부터 Govern 추가):
  `Govern(거버넌스)` · `Identify(식별)` · `Protect(보호)` · `Detect(탐지)` · `Respond(대응)` · `Recover(복구)`
- 강점: 비전문 경영진과도 소통 가능한 공통 언어. 다른 표준과 매핑 쉬움.
- 문서:
  - [NIST CSF 2.0 (CSWP 29)](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf)
  - [CSF 2.0 시작 가이드 (SP 1300)](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.1300.pdf)

---

## 3. ISO/IEC 27000 계열

국제표준화기구의 정보보안 표준군. **인증(certification)** 받을 수 있는 게 특징.

| 표준 | 내용 |
|------|------|
| **ISO 27001** | ISMS(정보보안 경영시스템) **요구사항**. 인증 대상. "무엇을 갖춰야 하나" |
| **ISO 27002** | 27001을 만족시키기 위한 **통제 실행 지침**(컨트롤 카탈로그). "어떻게 구현하나" |
| **ISO 27017** | **클라우드 서비스** 보안 통제 (27002의 클라우드 확장) → 클라우드 노트에서 상세 |

> 관계: 27001(요구) ← 27002(통제 지침) ← 27017(클라우드 추가 통제)

---

## 4. CSA (Cloud Security Alliance)

클라우드 보안 전문 비영리 단체. 클라우드 거버넌스의 사실상 표준.

- **CCM (Cloud Controls Matrix)**: 클라우드용 통제 프레임워크. 도메인별 통제 항목 + 다른 표준(ISO/NIST 등)과 **매핑** 제공.
- **STAR (Security, Trust, Assurance & Risk)**: CCM 기반의 **클라우드 사업자 인증/등록 프로그램**. (자가평가 Level 1 ~ 제3자 인증 Level 2)
- 상세 적용은 [Compliance_CSPM](../../../06_Infra_Pentesting/06_Cloud_Security/Compliance_CSPM/)에서.

---

## 5. CIS Benchmark

Center for Internet Security가 만든 **구체적 설정 체크리스트**. OS·클라우드·미들웨어별로 "이 값을 이렇게 설정하라"를 항목 단위로 제시. 자동 점검 도구와 잘 맞는다.

- 예: **CIS AWS Foundations Benchmark** (AWS 계정 기본 보안 설정)
- 클라우드별 CIS 적용 → 클라우드 노트 참조.

---

## 6. 참고

- NIST CSF 2.0: https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf
- NIST SP 1300 (Quick Start): https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.1300.pdf
- CSA: https://cloudsecurityalliance.org/
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks
