# 가상화 (Virtualization) 기초

하나의 물리 하드웨어를 여러 개의 독립된 가상 머신(VM)으로 나눠 쓰는 기술.
클라우드/IaaS의 토대이며, **보안 격리의 근본 원리**가 여기서 나온다.

> 이 폴더는 가상화의 **원리(왜 이렇게 동작하나)** 를 다룬다.
> OpenStack에서의 **적용·하드닝**은 [Nova_Compute](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/)로.

---

## 1. 왜 가상화인가

- **자원 효율**: 놀고 있는 물리 자원을 여러 워크로드가 나눠 씀
- **격리**: 한 VM의 장애·침해가 다른 VM에 번지지 않게
- **이식성**: VM은 파일 묶음 → 스냅샷·마이그레이션·복제 용이
- **탄력성**: 필요할 때 만들고 지움 (클라우드의 본질)

핵심 개념: **VMM/하이퍼바이저**가 물리 자원을 가로채(중재) 각 VM에게 "내가 진짜 하드웨어를 독점한다"는 환상을 준다.

---

## 2. 하이퍼바이저 타입

```
[Type 1 - 베어메탈]            [Type 2 - 호스티드]
┌───────────────┐            ┌───────────────┐
│ VM │ VM │ VM  │            │ VM │ VM        │
├───────────────┤            ├──────────┬────┤
│  하이퍼바이저   │            │하이퍼바이저│App │
├───────────────┤            ├──────────┴────┤
│   하드웨어      │            │   호스트 OS    │
└───────────────┘            ├───────────────┤
                             │   하드웨어      │
                             └───────────────┘
```

| | Type 1 (베어메탈) | Type 2 (호스티드) |
|---|---|---|
| 위치 | 하드웨어 위에 직접 | 호스트 OS 위에서 |
| 예 | KVM, Xen, ESXi, Hyper-V | VirtualBox, VMware Workstation |
| 성능·격리 | 높음 (공격면 작음) | 낮음 (호스트 OS도 공격면) |
| 용도 | **서버/클라우드** | 데스크톱/개발 |

> **KVM**: 리눅스 커널이 곧 하이퍼바이저가 되는 모듈. OpenStack Nova의 기본 백엔드. (보통 QEMU와 결합해 사용)

---

## 3. 구현 스택 — 리눅스에서 실제로 누가 하나

원리를 보기 전에, 리눅스에서 가상화를 실제로 돌리는 소프트웨어 3종:

| 계층 | 노트 | 핵심 |
|------|------|------|
| KVM·QEMU·libvirt | [KVM_QEMU_libvirt](KVM_QEMU_libvirt.md) | 가속기·VM본체·관리자의 3자 관계 |

---

## 4. 가상화의 3대 축

VM에게 "가짜 하드웨어"를 만들어주려면 CPU·메모리·I/O 세 가지를 모두 가상화해야 한다. 각각 별도 노트로:

| 축 | 노트 | 핵심 |
|----|------|------|
| CPU | [CPU_Virtualization](CPU_Virtualization.md) | 보호링·VT-x/AMD-V·트랩&에뮬레이트 |
| 메모리 | [Memory_Virtualization](Memory_Virtualization.md) | shadow PT·EPT/NPT·IOMMU |
| I/O | [IO_Virtualization](IO_Virtualization.md) | 에뮬레이션·virtio·PCI passthrough·SR-IOV |

그리고 이를 보안·격리 관점에서 보는 노트:

| 주제 | 노트 |
|------|------|
| 컨테이너 vs VM (격리 강도 비교) | [Containers_vs_VMs](Containers_vs_VMs.md) |
| 가상화가 깨지는 지점 (보안 다리) | [Isolation_Security](Isolation_Security.md) |

---

## 5. 용어 빠른 정리

| 용어 | 뜻 |
|------|-----|
| **Host / Guest** | 물리 머신(호스트) / 그 위의 VM(게스트) |
| **VMM (Hypervisor)** | 가상 머신을 만들고 자원을 중재하는 소프트웨어 |
| **전가상화(Full)** | 게스트 OS 수정 없이 그대로 구동 (하드웨어 지원 활용) |
| **반가상화(Para)** | 게스트가 자기가 VM임을 알고 협력 (virtio 등) → 빠름 |
| **트랩 & 에뮬레이트** | 게스트의 특권 명령을 하이퍼바이저가 가로채 대신 처리 |

---

## 6. 참고

- [KVM](https://www.linux-kvm.org/) · [QEMU](https://www.qemu.org/)
- 보안 연결: [Nova Hypervisor Isolation](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/Hypervisor_Isolation.md)
