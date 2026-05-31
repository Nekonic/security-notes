# 가상화 격리와 보안 (격리가 깨지는 지점)

가상화는 격리를 약속하지만, 그 격리는 **여러 층의 하드웨어·소프트웨어가 협력해야** 유지된다. 어느 한 층이 무너지면 격리가 깨진다. 이 노트는 "원리상 어디가 약한가"를 정리하고, OpenStack에서의 방어로 연결한다.

> 여기 = **원리(왜 깨지나)**. OpenStack 적용/하드닝 = [Nova_Compute](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/).

---

## 1. 격리의 층과 위협 지도

```
VM-A ───┐
        │ ③ 사이드채널 (공유 CPU/캐시/메모리)
VM-B ───┤
        │
[하이퍼바이저/QEMU] ← ① VM Escape (게스트→호스트)
        │ ② 자원 고갈 (DoS)
[호스트 OS / 커널]
        │ ④ 장치 DMA (IOMMU 없으면)
[하드웨어]
```

| # | 위협 | 깨지는 경계 | 1차 방어 |
|---|------|-------------|----------|
| ① | VM Escape | 게스트 → 하이퍼바이저 | QEMU/KVM 패치 + sVirt |
| ② | 자원 고갈 (DoS) | VM → 이웃 VM 성능 | cgroup, 오버커밋 통제 |
| ③ | 사이드채널 | VM ↔ VM (간접) | SMT 비활성, 마이크로코드, 전용 호스트 |
| ④ | DMA 공격 | 장치 → 호스트 메모리 | IOMMU(VT-d) |
| ⑤ | 잔존 데이터 | VM 삭제 후 → 다음 VM | 디스크/메모리 와이핑, 암호화 |

---

## 2. ① VM Escape

게스트가 하이퍼바이저(주로 QEMU의 장치 에뮬레이션) 취약점을 악용해 **호스트에서 코드 실행**.

- 대표 사례:
  - **VENOM** (CVE-2015-3456): 플로피 디스크 컨트롤러 에뮬레이션 버그
  - QEMU 각종 가상 장치(USB, 네트워크, 그래픽) 메모리 손상 버그
- 핵심 교훈: **에뮬레이트 장치 = 공격면.** 안 쓰는 장치 제거, QEMU 최신화
- 2차 방어 = **sVirt**: escape가 나도 SELinux/AppArmor 레이블로 QEMU 프로세스를 가둬 피해 차단

---

## 3. ③ 사이드채널 (가장 까다로움)

VM들이 같은 물리 CPU·캐시·메모리를 공유하기 때문에, **직접 접근 없이** 타이밍·전력 등 간접 신호로 이웃 VM 정보를 추정.

| 공격 | 악용 대상 |
|------|-----------|
| **Spectre / Meltdown** | CPU 투기적 실행 |
| **L1TF** (Foreshadow) | L1 캐시 + SMT |
| **MDS** (ZombieLoad 등) | CPU 내부 버퍼 |
| **캐시 공격** (Prime+Probe) | 공유 캐시 타이밍 |
| **KSM 누수** | 메모리 중복제거 타이밍 |

- 공통 원인: **자원 공유**. 특히 **SMT(하이퍼스레딩)** 는 형제 스레드가 코어 자원을 공유해 위험
- 방어: CPU 마이크로코드 패치, SMT 비활성, 민감 워크로드 **전용 호스트 분리**, KSM 비활성

> 사이드채널의 CPU 동작 원리 → [CPU_Virtualization](CPU_Virtualization.md)

---

## 4. ④ DMA 공격 & ⑤ 잔존 데이터

### DMA
PCI passthrough된 장치가 **IOMMU 없이** DMA하면 호스트 메모리 어디든 읽고 쓸 수 있음 → 즉시 격리 붕괴.
- 방어: passthrough 시 **IOMMU(VT-d/AMD-Vi) 필수** → [Memory_Virtualization](Memory_Virtualization.md), [IO_Virtualization](IO_Virtualization.md)

### 잔존 데이터 (Data Remanence)
VM을 지워도 물리 디스크/메모리에 내용이 남으면 다음에 그 자원을 받은 VM이 읽음.
- 방어: ephemeral 디스크 와이핑 또는 암호화, 메모리 페이지 초기화
- OpenStack 적용 → [Nova Ephemeral_Data](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/README.md)

---

## 5. 컨테이너의 경우

컨테이너는 **커널을 공유**하므로 격리가 더 얕다. 커널 syscall 취약점 하나로 탈출 가능.
- 멀티테넌트 + 강한 격리가 필요하면 VM 기반이거나 Kata/gVisor 같은 샌드박스
- 상세 → [Containers_vs_VMs](Containers_vs_VMs.md)

---

## 6. OpenStack/Nova로 연결

| 원리 위협 | Nova 방어 노트 |
|-----------|----------------|
| VM Escape, 사이드채널 | [Hypervisor_Isolation](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/Hypervisor_Isolation.md) |
| co-residency (적대 테넌트 공존) | [Scheduler_Isolation](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/README.md) |
| 잔존 데이터 | [Ephemeral_Data](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/README.md) |
| 마이그레이션 도청 | [Live_Migration](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/Live_Migration.md) |

---

## 7. 참고

- 사이드채널: Spectre/Meltdown/L1TF/MDS 공식 권고
- [OpenStack Security Guide — Compute](https://docs.openstack.org/security-guide/compute.html)
