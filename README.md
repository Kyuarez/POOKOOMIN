# 🐾 뿌꾸민 – POOKOOMIN

## 🎮 개요
<div align="center">
  <img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/57113b96-38cd-4d20-9e76-d01f9bf76084" />
</div>
닌텐도의 AR 게임 「피크민 블룸」을 보고 영감을 받아 만든 AR 게임입니다. 🌳

* **프로젝트 이름**: Pookoomin 🏠
* **프로젝트 지속기간**: 2025.06.13 ~ 2025.06.27
* **개발 엔진 및 기술**: Unity(AR Foundation), C#, Google static map API, Google Fit API, Android Studio(Java)
* **팀 멤버**: 팀 "뿌꾸의 산책" (김민정, 정보연, 한태규)

---

## 📖 게임 영상
[![Game Demo](https://img.youtube.com/vi/fPzQFjNRLDo/0.jpg)](https://youtube.com/shorts/fPzQFjNRLDo)

---

## 🕹️ 프로젝트 구현

### Google Static Map API
### 🚀 워크플로우: 현재 위치 기반 지도 로딩 과정
```mermaid
sequenceDiagram
    participant UnityApp
    participant GPSLocationService
    participant ILocationProvider
    participant GoogleStaticMapService
    participant UnityWebRequestTexture
    participant UI

    UnityApp->>GPSLocationService: Init
    GPSLocationService->>ILocationProvider: 바인딩 및 초기화 대기
    ILocationProvider-->>GPSLocationService: mapOrigin 설정

    loop 위치 업데이트 주기
        ILocationProvider-->>GPSLocationService: 위치 업데이트 (lat, lon)
        GPSLocationService->>GoogleStaticMapService: onMapRedraw 이벤트
        GoogleStaticMapService->>UnityWebRequestTexture: 이미지 요청
        UnityWebRequestTexture-->>GoogleStaticMapService: Texture2D 수신
        GoogleStaticMapService-->>UI: onComplete 콜백 → 지도 표시
    end
```

---

### Google Fit API & Android Native Code(Java)
```mermaid
sequenceDiagram
    participant UnityCSharp
    participant AndroidJavaClass
    participant GoogleFitActivity
    participant GoogleFitJava
    participant UnityGameLogic

    UnityCSharp->>AndroidJavaClass: RequestGoogleFitOAuth()
    AndroidJavaClass->>GoogleFitActivity: Start Intent
    GoogleFitActivity->>GoogleFitJava: 권한 요청 & OAuth
    GoogleFitJava->>GoogleFitJava: subscribeSensor()
    GoogleFitJava-->>UnityGameLogic: UnitySendMessage(걸음 수)

    Note over UnityGameLogic: 걸음 수 UI 업데이트 등

    UnityCSharp->>AndroidJavaClass: LoadThumbnailToGallery()
    AndroidJavaClass->>gallery.java: getFirstImage()
    gallery.java-->>UnityCSharp: Base64 → Texture2D

    UnityCSharp->>AndroidJavaClass: OpenGallery()
    AndroidJavaClass->>gallery.java: openGallery()

    UnityCSharp->>AndroidJavaClass: SaveImageToGallery()
    AndroidJavaClass->>AndroidMediaScanner: 저장 요청
```


---

### AR Foundation

### 🚀 워크플로우: AR 콘텐츠 인식 및 배치 과정
```mermaid
sequenceDiagram
    participant UnityApp
    participant ARContentManager
    participant User
    participant ARRaycastManager
    participant GroundPrefab
    participant PetAgent

    UnityApp->>ARContentManager: Init & Enable
    ARContentManager->>ARRaycastManager: 활성화

    User->>ARRaycastManager: PlaceObjectAtCenter()
    ARRaycastManager-->>ARContentManager: Raycast hit

    alt 최초 배치
        ARContentManager->>GroundPrefab: 인스턴스화 & NavMesh Build
        ARContentManager->>PetAgent: 인스턴스화 & NavMeshAgent 추가
        PetAgent->>ARCamera: 시선 설정
    else 후속 배치
        ARContentManager->>GroundPrefab: 위치 이동 & NavMesh 재빌드
        ARContentManager->>PetAgent: SetDestination()
        PetAgent->>ARCamera: 목표 도달 → 시선 설정
    end

    UnityApp->>ARContentManager: OnDisable()
    ARContentManager->>AllObjects: 리소스 정리
```
