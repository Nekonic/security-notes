# KVM · QEMU · libvirt

리눅스 가상화의 세 주역. 셋의 역할이 헷갈리기 쉬운데, 한 문장으로 정리하면:

> **KVM**은 빠르게 하는 가속기, **QEMU**는 VM 본체, **libvirt**는 관리자.

OpenStack Nova도 이 스택 위에서 돈다. (`nova-compute → libvirt → QEMU/KVM`)

---

## 1. 세 계층

| 계층 | 정체 | 역할 |
|------|------|------|
| **KVM** | 리눅스 **커널 모듈** (`kvm.ko` + `kvm-intel/amd`) | CPU·메모리 하드웨어 가상화(VT-x/AMD-V)를 유저공간에 노출 |
| **QEMU** | **유저공간 프로세스** (`qemu-system-x86_64`) | 장치 에뮬레이션 + KVM 호출. **이 프로세스 1개 = VM 1개** |
| **libvirt** | 관리 **데몬/API** (`libvirtd`, `virsh`) | VM 정의(XML)·생명주기 관리, QEMU 프로세스를 띄우고 감독 |

---

## 2. 각자 왜 필요한가

### KVM — 가속기
- 그냥 QEMU만 쓰면 CPU 명령을 **소프트웨어로 에뮬레이션** → 느림.
- KVM은 CPU의 가상화 확장(VT-x/AMD-V)을 써서 게스트 명령을 **하드웨어에서 직접** 실행 → 네이티브에 가까운 속도.
- 인터페이스는 `/dev/kvm` 장치 파일. QEMU가 이걸 열어서 가속을 요청한다.
- KVM 단독으로는 VM을 못 만든다 — **CPU/메모리만** 담당. 디스크·네트워크 같은 장치는 QEMU 몫.

### QEMU — VM 본체
- 게스트가 보는 **가상 하드웨어**(디스크, NIC, 그래픽, BIOS)를 제공.
- 가속이 있으면 `KVM`에 CPU를 맡기고, 자기는 장치 에뮬레이션에 집중.
- 호스트에서 보면 **그냥 하나의 프로세스**다. `ps`로 `qemu-system-x86_64`가 VM 개수만큼 보인다.

### libvirt — 관리자
- QEMU를 직접 긴 커맨드라인으로 띄우는 건 복잡 → libvirt가 **XML 정의**로 추상화.
- VM 시작/정지/스냅샷/마이그레이션을 표준 API로 제공. (`virsh`가 그 CLI)
- KVM/QEMU뿐 아니라 Xen, LXC 등도 같은 API로 다룰 수 있게 하는 **추상화 계층**.
- OpenStack Nova는 QEMU를 직접 안 만지고 **libvirt API로** 제어한다.

---

## 3. 어떻게 맞물리나

```
libvirt (libvirtd)
   │  XML 정의를 읽어 QEMU 커맨드라인 생성, 프로세스 실행
   ▼
QEMU (qemu-system-x86_64)   ← 유저공간 프로세스 = VM 본체
   │  /dev/kvm 열어 CPU/메모리 가속 요청
   ▼
KVM (커널 모듈)             ← VT-x/AMD-V로 하드웨어 가상화
   │
물리 CPU / 메모리
```

- 디스크 I/O, 네트워크 패킷 같은 **장치 동작은 QEMU**가 처리(또는 virtio로 게스트와 협력).
- CPU 명령 실행·메모리 매핑 같은 **특권 동작은 KVM**이 하드웨어로 처리.

---

## 4. 실물 흔적 (어디서 확인하나)

| 보고 싶은 것 | 명령/위치 |
|--------------|-----------|
| KVM 모듈 로드 여부 | `lsmod \| grep kvm`, `/dev/kvm` 존재 |
| 실행 중인 QEMU(=VM) | `ps -ef \| grep qemu-system` |
| libvirt가 아는 VM 목록 | `virsh list --all` |
| VM의 정의(XML) | `virsh dumpxml <도메인>` |

> OpenStack 인스턴스의 실제 도메인 XML 예시 →
> [Nova lab의 instance-00000004.xml](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/lab/instance-00000004.xml)
> (그 XML의 `<domain type='qemu'>`, `<emulator>qemu-system-x86_64`가 바로 이 스택)

---

## 5. 관련 노트
- CPU 가속의 원리(VT-x/VM Exit) → [CPU_Virtualization](CPU_Virtualization.md)
- 장치 가상화(virtio/passthrough) → [IO_Virtualization](IO_Virtualization.md)
- 격리·보안 적용 → [Nova Hypervisor 노트](../../../06_Infra_Pentesting/06_Cloud_Security/OpenStack/Nova_Compute/Hypervisor_Isolation.md)
