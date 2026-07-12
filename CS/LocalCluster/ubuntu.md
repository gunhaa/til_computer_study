# N100 ubuntu 설치 SOP

## Overview

- 목적
  - 부하 테스트를 위한 N100 저전력 미니 PC 기반의 가볍고 안정적인 리눅스 로컬 클러스터 인프라 환경 구축
- 대상 시스템
  - Intel Processor N100 (4 Cores, 4 Threads)
- OS 버전
  - Ubuntu Server 24.04.4 LTS

## 설치 절차

1. Ventoy USB를 이용해 구축
2. `ubuntu-24.04.4-live-server-amd64` 설치
3. `F7`로 부팅모드 Ventoy USB로 변경
4. `ubuntu-24.04.4-live-server-amd64` 선택
5. `Boot in normal mode` 선택
6. `Try or Install Ubuntu` 선택
7. `language` -> `English` 선택
8. `Update to the new installer` 랜선이 연결되어 있다면, 버그가 픽스된 최신 installer로 변경
9. `Done`
10. `Ubuntu Server`
- Additional options `Search for third-party drivers` 추가 선택(dirver를 자동으로 잡아줌)
11. `Network configuration`
- (현재 물리적인 랜 포트의 상태) 확인 후 진행 
12. `Proxy Configuration` skip
13. `Ubuntu archive mirro configuration` skip
- apt가 앞으로 프로그램이나 업데이트 파일들을 어디서 다운로드해 그 저장소를 지정 
14. `Guided storage configuration`
- Use an entire disk 선택
- Set up this disk as an LVM group 선택 (부족시 확장 가능 하도록 설정)
- Encrypt the LVM group with LUKS 선택X (암호화 설정)
15. `Storage configuration`
- 이전 Guided 설정으로 자동으로 설정이 잡혀 특별한 경우 없다면 스킵
- 다음 선택시, 모든 데이터 삭제 경고 메시지 출력 - 스킵
16. `Profile configuration`
- Your name(이름)
  - 설정 값: `gunha`
- Your servers name
  - 설정 값: `n100-data-node`
- Pick a username(터미널 계정 ID)
  - 설정 값: `gunha`
- Choose a password(.env 참조)
  - 설정 값: `${1EA9EA9AA61684F94C2CC204DF42C15A5ACAD68B}`
17. `Upgrate to Ubuntu Pro` skip
- 기업용 서비스 사용 여부 선택
18. `SSH configuration`
- Install OpenSSH server 선택
  - 초기 ssh연결이 가능하도록 OpenSSH 패키지 설치
- Import SSH key 선택 X
  - 다른 Key import 설정 시 사용, 설정 안됬다면 이전에 설정했던 비밀번호 사용
19. `Third-party drivers` skip
20. `Featured server snaps` skip
  - 추가 설치 선택 X
  - 미리 설치하면 설정 꼬일 수 있으니, 설치 후 apt를 이용해 필요한 것만 추가설치

## 설치 완료 확인 절차

- `hostnamectl`
  - hostname, OS 버전, architecture 확인
- `free -h`
  - RAM 확인
- `lsblk`
  - Disk 확인
- `ping -c 4 google.com`
  - 외부 인터넷 확인