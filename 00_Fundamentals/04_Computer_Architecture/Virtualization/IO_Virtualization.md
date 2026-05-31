# I/O 가상화

게스트는 디스크·네트워크 카드 같은 장치도 필요하다. 물리 장치는 하나인데 여러 VM이 써야 한다. 장치를 어떻게 나눠 줄 것인가 — 성능과 격리가 정면으로 부딪히는 영역.

---

## 1. 세 가지 방식 (성능 ↑, 격리 통제 ↓ 순)

```
①  에뮬레이션      ②  반가상화(virtio)    ③  패스스루(passthrough)
느림/안전           중간/표준              빠름/주의
호스트가 가짜 장치    게스트가 협력           물리 장치를 VM에 직접
완전 통제            드라이버 공유           IOMMU로 격리 필수
```

---

## 2. ① 전체 에뮬레이션 (QEMU)

하이퍼바이저(QEMU)가 실제 하드웨어(예: Intel e1000 NIC, IDE 디스크)를 **소프트웨어로 흉내**.
- 장점: 게스트가 표준 드라이버를 그대로 씀 (수정 불필요), 호스트가 모든 I/O를 가로채 **완전 통제**
- 단점: 모든 I/O가 VM Exit → 에뮬레이션 → 매우 **느림**
- 보안: 에뮬레이트하는 가상 장치 코드 자체가 **공격면** (예: VENOM = 플로피 컨트롤러 에뮬레이션 버그로 VM Escape)

> 교훈: **안 쓰는 에뮬레이트 장치는 제거**하면 공격면이 준다.

---

## 3. ② 반가상화 — virtio (현재 표준)

게스트에 **virtio 드라이버**를 설치해, 게스트와 하이퍼바이저가 **공유 링버퍼(virtqueue)** 로 효율적으로 데이터를 주고받음.
- 게스트가 "나는 VM이다"를 알고 협력 → 에뮬레이션보다 훨씬 빠름
- `virtio-net`, `virtio-blk`, `virtio-scsi` 등
- OpenStack/KVM의 기본 선택
- 보안: 에뮬레이션보다 공격면 작지만, virtio 백엔드(vhost) 자체의 버그는 여전히 위험

### vhost
virtio 데이터 경로를 QEMU(유저공간)에서 **커널(vhost-net)** 또는 별도 프로세스로 내려 더 빠르게. 성능↑이지만 경로가 커널이면 영향 범위도 커짐.

---

## 4. ③ 디바이스 패스스루 & SR-IOV

### PCI Passthrough
물리 PCI 장치(GPU, NIC)를 **특정 VM에 통째로** 할당. 거의 네이티브 성능.
- **반드시 IOMMU(VT-d/AMD-Vi) 필요** — 없으면 장치 DMA로 호스트 메모리 임의 접근 (치명적)
- 한 장치 = 한 VM 전용 (공유 불가)

### SR-IOV (Single Root I/O Virtualization)
하나의 물리 장치(주로 NIC)가 스스로를 여러 개의 **가상 기능(VF)** 으로 쪼개, 여러 VM에 나눠 줌.
```
물리 NIC
├── PF (Physical Function)  ← 호스트가 관리
├── VF1 → VM-A
├── VF2 → VM-B
└── VF3 → VM-C
```
- 패스스루급 성능 + 공유 가능
- 보안: VF 간 격리는 하드웨어/펌웨어에 의존 → 펌웨어 취약점 시 VF 경계 우회 위험

---

## 5. 보안 관점 요약

| 방식 | 공격면 | 핵심 방어 |
|------|--------|-----------|
| 에뮬레이션 | 가상 장치 코드 버그 (VM Escape) | 불필요 장치 제거, QEMU 패치 |
| virtio | virtio/vhost 백엔드 버그 | 최신화, sVirt로 가둠 |
| passthrough | IOMMU 미설정 시 DMA 공격 | **IOMMU 필수**, 장치 초기화/잔존 |
| SR-IOV | VF 격리 우회(펌웨어) | 펌웨어 업데이트, 신뢰 테넌트만 |

→ OpenStack 적용: [Nova Hypervisor_Isolation](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/Hypervisor_Isolation.md), 네트워크 측면은 [Neutron](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Network_Neutron/)

---

## 6. 참고

- [virtio 스펙](https://docs.oasis-open.org/virtio/virtio/v1.2/virtio-v1.2.html)
- IOMMU/SR-IOV → [Memory_Virtualization](Memory_Virtualization.md)
