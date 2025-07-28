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
게임을 실행하면, 실행한 디바이스에 따라 프로바이더 결정한다.
- 에디터에서는, 현재 위치를 결정할 수 없으므로 직접 플레이어를 Input으로 움직이며 위치 갱신한다.
- 디바이스에서는, 현재 위도, 경도 값을 받아서 실제 위치를 전송한다.

매 업데이트 문마다, 현재 위치를 갱신 시켜서 Google Static Map API로 지도 값 요청

지도값을 텍스쳐로 받아서, 플레이어를 중심으로 9칸 이미지를 갱신한다.
 
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

### Google Static Map API
🚀 워크플로우: Google Fit 권한 요청 및 걸음 수 동기화, 갤러리 접근
Unity에서 Java 클래스를 호출하여 Google Fit 권한 요청 및 센서 등록을 진행합니다.
(개별 자바 클래스는 안드로이드 스튜디오 외부 라이브러리로 제작해서 aar 파일 형태로 빌드 :: 외부 플러그인)

걸음 수 데이터는 UnitySendMessage를 통해 유니티로 전달되어 게임 로직과 UI에 반영됩니다.

갤러리 접근 기능은 Java 측에서 이미지 조회/앱 실행/저장 기능을 제공하며 Unity와 연동됩니다.

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

### UI : MVC
Model (ObservableModel<T>)
값 변경 시 이벤트를 발생시키는 데이터 모델.
뷰에 직접 접근하지 않고 데이터 변경만 관리합니다.

View (UIView)
Unity MonoBehaviour를 상속한 UI 요소입니다.
모델의 변화에 따라 UI를 갱신하며, 사용자 입력을 컨트롤러로 전달합니다.

Controller (UIController<View, Model>)
모델과 뷰를 연결하며, 입력 처리 및 모델 → 뷰 바인딩을 담당합니다.

Factory (UIControllerFactory)
DI 없이 컨트롤러를 생성/캐싱/제거하는 팩토리 클래스입니다.

```mermaid
classDiagram
    class ObservableModel~T~ {
        - T _value
        + T Value
        + event PropertyChanged
    }

    class INotifyPropertyChanged~T~ {
        + event PropertyChanged
    }

    ObservableModel~T~ --|> INotifyPropertyChanged~T~

    class UIView {
        + Initialize(anchor)
        + CloseUI(isCloseAll)
    }

    class UIController~View, Model~ {
        - View view
        - Model model
        + BindInputEvents()
        + BindModelToView()
    }

    class UIControllerFactory {
        - controllerDict : Dictionary<UIView, object>
        + GetOrCreateController<TController, TView, TModel>()
        + RemoveController(view)
    }

    UIControllerFactory --> UIController
    UIController --> UIView
    UIController --> ObservableModel

```
