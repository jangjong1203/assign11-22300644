# Week 11 — Attachment Board (Student Sample)

Spring Boot + Thymeleaf + JPA + MariaDB: 게시글 CRUD / 다중 첨부파일 업로드 / 다운로드

## 실행하기
1. Java 17+ 및 MariaDB 준비, `CREATE DATABASE web11 CHARACTER SET utf8mb4;`
2. 환경변수 `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`를 자신의 MariaDB에 맞게 설정. 기본값은 `jdbc:mariadb://localhost:3306/web11`, `webuser`, `webpass`.
3. Gradle이 설치된 환경: `gradle bootRun` / IntelliJ: `AttachmentBoardApplication.main()` 실행.
4. 접속: http://localhost:8080/posts


① Source Code Analysis

Sample Project의 소스코드를 읽고 다음 8개 질문에 대한 답을 README.md에 자신의 말로 작성합니다.

Project Structure & Spring MVC

    PostController, PostService, PostRepository는 각각 어떤 역할을 하며, 서로 어떻게 연결되는가?
    controller는 http의 요청을 받고 응답을 보냅니다. service는 트랜잭션단위로 비즈니스 로직을 수행하고 파일을 저장을 관리합니다.
    repository는 JPA를 통해서 데이터베이스의 CRUD 작업을 실행합니다.
    controller->service->repository 순서로 연결되어있습니다.

    게시글 작성 시 HTML Form의 입력값과 첨부파일은 Controller에 어떻게 전달되는가?
    form의 입력값은 enctype 이 설정되어있어 제출 버튼을 누르면 텍스트 입력값들과 첨부파일들이 HTTP POST 요청에 담겨 /posts로 보내집니다.
    controller에서 텍스트는 @Valid,@ModelAttribute를 통해 postForm객체에 맞는 형식인지 검사합니다.
    파일은 @RequestParam을 통해 List<MultipartFile>에 저장됩니다.

JPA & Entity Relationship
    
    Post와 Attachment Entity는 어떤 관계이며, @ManyToOne과 @OneToMany는 각각 어떤 역할을 하는가?
    Post의 게시글 한개에 Attachment의 첨부파일이 여러개가 들어있는 다대일 관계입니다.    
    @ManyToOne는 Attachment에서 여러 첨부파일들이 post에 있는 특정 게시글에 포함되어있다고 알려주는 역할을 합니다.
    @OneToMany는 post에서 게시글을 호출하면 해당 게시글에 속해있는 첨부파일들을 불러오는 역할을 합니다.

    cascade = CascadeType.ALL과 orphanRemoval = true는 어떤 역할을 하며, 실제 디스크에 저장된 파일에도 영향을 주는가?
    cascade = CascadeType.ALL는 게시글을 저장하거나 삭제할때 자동적으로 안에 들어있는 첨부파일들도 저장하거나 제거해주는 역할을 합니다.
    orphanRemoval = true는 게시글에서 특정 첨부파일만 제거 하고싶을때 연결만 끊어줘도 자동적으로 삭제까지 시켜주는 역할을 합니다.
    실제 파일에는 영향을 주지않고 DB에서만 관리가 이루어집니다.

File Upload

    MultipartFile은 무엇이며, 업로드된 파일은 어떤 과정을 거쳐 서버에 저장되는가?
    MultipartFile는 업로드된 파일들을 관리하는 도구로 실제로 저장된 파일이 아니라 http로 전달받은 받은 파일 입니다.
    업로드된 파일은 컨트롤러 create에서 서비스로 넘어가고 서비스 store에서 적절한 파일인지 검사후 서버 폴더에 파일을 복사해서 저장합니다.
    이후 파일정보들은 Attachment 객체를 만들어 서비스에서 데이터베이스에 저장합니다.
    
    파일을 원본 이름 대신 UUID 기반 파일명으로 저장하는 이유는 무엇인가?
    똑같은 이름을 가진 파일이 올라왔을때 생기는 오류나 기타 오류(덮어쓰기나 이상한 이름으로 해킹 등)를 막기위해 UUID를사용합니다.

File Download & Security
    
    파일 다운로드 시 Resource, ResponseEntity, Content-Disposition은 각각 어떤 역할을 하는가?
    Resource는 서버폴더에서 꺼내온 원본 파일을 가져오는 역할을 합니다.
    Content-Disposition는 원래 파일명의 이름으로 다운로드 창을 만드는 역할을 합니다.
    ResponseEntity는 상태나 만들어진 창, 파일의 내용 들을 담아서 브라우저에 보내는 역할을 합니다.

    파일 저장 과정에서은 normalize()와 startsWith()를 사용하는 이유는 무엇인가?
    파일이름에 ../../ 와 같은 기호를 넣어 기존 폴더가 아니라 다른 상위 폴더로 이동해서 
    저장해야하는 폴더가 아닌 다른 폴더를 건드리는걸 막기 위해서입니다.


## 구현된 기능
- 게시글 목록 / 생성 / 상세 / 수정 / 삭제
- 게시글 생성 시 여러 파일 선택, UUID 이름으로 `uploads/`에 저장
- `attachments` 테이블에 메타데이터 저장
- 첨부파일 다운로드 (`GET /attachments/{id}/download`)

## 학생 과제 (TODO)
1. 코드를 분석하여 업로드/다운로드 요청 흐름 정리
2. 화면이나 정보 표시 3곳 이상 수정
3. **이미지 미리보기** 구현: `GET /attachments/{id}/view` + 상세 화면 `<img>` 표시. 이미지(jpg/jpeg/png/gif)만 허용하고 `Content-Type` 정확히 설정
4. 정상/오류 테스트 후 README 및 Weekly Review 작성

## 코드 위치
- `PostController`: 게시글 화면 / Form
- `AttachmentController`: 다운로드 + 미리보기 TODO
- `PostService`: JPA CRUD와 파일 저장 연결
- `FileStorageService`: 파일 검증 / UUID 저장 / 로드
- `Post`, `Attachment`: 1:N 관계
- `templates/posts/detail.html`: 미리보기 TODO

## 주의사항
- 학습용 프로젝트입니다. 로그인/접근 권한, 업로드 파일의 실제 바이트 검증, 악성 파일 검사, CSRF 보호, 파일 저장과 DB 트랜잭션의 완전한 일관성 보장 등은 구현하지 않았습니다. 인터넷 공개 배포 전에 보강이 필요합니다.
- `spring.jpa.hibernate.ddl-auto=update`는 학습용 설정이며 운영 DB에는 그대로 사용하지 않습니다.
- Render 기본 파일 시스템에 저장된 업로드 파일은 재시작/재배포 시 유지되지 않을 수 있습니다. 이번 과제는 **로컬 파일 처리**를 평가합니다.
