# AppleLog 앱 시스템 설계서 (System Design)

이 문서는 과수 농장 관리 모바일 앱인 **AppleLog**의 기술 스택, 아키텍처 패턴, 데이터 흐름 및 향후 확장성에 대해 기술한 기술 설계 문서입니다.

## 1. 기술 스택 (Tech Stack)

AppleLog는 외부 라이브러리 의존성을 최소화하고, 가볍고 빠른 모바일 렌더링을 위해 순수 웹 표준 기술을 사용하여 구축되었습니다.

*   **HTML5**: 앱의 뼈대와 구조 설정 (`index.html` 단일 파일)
*   **Tailwind CSS (CDN)**: 빠르고 일관된 반응형 UI 및 다크 모드, 애니메이션(트랜지션) 스타일링
*   **Vanilla JavaScript (ES6)**: 별도의 프레임워크(React, Vue) 없이 DOM 조작을 통한 SPA 라우팅 및 데이터 바인딩
*   **Google Material Symbols**: 통일감 있는 고품질 백터 아이콘 리소스 활용

## 2. 아키텍처 패턴 (Architecture)

본 앱은 **바닐라 자바스크립트 기반의 SPA(Single Page Application)** 패턴을 따릅니다. 여러 개의 HTML 파일을 넘나들지 않고, 하나의 `index.html` 내에서 뷰(View)를 전환합니다.

*   **View 캡슐화**: 각 화면은 `<section class="app-view">`로 모듈화되어 있으며, 평소에는 `hidden` 클래스를 통해 숨겨져 있습니다.
*   **라우팅(Routing) 로직**: `switchView(viewId)` 전역 함수를 호출하면, 현재 활성화된 화면을 숨기고 타겟 ID를 가진 `section`의 `hidden` 속성을 해제합니다. 
*   **상태 동기화**: 뷰 전환 시, 하단 네비게이션 탭의 액티브 상태(아이콘 색상/모양)와 상단 헤더의 모드(일반/다크모드)가 `switchView` 내부에서 동적으로 재계산되어 동기화됩니다.

## 3. 핵심 기능 동작 원리 (Key Workflows)

### A. 이미지 업로드 및 미리보기 (FileReader API)
새 일지 작성 화면에서 네이티브 `<input type="file" accept="image/*">`를 활용합니다.
스마트폰에서는 이 태그가 자동으로 **'카메라로 촬영'** 또는 **'앨범에서 선택'** 옵션을 제공합니다.
사용자가 이미지를 선택하면, JS의 `FileReader` API가 파일을 읽어들여 임시 **Base64 (Data URL)** 형태로 변환한 후, `<img>` 태그의 `src`에 즉시 할당하여 실시간 미리보기를 제공합니다.

### B. 데이터 생성 및 DOM 삽입 (Data Flow)
현재는 서버(DB)가 없는 프론트엔드 목업 상태로, **DOM 조작을 통한 인메모리(In-Memory) 저장 방식**을 사용합니다.
1.  **데이터 추출**: `saveLog()` 함수가 폼(Form)에서 입력받은 텍스트(카테고리, 내용)와 이미지(Base64) 값을 읽어옵니다.
2.  **HTML 템플릿 렌더링**: 자바스크립트 템플릿 리터럴(Template Literal)을 사용하여 새로운 일지 카드 HTML 문자열을 조립합니다.
3.  **DOM 삽입**: `insertAdjacentHTML('afterbegin', ...)` 메서드를 사용해 전체 일지 목록(`april-logs-container`)의 최상단(DOM 트리의 앞부분)에 카드를 끼워 넣습니다.
4.  **UI 초기화**: 저장이 완료되면 폼 입력값을 리셋하고 성공(Success) 뷰로 전환합니다.

## 4. 모바일 및 UI/UX 설계 지향점

*   **Mobile-First**: 모바일 디바이스(스마트폰) 세로 화면에 최적화된 레이아웃(하단 탭 바, 큰 버튼, 카드형 디자인)을 갖춥니다.
*   **Micro-Interactions**: 모든 버튼과 카드는 `:active` 또는 `group-hover`를 통해 누를 때 미세하게 작아지거나(`scale-95`) 그림자가 강조되는 반응성을 가집니다.
*   **다크 모드 활용**: 'AI 진단 스캔' 모드 진입 시 몰입감을 높이기 위해 상단 헤더가 애니메이션과 함께 검은색 배경으로 전환됩니다.

## 5. 향후 확장성 고려 (Future Scalability)

현재의 순수 프론트엔드 상태에서 실제 서비스로 발전하기 위한 설계 로드맵입니다.

1.  **상태 관리(State Management) 도입**: 앱이 커지면 `switchView` 방식의 전역 변수 관리가 복잡해질 수 있습니다. 향후 React/Vue 같은 프레임워크나 바닐라 JS 기반의 옵저버(Observer) 패턴 도입을 고려합니다.
2.  **BackendaaS (BaaS) 연동**: **Supabase** 또는 **Firebase**를 연동하여, `saveLog()` 시 DOM에 바로 그리는 대신 서버의 PostgreSQL(DB)에 JSON 형태로 INSERT하고, 전체 일지 화면 로드 시 SELECT 해오는 API 기반 아키텍처로 변경해야 합니다.
3.  **오브젝트 스토리지 연동**: 용량이 큰 Base64 이미지를 DB에 직접 넣지 않고, 클라우드 스토리지(S3 등)에 업로드 후 해당 이미지 URL 링크만 DB 테이블에 저장하도록 데이터 스키마를 분리해야 합니다.
4.  **PWA 전환**: `manifest.json`과 `Service Worker`를 추가해 웹을 넘어 실제 네이티브 앱과 동일한 오프라인 작동 및 홈 화면 설치 경험을 제공할 수 있습니다.
