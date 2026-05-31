# CPU 가상화

게스트 OS는 자기가 진짜 CPU를 독점한다고 믿는다. 하지만 실제로는 여러 게스트가 물리 CPU를 나눠 쓴다. 이 환상을 어떻게 만드는가.

---

## 1. 문제 — 보호 링(Protection Ring)

x86 CPU는 권한을 4단계 링으로 나눈다.

```
Ring 0  ← 커널 (특권 명령 실행 가능)
Ring 1,2 ← (거의 안 씀)
Ring 3  ← 사용자 앱
```

- 원래 **OS 커널이 Ring 0**에서 특권 명령(하드웨어 제어)을 실행
- 그런데 가상화하면? 게스트 커널도 Ring 0을 원하지만, 거기엔 **하이퍼바이저**가 있어야 한다
- 게스트 커널을 Ring 0에 두면 하드웨어를 직접 만져 다른 VM을 침범 → 안 됨

→ **게스트 커널의 특권 명령을 어떻게든 가로채야** 한다. 이게 CPU 가상화의 본질.

---

## 2. 해결 방식의 진화

### (1) 트랩 & 에뮬레이트 (이상적)
게스트가 특권 명령을 실행하면 CPU가 **트랩**(예외) 발생 → 하이퍼바이저가 가로채 대신 안전하게 처리.
- 문제: x86에는 트랩 없이 조용히 실패하는 **"민감하지만 비특권"인 명령**들이 있어 순수 트랩&에뮬레이트가 불가능했음 (x86의 가상화 구멍)

### (2) 바이너리 변환 (Binary Translation) — 초기 VMware
게스트 코드를 실행 직전에 스캔해 위험한 명령을 안전한 것으로 **번역**. 동작하지만 느리고 복잡.

### (3) 반가상화 (Paravirtualization) — Xen 초기
게스트 OS를 수정해, 특권 명령을 직접 하지 말고 하이퍼바이저에게 **하이퍼콜(hypercall)** 로 요청. 빠르지만 게스트 OS 수정 필요.

### (4) 하드웨어 보조 가상화 (현재 표준) ⭐
CPU에 가상화 전용 모드를 추가 → **Intel VT-x / AMD-V**.

---

## 3. 하드웨어 보조 가상화 (VT-x / AMD-V)

CPU에 **새로운 실행 모드**를 도입:

```
VMX root 모드     ← 하이퍼바이저가 여기서 동작
VMX non-root 모드 ← 게스트가 여기서 동작 (Ring 0~3 그대로 가짐)
```

- 게스트는 자기 Ring 0을 가져서 OS 수정이 불필요(**전가상화**)
- 게스트가 민감한 동작을 하면 → **VM Exit**(non-root→root 전환)으로 하이퍼바이저에 제어가 넘어감
- 하이퍼바이저 처리 후 → **VM Entry**로 게스트로 복귀

| 용어 | 뜻 |
|------|-----|
| **VMCS** (Intel) | VM별 상태를 저장하는 제어 구조 |
| **VM Exit / Entry** | 게스트↔하이퍼바이저 전환 (비쌈 — 잦으면 성능 저하) |
| **하이퍼콜** | 게스트가 하이퍼바이저에 일을 요청 (반가상화) |

---

## 4. 보안 관점

- **VM Exit 경로가 공격면**: 게스트가 VM Exit를 유발해 하이퍼바이저 코드를 건드림 → 버그 있으면 [VM Escape](Isolation_Security.md)
- **CPU 상태 누수**: 컨텍스트 전환 시 레지스터·캐시가 깨끗이 분리 안 되면 정보 누출
- **사이드채널**: Spectre/Meltdown은 CPU의 투기적 실행을 악용 → VM 경계를 우회. SMT(하이퍼스레딩) 공유 시 위험
  → 상세 연결: [Isolation_Security](Isolation_Security.md), [Nova Hypervisor_Isolation](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/Hypervisor_Isolation.md)

---

## 5. 참고

- Intel VT-x / AMD-V 아키텍처 매뉴얼
- 사이드채널(Spectre/Meltdown) → [Isolation_Security](Isolation_Security.md)
