# good-restuarant project

## github project

<https://github.com/orgs/good-restaurant/projects/1>

## entire project architecture

```mermaid
flowchart TD

    %% 클라이언트 계층
    user --> gs-client
    class user,gs-client clientLayer

    gs-client --> gs-main-api
    gs-client --> gs-s3object-api
    gs-client --> gs-geoserver

    %% 서버간 통신
    gs-main-api --> gs-s3object-api

    %% 애플리케이션 계층
    gs-main-api --> gs-database
    gs-s3object-api --> gs-database
    class gs-main-api,gs-s3object-api,gs-geoserver,gs-database appLayer

    %% 오브젝트 스토리지 계층
    gs-s3object-api --> cephRGW --> cephVM
    class cephRGW,cephVM storageLayer

    %% 컨테이너 호스팅 계층
    gs-client --> nodecontainer
    gs-main-api --> jvmcontainer
    gs-geoserver --> containerGeoserver
    gs-database --> postgrescontainer
    gs-s3object-api --> jvmcontainer

    nodecontainer --> runnerVM
    containerGeoserver --> runnerVM
    postgrescontainer --> runnerVM
    jvmcontainer --> runnerVM
    class nodecontainer,containerGeoserver,postgrescontainer,jvmcontainer containerLayer

    %% VM 및 호스트
    cephVM --> proxmoxNode
    runnerVM --> proxmoxNode
    class runnerVM,proxmoxNode infraLayer


    %% 색상 정의
    classDef clientLayer fill:#90c8ff,stroke:#1e4f9c,stroke-width:1px
    classDef appLayer fill:#b5e8b0,stroke:#1d7c3a,stroke-width:1px
    classDef storageLayer fill:#ffd39c,stroke:#b26b00,stroke-width:1px
    classDef containerLayer fill:#dab6ff,stroke:#7c3aed,stroke-width:1px
    classDef infraLayer fill:#d7d7d7,stroke:#6b6b6b,stroke-width:1px


```

## each service

### S3

```mermaid
sequenceDiagram
    actor Client
    participant Frontend
    participant Backend
    participant S3API
    participant RGW

    Client ->> Frontend: 이미지 업로드 요청
    Frontend ->> Backend: 프리사인 요청
    Backend ->> S3API: 프리사인 발급 요청
    S3API ->> RGW: 내부 처리 요청
    RGW ->> S3API: 프리사인 데이터 전달
    S3API ->> Backend: 프리사인 URL 반환
    Backend ->> Frontend: 프리사인 전달
    Client ->> RGW: 실제 이미지 업로드
    RGW ->> Client: 업로드 완료 응답
```

### geoserver

```mermaid
sequenceDiagram
    actor Client
    participant GeoServer
    participant Storage

    Client ->> GeoServer: WMS WFS GWC 요청
    GeoServer ->> Storage: 공간 데이터 조회 요청
    Storage ->> GeoServer: GeoTIFF 또는 벡터 데이터 반환
    GeoServer ->> GeoServer: 좌표 변환과 래스터 렌더링 수행
    Note right of GeoServer: 렌더링 결과 캐싱
    GeoServer ->> Client: PNG 이미지 전달
```

### CI/CD

```mermaid
sequenceDiagram
    actor Developer as 개발자
    participant GitHub
    participant GitLabRepository as GitLab
    participant Runner as 러너
    participant Service as 각 서비스

    Developer ->> GitHub: 코드 푸시
    GitHub ->> GitLabRepository: github action <br> 레포지토리 푸시

    opt CI 단계
        GitLabRepository ->> Runner: 파이프라인 트리거
        Runner ->> GitLabRepository: 소스코드 가져오기
        Runner ->> Runner: 빌드 이미지 생성
        Runner ->> GitLabRepository: 이미지 태깅과 레지스트리 업로드
        end

    opt CD 단계
        GitLabRepository ->> Runner: 레지스트리 다운로드
        Note right of Runner: 애플리케이션 수준
        Runner ->> Service: 새 버전 배포
        Note right of Runner: 데이터베이스의 경우
        Runner ->> Service: 새로운 설정 반영
    end

        Runner ->> Developer: 파이프라인 성공 알림
```

### FE/BE

```mermaid
sequenceDiagram
    actor Client
    participant Frontend
    participant Backend
    participant DB

    Client ->> Frontend: Web UI 접근
    Frontend ->> Backend: API 요청
    Backend ->> DB: 데이터 조회 또는 저장
    DB ->> Backend: 결과 반환
    Backend ->> Frontend: 응답 반환
    Frontend ->> Client: 렌더링된 결과 표시
```
