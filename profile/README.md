# Capstone Design2 (Mobile Programming)
## AI 공지 검색 챗봇 프로그램
**🌟간트 차트:** https://docs.google.com/spreadsheets/d/1E2I-yOQkWpHmZ9G4DxZI66Tl0gs0sVFjFpinn9Yl7dI/edit?usp=sharing<br>
**🌟노션:** https://www.notion.so/3207e96bc6788011b4fac9784e019ebc?v=3207e96bc6788091b7e5000cf1314ddc&source=copy_link<br><br><br>

<br>UI 설계<br>
<img width="665" height="532" alt="image" src="https://github.com/user-attachments/assets/e602cf50-33f2-456a-83e0-2893c909c701" /><br>
동작 화면<br>
<img width="519" height="859" alt="image" src="https://github.com/user-attachments/assets/ccbccf8c-c37e-4ce4-adb9-a3338acdf1df" /><br><br><br>

## 🌟3TEAM 소개🌟
#### 정세영- PM

- notion 기반 팀프로젝트 진행
- 요구사항 정의서를 기반으로 개발될 수 있도록 함
- 간트 차트, 주간 업무 보고서로 일정 관리
- github로 코드 취합, 단위/통합 테스트 진행

#### 이정우- 프론트엔드(화면 설계, 구현)

- 개발 환경: Android Studio (Android Native 환경)
- 개발 언어: Java, XML
- 네트워크 통신: Retrofit2 (서버 데이터 송수신용)
- 이미지/애니메이션: Glide (이미지 처리), Lottie (로딩 애니메이션)
- UI 컴포넌트: Material Design, ConstraintLayout

#### 김수현- 백엔드(크롤링, DB)

#### 박성현- 백엔드(DB 키워드 추출, gpt api 연결)

- 크롤러: Jsoup 기반
- Spring Boot + JPA
- 데이터베이스: MySQL
- AI Model : ChatGPT


<br><br><br><br>
## 🌟기능 설명🌟

### 🔥프론트엔드
- **앱 실행 및 검색**: 앱 실행 시 메인 화면 중앙에 직관적인 검색창을 배치하여 사
용자가 구어체 질문을 입력할 수 있도록 구성
- **로딩 및 상태 표시**: AI 분석 시간 동안 Lottie 애니메이션 등을 활용해 시각적 피
드백 제공
- **결과 출력**
    - 상단 (AI 요약): AI가 분석해서 보내준 답변 내용을 가독성 좋게 배치
    - 하단 (출처 원문): 답변의 출처인 실제 학교 공지사항을 나열
- **상세 원문 확인**: 하단의 공지를 클릭하면 앱 내장 WebView를 통해 학교 홈페이
지의 해당 공지 글로 즉시 연결
- **예외 상황 처리**: 검색 결과가 없거나 네트워크 오류 시 안내 문구와 일러스트 노출

**<작동 원리>**
1. **질문 전송**: 사용자가 전송 버튼 클릭 시 Retrofit2를 통해 검색어 데이터를 서버로
전송
2. **비동기 처리**: 서버 분석 대기 시간 동안 UI 스레드가 멈추지 않도록 처리
3. **데이터 바인딩**: 서버 응답 성공 시, 상단 텍스트뷰에는 요약본을, 하단
NoticeAdapter에는 데이터 배열을 전달하여 화면을 동적으로 갱신


### 🔥백엔드<br>
#### !학사, 장학, 행사만 각 100개씩 뽑음, 매일 아침 업데이트!<br>
- 크롤링 후 정제된 데이터를 DB에 저장<br>
<br>
사용자 질문 입력 → 앱에서 서버로 요청 → 키워드 추출 → DB 검색 → GPT 전달 → 답변 생성 → 결과 반환<br>
<br>
- **공지사항 수집**
    - 공지 URL 기반 수집 및 content 추출 후 DB 저장
    - 해시 비교로 수정 여부 감지
- **사용자 질문 처리**
    - 자연어 질문 → 키워드 정규화 → DB 검색 → GPT 전달 → 답변 생성
- **예외 처리**
    - 공지 없음 / GPT 실패 네트워크 오류 대응<br>
<br><br>
**크롤링 파트**<br>
SyuNoticeCrawler.java<br>
"학교 공지사항 목록 페이지에 접속해서 공지 URL들을 수집하는 파일입니다. 학사/장학/행사 카테고리별로 페이지를 넘기면서 URL만 뽑아옵니다. Jsoup으로 HTML을 읽고 불필요한 링크나 외부 링크는 필터링해서 실제 공지 URL만 남깁니다."<br>
SyuNoticeParser.java<br>
"크롤러가 수집한 URL에 하나씩 접속해서 제목, 날짜, 본문을 추출하는 파일입니다. 크롤링이 URL 수집이라면 파싱은 실제 내용 추출입니다. HTML에서 필요한 데이터만 골라내고 불필요한 태그나 공백을 제거해서 정제합니다."<br>
CrawledNotice.java<br>
"크롤링하고 파싱한 결과를 담는 그릇 역할입니다. 제목, 날짜, 본문, URL, 카테고리를 하나의 객체로 묶어서 DB 저장 전까지 들고 다닙니다."<br>
NoticeCrawlerService.java<br>
"크롤러, 파서, DB저장을 하나로 묶어서 자동 실행시키는 파일입니다. @Scheduled로 매일 오전 8시에 전체 과정을 자동으로 돌립니다. 학사→장학→행사 순서로 카테고리별 크롤링을 순차 실행하고 예외 발생해도 다음 공지로 넘어가도록 처리했습니다."<br>
<br><br>
**DB 파트**<br>
JdbcNoticeRepository.java<br>
"파싱된 공지 데이터를 MySQL DB에 저장하는 파일입니다. ON DUPLICATE KEY UPDATE를 써서 같은 URL의 공지가 다시 들어오면 중복 저장 없이 덮어쓰기만 합니다. JPA 대신 JDBC를 직접 쓴 이유는 ON DUPLICATE KEY UPDATE가 MySQL 전용 문법이라 JPA로는 처리가 어렵기 때문입니다."<br>
NoticeRepository.java<br>
"저장된 공지를 꺼내오는 파일입니다. JPA 인터페이스로 메서드 이름만 정의하면 쿼리가 자동 생성됩니다. 전체 목록, 카테고리별, 키워드 검색 세 가지 방식으로 조회할 수 있습니다."<br>
Notice.java<br>
"DB notices 테이블과 1대1로 매핑되는 파일입니다. id, title, date, url, body, category, fetchedAt 컬럼이 여기서 정의됩니다."<br>
application.properties<br>
"DB 주소, 계정, 비밀번호 등 연결 설정이 들어있는 파일입니다."<br>
<br><br>
**flow**<br>
SyuNoticeCrawler (URL 수집)<br>
↓<br>
SyuNoticeParser (내용 추출)<br>
↓<br>
CrawledNotice (데이터 담기)<br>
↓<br>
JdbcNoticeRepository (DB 저장)<br>
↓<br>
NoticeRepository (DB 조회)
