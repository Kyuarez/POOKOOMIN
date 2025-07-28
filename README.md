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
[Unity App Start]
  -> [GPSLocationService] Init
     -> [ILocationProvider] (Device/Simulated) 바인딩 & 준비 대기
        -> 현재 위치 (mapOrigin) 설정

[ILocationProvider] 위치 업데이트 (주기적)
  -> [GPSLocationService] (onLocationUpdated)
     -> 현재 위치 (latitude, longitude) 업데이트 (mapCenter)
     -> [onMapRedraw] 이벤트 발생 (지도 재그리기 요청)

[onMapRedraw] 이벤트 트리거
  -> [GoogleStaticMapService] LoadMap(현재 위치, 줌, 크기) 호출
     -> Google Static Map API URL 생성
     -> [UnityWebRequestTexture] 이미지 비동기 다운로드
     -> [Texture2D] 획득 &rarr; onComplete 콜백 호출

[UI/Map Display Component]
  -> [Texture2D] 수신
     -> RawImage/Material에 할당 &rarr; 화면에 지도 표시



### Google Fit API & Android Native Code(Java)

[Android Studio] .aar 플러그인 개발

1.  **갤러리 (`gallery.java`)**
    * `getFirstImage()`: 갤러리 쿼리 &rarr; 최신 이미지 `Bitmap` &rarr; **Base64** 인코딩 &rarr; Unity 반환
    * `openGallery()`: `Intent.ACTION_VIEW` &rarr; **갤러리 앱 실행**

2.  **Google Fit (`googleFit.java`, `googleFitPermissionActivity.java`)**
    * `googleFitPermissionActivity` Start: `ACTIVITY_RECOGNITION` 권한 요청 &rarr; Google Fit OAuth 진행
    * OAuth 성공 시: `googleFit.subscribeSensor()` 호출
    * `subscribeSensor()`: `FitnessOptions` (걸음 수 읽기) 정의 &rarr; `SensorsClient.add()` &rarr; **누적 걸음 수 센서 구독**
    * 센서 데이터 수신 (주기적) &rarr; `UnityPlayer.UnitySendMessage` (Reflection) &rarr; Unity로 걸음 수 데이터 전송

[Unity Editor]
  -> .aar 파일 `Assets/Plugins/Android`에 추가 & Manifest/Gradle 설정

[Unity C# 코드]
1.  **Google Fit 요청 (`GoogleFitUtil.cs`)**
    * `RequestGoogleFitOAuth()` &rarr; `AndroidJavaClass` (`UnityPlayer`, `currentActivity`)
    * 새 `Intent` &rarr; `googleFitPermissionActivity` 시작 (권한/로그인 플로우 시작)

2.  **갤러리 접근 (`PictureUtil.cs`)**
    * `LoadThumbnailToGallery()` &rarr; `AndroidJavaClass` (`gallery`) &rarr; `CallStatic("getFirstImage")` &rarr; Base64 수신 &rarr; `Texture2D` 변환
    * `OpenGallery()` &rarr; `gallery.CallStatic("openGallery")` &rarr; 갤러리 앱 실행
    * `SaveImageToGallery()` &rarr; Android 미디어 스캐너 `Intent` &rarr; 갤러리 저장

[Unity Game Logic]
  -> Unity C# 스크립트 &larr; `UnitySendMessage`를 통해 Google Fit 걸음 수 데이터 수신 (e.g., UI 업데이트, 보상)
  -> 갤러리 이미지 (`Texture2D`) &rarr; 게임 내 UI에 활용 (e.g., 프로필 사진)

---

### AR Foundation

### 🚀 워크플로우: AR 콘텐츠 인식 및 배치 과정
[Unity App Start/AR Mode Activate]
  -> [ARContentManager] Init & Enable
     -> [ARPlaneManager], [ARRaycastManager] 활성화
     -> 펫 프리팹 로드

[User Interaction] PlaceObjectAtCenter() 호출
  -> 화면 중앙 &rarr; [ARRaycastManager] &rarr; AR 평면 레이캐스트

[레이캐스트 성공]
  -> **최초 배치 시:**
     -> `groundPrefab` 인스턴스화 &rarr; [NavMeshSurface] **NavMesh 빌드** (다음 프레임)
     -> `agentPrefab` (펫) 인스턴스화 &rarr; 펫에 [NavMeshAgent] 추가
     -> 펫 초기 시선 AR 카메라로 설정
  -> **후속 배치 시:**
     -> `_instanceGround` 새 위치로 이동 &rarr; NavMesh 재빌드
     -> [NavMeshAgent] `SetDestination()` &rarr; 펫 이동 명령
     -> 펫 목표 도달 시 &rarr; 펫 시선 AR 카메라로 조정

[ARContentManager] OnDisable()
  -> 펫, 바닥 파괴 & 모든 AR 평면 비활성화 (리소스 정리)
