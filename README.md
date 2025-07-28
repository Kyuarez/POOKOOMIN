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

**1. 시스템 초기화 및 GPS 서비스 준비**
    * **`[GPSLocationService]`** 컴포넌트가 초기화됩니다.
    * **환경 분기:**
        * `Unity Editor` 환경에서는 **`[SimulatedLocationProvider]`** (시뮬레이션용 GPS)를 동적으로 생성하여 사용합니다.
        * `실제 빌드` 환경에서는 **`[DeviceLocationProvider]`** (장치 GPS)를 동적으로 생성하여 사용합니다.
    * **목적:** 개발 및 테스트 효율성, 그리고 실제 장치 환경에서의 GPS 정보 획득을 유연하게 처리하기 위함입니다.
    * **`[GPSLocationService]`**는 `ILocationProvider` 인터페이스를 통해 실제 GPS 제공자와 바인딩됩니다.
    * **`[GPSLocationService]`**는 `ILocationProvider`가 GPS 서비스 실행 준비(권한 획득, 초기화 완료 등)가 완료될 때까지 대기합니다.
    * 준비가 완료되면 현재 위치를 초기 지도 원점(`mapOrigin`)으로 설정하고, 서비스 준비 완료 상태를 외부에 알립니다.

**2. 실시간 위치 정보 획득 및 전파**
    * **`[GPSLocationService]`**가 활성화되면 **`[ILocationProvider]`** (예: `DeviceLocationProvider`)의 `StartService()` 메서드를 호출합니다.
    * **`[DeviceLocationProvider]`** (또는 시뮬레이션 Provider):
        * 모바일 기기에서 **GPS 접근 권한을 확인하고 필요시 사용자에게 요청**합니다.
        * GPS 장치가 활성화되어 있는지 확인하고, Unity의 `Input.location` API를 사용하여 GPS 서비스 시작 및 설정(정확도, 갱신 주기)을 관리합니다.
        * GPS 서비스가 초기화될 때까지 대기합니다 (타임아웃 처리 포함).
        * 서비스가 성공적으로 시작되면, 설정된 주기(예: 1초)마다 최신 위치 데이터(`latitude`, `longitude`, `altitude`, `accuracy`, `timestamp`)를 지속적으로 획득합니다.
        * 새로운 위치 데이터가 획득될 때마다 **`onLocationUpdated` 이벤트**를 발생시킵니다.
            * **목적:** **옵저버 패턴(Observer Pattern)**을 활용하여 위치 정보 제공자와 이를 사용하는 `GPSLocationService` 간의 **느슨한 결합(Loose Coupling)**을 유지하며 데이터를 비동기적으로 전달합니다.

**3. 지도 중심 업데이트 및 재그리기 요청**
    * **`[GPSLocationService]`**는 `ILocationProvider`의 `onLocationUpdated` 이벤트를 구독하여 갱신된 위치 데이터를 수신합니다.
    * 수신된 최신 위치 정보로 `GPSLocationService` 내부의 현재 위치 관련 속성들을 업데이트합니다.
    * `CenterMap()` 메서드를 호출하여 지도의 중심 좌표(`mapCenter`)를 갱신된 현재 위치로 설정합니다.
    * **`onMapRedraw` 이벤트**를 발생시킵니다.
        * **목적:** 이 이벤트는 **지도를 다시 그려야 할 필요성**을 다른 지도 렌더링 담당 모듈(예: `GoogleStaticMapService`)에 알립니다.

**4. Google Static Map 이미지 로드**
    * **`[GoogleStaticMapService]`** (또는 지도를 표시할 다른 컴포넌트)는 **`GPSLocationService`의 `onMapRedraw` 이벤트를 구독**하고 있습니다.
    * 이벤트 발생 시 **`[GoogleStaticMapService]`**의 `LoadMap()` 메서드를 호출합니다.
    * **`[GoogleStaticMapService]`**는 현재 지도의 중심 좌표(`latitude`, `longitude`), 설정된 줌 레벨(`ZoomLevel`), 지도 크기(`mapTileSizePixels` 등)를 기반으로 **Google Static Map API 요청 URL을 동적으로 생성**합니다.
    * `UnityWebRequestTexture.GetTexture()`를 사용하여 생성된 URL로 웹 요청을 보내 **지도 이미지(`Texture2D` 형태)를 비동기적으로 다운로드**합니다.
    * 이미지 다운로드가 완료되면, 미리 정의된 콜백(`onComplete`)을 통해 로드된 `Texture2D` 이미지를 실제 화면에 지도를 렌더링할 UI 컴포넌트에 전달합니다.

**5. 지도 화면 표시**
    * `GoogleStaticMapService`로부터 전달받은 `Texture2D` 이미지는 Unity UI의 `RawImage` 컴포넌트나 게임 오브젝트


### Google Fit API & Android Native Code(Java)

### 🚀 워크플로우: Unity - Android Native Plugin 연동

**1. Android Native Plugin 개발 (`.aar` 파일 생성)**

* **Android Studio 환경:**
    * `com.example.usergallery` 패키지 (`gallery.java`):
        * **가장 최근 이미지 가져오기:** 기기 갤러리에서 최근 추가된 이미지를 쿼리하여 해당 이미지 파일을 `Bitmap`으로 로드합니다. 이 `Bitmap`은 Unity에서 사용하기 쉽도록 **Base64 문자열로 인코딩하여 반환**됩니다.
        * **갤러리 앱 열기:** `Intent.ACTION_VIEW`를 사용하여 기기의 기본 갤러리 애플리케이션을 실행합니다.
    * `com.example.usergooglefit` 패키지 (`googleFit.java`, `googleFitPermissionActivity.java`):
        * **Google Fit 권한 요청(`googleFitPermissionActivity.java`):**
            * Android 10(API 29) 이상에서 필요한 **`ACTIVITY_RECOGNITION` 권한을 요청**합니다.
            * Google Fit API 접근을 위한 **Google 계정 로그인 및 OAuth 동의 절차**를 진행합니다 (`GoogleSignIn.requestPermissions`).
            * 이 모든 권한 요청은 사용자와의 상호작용이 필요한 `Activity`를 통해 처리됩니다.
        * **Google Fit 센서 구독(`googleFit.java`):**
            * 권한 획득 성공 시, `FitnessOptions.builder().addDataType(DataType.TYPE_STEP_COUNT_CUMULATIVE)`를 통해 **누적 걸음 수 데이터에 대한 읽기 권한을 정의**합니다.
            * `SensorsClient.add()`를 사용하여 정의된 `DataType`(누적 걸음 수)에 대한 **센서 구독을 시작**합니다.
            * 설정된 샘플링 주기(예: 5초)마다 콜백(`dataPoint -> {...}`)을 통해 센서 데이터를 수신합니다.
        * **Unity로 데이터 전송:** 수신된 걸음 수 데이터(`stepCount`)는 Java Reflection(`UnityPlayer.UnitySendMessage`)을 사용하여 Unity 엔진의 특정 GameObject(예: "GoogleFitService")의 특정 메서드(예: "onStepCountChanged")로 문자열 형태로 전달됩니다. 이를 통해 **Android 네이티브에서 Unity로의 실시간 데이터 스트리밍**을 구현합니다.

* **빌드:** 위 Java 코드들을 Android Studio에서 컴파일하여 `.aar` 파일(플러그인 라이브러리)로 패키징합니다.

**2. Unity 프로젝트에 플러그인 통합**

* **`Assets/Plugins/Android` 폴더:** 생성된 `.aar` 파일을 Unity 프로젝트의 `Assets/Plugins/Android` 경로에 배치합니다. Unity 빌드 시스템은 이 경로의 `.aar` 파일을 자동으로 Android 빌드에 포함시킵니다.
* **Android 빌드 설정:** Unity 프로젝트의 Android 빌드 설정에서 필요한 권한(`AndroidManifest.xml`에 `ACTIVITY_RECOGNITION`, `QUERY_ALL_PACKAGES` 등)을 명시하고, Google Fit SDK 종속성을 Gradle 파일에 추가합니다.

**3. Unity에서 Android 플러그인 기능 호출**

* **`GoogleFitUtil.cs` (C#):**
    * `RequestGoogleFitOAuth()` 메서드는 **`AndroidJavaClass` 및 `AndroidJavaObject`를 활용**하여 Unity의 현재 Android `Activity`를 가져옵니다.
    * 새로운 Android `Intent`를 생성하고, 이를 통해 `googleFitPermissionActivity`를 시작하여 **Google Fit 권한 요청 플로우를 트리거**합니다.
    * `#if UNITY_ANDROID` 컴파일 지시자를 사용하여 Android 빌드 환경에서만 네이티브 코드가 포함되도록 플랫폼 의존성을 관리합니다.
* **`PictureUtil.cs` (C#):**
    * `LoadThumbnailToGallery()` 메서드는 `AndroidJavaClass`를 사용하여 Android 플러그인의 `com.example.usergallery.gallery` 클래스를 참조하고, `CallStatic("getFirstImage", currentActivity)`를 통해 **가장 최근 이미지의 Base64 문자열을 요청**합니다.
    * `UIUtil.Base64ToTexture2D()` (외부 유틸리티로 추정)를 사용하여 Base64 문자열을 Unity의 `Texture2D` 객체로 변환하여 게임 내에서 이미지로 사용할 수 있게 합니다.
    * `OpenGallery()` 메서드는 `galleryClass.CallStatic("openGallery", currentActivity)`를 호출하여 **Android 갤러리 앱을 직접 실행**합니다.
    * `SaveImageToGallery()` 메서드는 저장할 이미지 파일 경로를 받아 Android의 미디어 스캐너 `Intent`를 브로드캐스트하여 갤러리에 이미지가 추가되도록 합니다.

**4. Unity로 데이터 수신 및 게임 로직 연동**

* Google Fit 플러그인에서 `UnitySendMessage`를 통해 전달된 걸음 수 데이터는 Unity 스크립트(예: "GoogleFitService" GameObject의 "onStepCountChanged" 메서드)에서 수신됩니다.
* Unity는 수신된 걸음 수 데이터를 파싱하여 게임 내 UI 업데이트, 보상 지급 등 **구글 핏 데이터를 활용한 게임 로직을 실행**합니다.
* 갤러리에서 로드된 `Texture2D`는 게임 내 프로필 이미지, 아이템 썸네일 등 다양한 UI 요소에 동적으로 적용될 수 있습니다.

---

### **🌟 기술적 고려사항 및 강점**

* **Unity-Android JNI Bridge:** `AndroidJavaClass`와 `AndroidJavaObject`를 활용하여 Unity C# 코드와 Android Java/Kotlin 네이티브 코드 간의 효율적인 상호작용(JNI Bridge)을 구현했습니다.
* **플러그인 모듈화:** Google Fit 및 갤러리 기능을 독립적인 `.aar` 플러그인으로 분리하여 **재사용성 및 유지보수성**을 높였습니다.
* **권한 관리:** Android 런타임 권한(`ACTIVITY_RECOGNITION`) 및 Google Fit API의 OAuth 흐름을 체계적으로 처리하여 **사용자 동의 기반의 안전한 데이터 접근**을 보장합니다.
* **실시간 데이터 처리:** Google Fit 센서 데이터의 주기적인 구독과 `UnitySendMessage`를 통한 실시간 전송으로 **라이브 데이터 연동** 기능을 제공합니다.
* **에디터 시뮬레이션 지원:** 개발 편의성을 위해 비 Android 환경에서의 로직 처리를 위한 `#if UNITY_ANDROID`와 같은 **플랫폼 의존성 분기 처리**를 적용했습니다.

---

### AR Foundation

### 🚀 워크플로우: AR 콘텐츠 인식 및 배치 과정

본 모듈은 크게 **AR 평면 기반 콘텐츠 배치**와 **AR 이미지 트래킹 기반 콘텐츠 배치**라는 두 가지 주요 흐름으로 구성됩니다.

#### **1. AR 평면 기반 콘텐츠 배치 (`ARContentManager`)**

이 컴포넌트는 사용자의 터치(또는 화면 중앙 레이캐스트)를 통해 인식된 AR 평면에 가상의 펫과 움직일 수 있는 바닥을 배치하고 관리하는 역할을 합니다.

* **초기화 및 준비 (`Awake`, `OnEnable`, `OnDisable`)**
    * `ARPlaneManager`와 `ARRaycastManager` 컴포넌트를 초기화하여 AR 평면 감지 및 레이캐스트 기능을 활성화합니다.
    * 펫 프리팹(`LittleSquirrel`)을 로드하고 스케일을 조정합니다.
    * 모듈이 비활성화될 때(종료 시) 생성된 펫과 바닥 오브젝트를 파괴하고, 감지된 모든 AR 평면을 비활성화하여 리소스를 정리합니다.

* **AR 평면 감지 및 레이캐스트 (`PlaceObjectAtCenter`)**
    * 사용자가 화면 중앙을 기준으로 **`PlaceObjectAtCenter()`** 메서드를 호출하면, `ARRaycastManager`를 사용하여 현재 AR 카메라가 바라보는 화면 중앙에서 실제 AR 평면(`TrackableType.PlaneWithinPolygon`)으로 레이캐스트를 시도합니다.
    * **목적:** 사용자가 직관적으로 원하는 위치에 AR 콘텐츠를 배치할 수 있도록 돕습니다.

* **AR 콘텐츠 초기 배치 (최초 레이캐스트 성공 시)**
    * 레이캐스트에 성공하고 이것이 **최초 배치 시도**라면:
        * **`groundPrefab` (바닥)**을 레이캐스트 충돌 지점의 위치와 회전에 맞춰 인스턴스화합니다.
        * **`NavMeshSurface` 빌드:** `_instanceGround`에 부착된 `NavMeshSurface` 컴포넌트를 통해 **다음 프레임에 NavMesh를 빌드**합니다. 이는 Mesh가 완전히 초기화된 후 NavMesh를 생성하여 안정성을 확보하기 위한 **코루틴(`BuildNavMeshNextFrame`)**을 활용합니다.
        * **`agentPrefab` (펫)**을 동일한 위치와 회전에 맞춰 인스턴스화합니다.
        * 펫의 `PetController`를 초기화하고, AR 카메라를 바라보도록 초기 회전 값을 조정합니다.
        * 펫에 **`NavMeshAgent` 컴포넌트를 추가**하여 이후 자율 이동이 가능하도록 설정합니다.
    * **목적:** AR 공간에 가상 오브젝트가 현실감 있게 배치될 수 있는 환경을 조성하고, 펫의 이동을 위한 NavMesh 기반을 마련합니다.

* **AR 콘텐츠 이동 및 펫 내비게이션 (후속 레이캐스트 성공 시)**
    * 최초 배치가 완료된 후 **다시 레이캐스트에 성공**하면:
        * **`_instanceGround` (바닥)** 오브젝트의 위치를 새로운 레이캐스트 충돌 지점으로 이동시킵니다.
        * 이동된 바닥에 대해 **NavMesh를 다시 빌드**합니다 (`BuildNavMeshNextFrame` 코루틴 재활용).
        * **`NavMeshAgent`를 사용하여 펫(`_instancePet`)을 새로운 충돌 지점으로 이동**시키도록 명령합니다 (`agent.SetDestination(hitPose.position)`).
        * 펫이 목표 지점에 도달하면(경로가 없거나 속도가 0에 가까워지면), 펫의 시선을 다시 AR 카메라 방향으로 향하도록 조정합니다.
    * **목적:** 사용자가 지정한 AR 평면 위에서 펫이 자연스럽게 이동하고, 바닥이 펫의 이동 경로에 맞춰 업데이트되도록 합니다.

#### **2. AR 이미지 트래킹 기반 콘텐츠 배치 (`TrackedImageHandler`)**

이 컴포넌트는 미리 정의된 참조 이미지(Reference Image Library에 등록된 이미지)를 실제 카메라에서 인식하고 추적하여, 해당 이미지 위에 특정 AR 콘텐츠를 배치하는 역할을 합니다.

* **초기화 (`Start`)**
    * `ARTrackedImageManager` 컴포넌트를 참조하고, `trackablesChanged` 이벤트에 `OnTrackablesChanged` 메서드를 리스너로 등록합니다.
    * **목적:** AR 시스템이 새로운 이미지 인식, 기존 이미지 업데이트, 또는 이미지 상실을 감지할 때마다 즉시 알림을 받기 위함입니다.

* **추적된 이미지 변경 이벤트 처리 (`OnTrackablesChanged`)**
    * `ARTrackedImageManager`로부터 `args` (추적된 이미지 변경 이벤트 인자)를 전달받습니다. `args`는 새로 추가된(`added`), 업데이트된(`updated`), 제거된(`removed`) 이미지 목록을 포함합니다.
    * **`args.added` (새로운 이미지 인식):**
        * 새롭게 인식된 `ARTrackedImage` 객체에 대해 `_placePrefab`을 인스턴스화합니다.
        * 생성된 프리팹의 위치와 회전을 인식된 이미지의 `transform.position` 및 `transform.rotation`에 일치시킵니다.
        * `_placedMarkers` 딕셔너리에 인식된 이미지의 `trackableId`를 키로, 생성된 프리팹을 값으로 저장하여 관리합니다.
    * **`args.updated` (기존 이미지 위치/상태 업데이트):**
        * 이미 추적 중인 이미지의 위치나 회전이 변경될 경우, `_placedMarkers` 딕셔너리에서 해당 이미지에 연결된 프리팹을 찾아 업데이트된 이미지의 `transform.position` 및 `transform.rotation`에 맞춰 위치와 회전을 갱신합니다.
    * **`args.removed` (이미지 추적 상실):**
        * 더 이상 추적되지 않는 이미지에 대해서는 해당 이미지와 연결된 프리팹을 파괴하는 로직을 구현할 수 있습니다 (현재 코드에서는 주석 처리되어 있지만, 실제 구현에서는 제거 로직이 필요).
    * **목적:** 실제 세계의 이미지 마커 위에 AR 콘텐츠를 정확하고 동적으로 배치하며, 마커의 움직임에 따라 콘텐츠가 함께 움직이도록 동기화합니다.

---

### **🌟 기술적 강점**

* **Unity AR Foundation 활용:** 최신 AR 개발 표준인 AR Foundation을 사용하여 플랫폼(iOS/Android)에 구애받지 않는 통합된 AR 경험을 제공합니다.
* **AR 평면/이미지 동시 인식:** 두 가지 주요 AR 추적 방식(평면 감지 및 이미지 트래킹)을 모두 구현하여 다양한 AR 상호작용 시나리오에 대응할 수 있습니다.
* **NavMesh 기반 펫 이동:** AR 공간 내에서 가상 펫이 현실적인 경로를 따라 자율적으로 이동할 수 있도록 Unity의 NavMesh 시스템을 효과적으로 활용했습니다.
* **모듈화 및 이벤트 기반 디자인:** 각 AR 기능(평면 관리, 레이캐스트, 이미지 트래킹)이 독립적인 컴포넌트로 분리되어 있으며, 이벤트(`trackablesChanged`)를 통해 상호작용하여 **모듈 간의 결합도를 낮추고 유지보수성**을 높였습니다.
* **강화된 사용자 상호작용:** AR 환경에서 오브젝트 배치 및 펫 내비게이션을 통해 사용자와 가상 콘텐츠 간의 몰입감 있는 상호작용을 제공합니다.
