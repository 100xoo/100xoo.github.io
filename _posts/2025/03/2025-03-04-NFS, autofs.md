---
layout: single
title: NFS, AutoFS
categories: Linux
tags:
  - Linux
  - File-System
toc: true
---
## **목차**

1. NFS/AutoFS 개요
2. NFS+AutoFS 구성 흐름
3. NFS+AutoFS 구성 장점 및 팁


## 1. NFS란? 
- Network File System 
- 서버 디렉터리를 네트워크로 공유, 클라이언트가 원격 마운트
- 대표적 파일 공유 프로토콜 (POSIX 권한 유지)

### 주요 특징
| 항목 | 내용 |
|---|---|
| 프로토콜 | TCP/UDP (주로 TCP) |
| 권한 관리 | 파일/디렉터리 퍼미션 유지 |
| 보안 | IP 제한, NFSv4에서는 Kerberos 가능 |
| 대표 명령 | `exportfs`, `showmount`, `mount` |

---

## 2. AutoFS란?
- 필요할 때만 마운트, 일정 시간 미사용 시 자동 언마운트
- 시스템 부하 감소, 재부팅 시 마운트 문제 방지

### 주요 특징
| 항목 | 설명 |
|---|---|
| 자동 마운트 | 디렉터리에 접근할 때 자동 마운트 |
| 비활성화 관리 | 일정 시간 사용하지 않으면 자동 해제 (timeout) |
| 설정 파일 | `/etc/auto.master` 와 `/etc/auto.*`로 관리 |
| 메모리 절약 | 필요할 때만 마운트하므로 리소스 절감 |

### 주요 구성 파일
| 파일명 | 설명 |
|---|---|
| /etc/auto.master | 마스터 설정 파일 (맵파일 지정) |
| /etc/auto.* | 실제 마운트 규칙 정의 |

---

## 3. NFS + AutoFS 구성 흐름
```
[사용자 접근] → [AutoFS 동작] → [NFS 마운트]
→ [파일 접근 가능] → [미사용 시 언마운트]
```

```
+------------------+
| NFS 서버          |
| /data/share       |
|                  |
+------------------+
           |
           | 192.168.0.0/24
           |
+------------------+
| NFS 클라이언트    |
| autofs 관리      |
| /mnt/nfs/share   |
|                  |
+------------------+
```

## 4. NFS 서버 설정 예시

### /etc/exports
```
/data/shared 192.168.0.0/24(rw,sync,no_root_squash)
```

### 서비스 재시작
```
systemctl restart nfs-server
exportfs -rv
```


## 5. 클라이언트 AutoFS 설정 예시

### /etc/auto.master
```
/mnt/nfs /etc/auto.nfs
```

### /etc/auto.nfs
```
share -fstype=nfs,rw 192.168.0.100:/data/shared
```

### 서비스 재시작
```
systemctl restart autofs
```

### 마운트 상태 확인
```
cat /proc/mounts | grep autofs
```

---
## ✅ 6. NFS+autofs 구성 장점
| 장점     | 설명                                   |
| ------ | ------------------------------------ |
| 성능     | 부팅 시 불필요한 마운트 X (필요할 때만 마운트)         |
| 자원 절약  | 필요할 때만 마운트하므로 메모리, 네트워크 부담 감소        |
| 관리 편의성 | 클라이언트 설정 변경 없이 서버 추가 가능 (auto.*만 수정) |
| 장애 복원력 | 네트워크 장애 시 즉시 언마운트, 재접속 시 자동 재마운트     |

### 📌 기타 팁
- **NFSv4 권장** (보안 및 성능 개선)
- `/etc/exports`에서 `ro`/`rw`, `no_root_squash` 등 옵션 잘 체크
- 필요하면 **NFS + Kerberos**로 보안 강화 가능
- 공유 디렉터리 권한 관리는 **NFS 서버**에서 설정  
- 클라이언트는 기본적으로 읽기/쓰기만 제어 가능  
- SELinux 활성 환경에서는 NFS 관련 보안 컨텍스트 확인 필수 (RHEL계열)  
- 네트워크 불안정한 환경에서는 autofs timeout 조정 권장
```
/etc/autofs.conf
timeout = 300  # 기본 5분, 필요시 조정
```

---
