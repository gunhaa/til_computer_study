# Git 내부구조

> 참고: https://git-scm.com/book/ko/v2

- Git은 Content-addressable 파일 시스템이다
  - 즉, Git의 핵심은 단순한 Key-Value 데이터 저장소
- object database 시스템으로 구성되어 있다
- HEAD 파일, index 파일, objects 디렉토리, refs 디렉토리로 구성되어 있으며, 이 네 항목이 Git의 핵심
  - objects 디렉토리는 모든 컨텐트를 저장하는 데이터베이스
  - refs 디렉토리에는 커밋 개체의 포인터(브랜치, 태그, 리모트 등)를 저장
  - HEAD 파일은 현재 Checkout 한 브랜치를 가리킴
  - index 파일은 Staging Area의 정보를 저장
- Tag/Commit/Tree/Blob 4가지 핵심 객체로 구성 되어있다
- 각 파일의 전체를 저장하며, 이전의 VCS들과 달리 스냅샷을 저장한다
- Linux 파일 시스템 inode - dentree구조와 비슷하다

## Tag

- (TODO)상대적으로 덜 사용되어, 차후 사용시 추가
- 특정 커밋에 대한 불변의 이정표

## Commit

- 파일명이 SHA1 해시값이고(Commit, Blob, Tree) 모두, 필요한 키값(SHA1) 각 파일에 인덱싱되어있고 그 파일내부에 압축된 데이터가 있다
  - 내용은 암호화가 아닌, 효율을 위해 zlib으로 압축하여 저장
  - SHA1로 계산된 Hash값
- Commit에 대해 기록되는 객체이다
- 소유 키 값
  - Root Tree의 Key
  - Parent Commit의 Key

## Tree

- Tree/Blob 객체를 Composition 관계로 소유하고 있다
- Tree/Blob의 recursive하게 표현 가능하다
- Directory를 표현하는 객체이며, 내부에 Blob 객체를 가지고 있을 수 있다
- Blob을 인덱싱하며 dentree와 비슷한 인덱싱 구조를 가지고있다
  - 관리하는 파일명을 가지고 있다
- 소유 키 값
  - Tree의 Key
  - Blob의 Key

## Blob

- 파일의 내용을 저장
- 소유 키 값
  - Blob의 Key
