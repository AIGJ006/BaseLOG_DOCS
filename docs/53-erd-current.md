# 53. 지금 DB 구조 (ERD, Flyway V24 기준)

2026-11-25 로컬 DB에 V1~V24 마이그레이션을 모두 적용한 뒤 PostgreSQL 카탈로그(`information_schema`·`pg_constraint`·컬럼 주석)에서 자동으로 뽑았다. 손으로 고치지 말고, 새 마이그레이션이 생기면 다시 뽑는다.

- 컬럼표·제약·V2~V24 변경 이유는 [51-erd-unified.md](51-erd-unified.md)(V1 원문 + "V1 이후 바뀐 것")에 있다. 이 문서는 **한눈에 보는 그림**이다.
- `follow` 테이블은 V6에서 친구(`friendship`)로 바꾸며 행만 지우고 팀 ERD에 맞춰 남겨 두었다. 코드는 읽거나 쓰지 않는다(탈퇴 정리의 DELETE만).
- 테이블 31개, 외래 키 53개. 모듈별로 나눠 그렸고, 맨 위 그림은 테이블 사이 관계만 보인다.

## 전체 관계

```mermaid
erDiagram
  member ||--o{ auth_identity : "member_id (RESTRICT)"
  member ||--o{ blog_visit_daily : "owner_id (CASCADE)"
  member ||--o{ category : "member_id (CASCADE)"
  category |o--o{ category : "parent_id (RESTRICT)"
  member ||--o{ comment : "author_id (RESTRICT)"
  member |o--o{ comment : "hidden_by (RESTRICT)"
  comment |o--o{ comment : "post_id·parent_id (CASCADE)"
  post ||--o{ comment : "post_id (CASCADE)"
  member |o--o{ comment : "reply_to_member_id (RESTRICT)"
  member ||--o{ follow : "followee_id (RESTRICT)"
  member ||--o{ follow : "follower_id (RESTRICT)"
  member ||--o{ friendship : "member_a_id (RESTRICT)"
  member ||--o{ friendship : "member_b_id (RESTRICT)"
  member ||--o{ friendship : "requested_by (RESTRICT)"
  member ||--o{ image : "uploader_id (RESTRICT)"
  member |o--o{ inquiry : "answered_by (SET NULL)"
  image |o--o{ inquiry : "image_id (SET NULL)"
  member ||--o{ inquiry : "member_id (CASCADE)"
  member ||--o{ member_agreement : "member_id (RESTRICT)"
  member |o--o{ member_suspension : "lifted_by (RESTRICT)"
  member ||--o{ member_suspension : "member_id (RESTRICT)"
  member ||--o{ member_suspension : "suspended_by (RESTRICT)"
  member ||--o{ nest : "member_id (CASCADE)"
  member ||--o{ nest_item : "member_id (CASCADE)"
  member ||--o{ nest_point : "member_id (CASCADE)"
  comment |o--o{ notification : "comment_id (CASCADE)"
  member |o--o{ notification : "last_actor_id (RESTRICT)"
  post |o--o{ notification : "post_id (CASCADE)"
  member ||--o{ notification : "receiver_id (RESTRICT)"
  report |o--o{ notification : "report_id (SET NULL)"
  member ||--o{ notification_actor : "actor_id (RESTRICT)"
  notification ||--o{ notification_actor : "notification_id (CASCADE)"
  member ||--o{ notification_mute : "member_id (RESTRICT)"
  member ||--o{ post : "author_id (RESTRICT)"
  category |o--o{ post : "category_id (SET NULL)"
  member |o--o{ post : "hidden_by (RESTRICT)"
  post ||--o{ post_attachment : "post_id (CASCADE)"
  member ||--o{ post_collaborator : "member_id (CASCADE)"
  post ||--o{ post_collaborator : "post_id (CASCADE)"
  post ||--o{ post_draft : "post_id (CASCADE)"
  image ||--o{ post_image : "image_id (CASCADE)"
  post ||--o{ post_image : "post_id (CASCADE)"
  member ||--o{ post_like : "member_id (RESTRICT)"
  post ||--o{ post_like : "post_id (CASCADE)"
  post ||--o{ post_tag : "post_id (CASCADE)"
  tag ||--o{ post_tag : "tag_id (RESTRICT)"
  post ||--o{ post_view_daily : "post_id (CASCADE)"
  report_case ||--o{ report : "case_id (RESTRICT)"
  member ||--o{ report : "reporter_id (RESTRICT)"
  comment |o--o{ report_case : "comment_id (SET NULL)"
  member |o--o{ report_case : "handled_by (RESTRICT)"
  post |o--o{ report_case : "post_id (SET NULL)"
  member ||--o{ report_case : "target_author_id (RESTRICT)"
```

## 회원·계정 (account)

- `member` — 회원
- `auth_identity` — 로그인 수단
- `member_agreement` — 회원 동의
- `member_suspension` — 회원 정지 이력
- `friendship` — 친구 관계
- `follow` — 팔로우

```mermaid
erDiagram
  member {
    bigint id PK "회원 번호"
    varchar handle "블로그 주소"
    varchar nickname "닉네임"
    timestamptz nickname_changed_at "닉네임 변경 일자"
    varchar bio "소개"
    varchar role "권한"
    varchar status "회원 상태"
    varchar default_visibility "기본 공개 범위"
    timestamptz created_at "생성 일자"
    timestamptz updated_at "수정 일자"
    timestamptz withdrawn_at "탈퇴 신청 일자"
    timestamptz last_active_at "최근 활동 일자"
    boolean last_active_visible "최근 활동 공개 여부"
    timestamptz deleted_at "익명 처리 일자"
    varchar banner_color
  }
  auth_identity {
    bigint id PK "로그인 수단 번호"
    bigint member_id FK "회원 번호"
    varchar provider "로그인 방식"
    varchar provider_user_id "로그인 식별값"
    varchar email "이메일"
    varchar password_hash "비밀번호 해시"
    timestamptz email_verified_at "이메일 인증 일자"
    timestamptz created_at "생성 일자"
    timestamptz last_login_at "마지막 로그인 일자"
  }
  member_agreement {
    bigint member_id PK,FK "회원 번호"
    varchar type PK "동의 종류"
    varchar version "동의 버전"
    timestamptz agreed_at "동의 일자"
  }
  member_suspension {
    bigint id PK "정지 번호"
    bigint member_id FK "정지 회원 번호"
    varchar reason "정지 사유"
    timestamptz started_at "정지 시작 일자"
    timestamptz ends_at "정지 종료 일자"
    bigint suspended_by FK "정지 관리자 번호"
    timestamptz lifted_at "정지 해제 일자"
    bigint lifted_by FK "해제 관리자 번호"
  }
  friendship {
    bigint member_a_id PK,FK "회원 A 번호"
    bigint member_b_id PK,FK "회원 B 번호"
    bigint requested_by FK "요청 회원 번호"
    varchar status "관계 상태"
    timestamptz created_at "생성 일자"
    timestamptz accepted_at "수락 일자"
  }
  follow {
    bigint follower_id PK,FK "팔로우하는 회원 번호"
    bigint followee_id PK,FK "팔로우받는 회원 번호"
    timestamptz created_at "생성 일자"
  }
  member ||--o{ auth_identity : "member_id (RESTRICT)"
  member ||--o{ follow : "followee_id (RESTRICT)"
  member ||--o{ follow : "follower_id (RESTRICT)"
  member ||--o{ friendship : "member_a_id (RESTRICT)"
  member ||--o{ friendship : "member_b_id (RESTRICT)"
  member ||--o{ friendship : "requested_by (RESTRICT)"
  member ||--o{ member_agreement : "member_id (RESTRICT)"
  member |o--o{ member_suspension : "lifted_by (RESTRICT)"
  member ||--o{ member_suspension : "member_id (RESTRICT)"
  member ||--o{ member_suspension : "suspended_by (RESTRICT)"
```

## 글 (post·category)

- `post` — 글
- `post_draft` — 글 작업본
- `post_collaborator` — —
- `category` — —
- `post_attachment` — —
- `attachment_trash` — —

```mermaid
erDiagram
  post {
    bigint id PK "글 번호"
    bigint author_id FK "작성자 번호"
    varchar title "제목"
    text content_md "본문 원문"
    text content_html "렌더링 본문"
    varchar excerpt "목록 요약"
    varchar thumbnail_url "썸네일 주소"
    varchar status "글 상태"
    varchar visibility "공개 범위"
    bigint view_count "조회 수"
    int like_count "좋아요 수"
    int comment_count "댓글 수"
    bigint edit_version "편집 버전"
    int render_version "렌더링 버전"
    timestamptz published_at "최초 발행 일자"
    timestamptz first_public_at "최초 공개 일자"
    timestamptz edited_at "재발행 일자"
    timestamptz created_at "생성 일자"
    timestamptz updated_at "수정 일자"
    timestamptz deleted_at "삭제 일자"
    timestamptz hidden_at "숨김 일자"
    bigint hidden_by FK "숨긴 관리자 번호"
    varchar hidden_reason "숨김 사유"
    bigint category_id FK
    timestamptz pinned_at "고정한 시각 (없으면 고정 안 함, 휴지통으로 옮기면 지움)"
  }
  post_draft {
    bigint post_id PK,FK "글 번호"
    varchar title "작업 제목"
    text content_md "작업 본문"
    bigint edit_version "편집 버전"
    timestamptz created_at "생성 일자"
    timestamptz updated_at "수정 일자"
  }
  post_collaborator {
    bigint post_id PK,FK
    bigint member_id PK,FK
    timestamptz created_at
  }
  category {
    bigint id PK
    bigint member_id FK
    bigint parent_id FK
    varchar name
    varchar name_key
    int position
    timestamptz created_at
    timestamptz updated_at
  }
  post_attachment {
    bigint id PK
    bigint post_id FK
    varchar storage_key
    varchar original_name
    bigint size_bytes
    timestamptz created_at
  }
  attachment_trash {
    varchar storage_key PK
    timestamptz queued_at
  }
  category |o--o{ category : "parent_id (RESTRICT)"
  category |o--o{ post : "category_id (SET NULL)"
  post ||--o{ post_attachment : "post_id (CASCADE)"
  post ||--o{ post_collaborator : "post_id (CASCADE)"
  post ||--o{ post_draft : "post_id (CASCADE)"
```

## 사진 (media)

- `image` — 사진
- `post_image` — 글-사진 연결

```mermaid
erDiagram
  image {
    bigint id PK "사진 번호"
    bigint uploader_id FK "올린 회원 번호"
    varchar storage_key "저장 경로"
    varchar thumb_storage_key "썸네일 경로"
    varchar content_type "파일 형식"
    int size_bytes "원본 크기"
    int thumb_size_bytes "썸네일 크기"
    int width "가로"
    int height "세로"
    varchar status "사진 상태"
    varchar purpose "용도"
    timestamptz detached_at "연결 해제 일자"
    timestamptz created_at "생성 일자"
  }
  post_image {
    bigint post_id PK,FK "글 번호"
    bigint image_id PK,FK "사진 번호"
  }
  image ||--o{ post_image : "image_id (CASCADE)"
```

## 태그 (tag)

- `tag` — 태그
- `post_tag` — 글-태그 연결

```mermaid
erDiagram
  tag {
    bigint id PK "태그 번호"
    varchar name "태그 이름"
    timestamptz created_at "생성 일자"
  }
  post_tag {
    bigint post_id PK,FK "글 번호"
    bigint tag_id PK,FK "태그 번호"
    smallint position "입력 순서"
  }
  tag ||--o{ post_tag : "tag_id (RESTRICT)"
```

## 반응 (interaction)

- `comment` — 댓글
- `post_like` — 글 좋아요
- `post_view_daily` — 일별 조회 수

```mermaid
erDiagram
  comment {
    bigint id PK "댓글 번호"
    bigint post_id FK "글 번호"
    bigint author_id FK "작성자 번호"
    bigint parent_id FK "부모 댓글 번호"
    bigint reply_to_member_id FK "답글 대상 회원 번호"
    varchar content "내용"
    timestamptz created_at "생성 일자"
    timestamptz updated_at "수정 일자"
    timestamptz deleted_at "삭제 일자"
    timestamptz hidden_at "숨김 일자"
    bigint hidden_by FK "숨긴 관리자 번호"
    varchar hidden_reason "숨김 사유"
    boolean secret "비밀 댓글 여부"
  }
  post_like {
    bigint post_id PK,FK "글 번호"
    bigint member_id PK,FK "회원 번호"
    timestamptz created_at "생성 일자"
  }
  post_view_daily {
    bigint post_id PK,FK "글 번호"
    date view_date PK "조회 일자"
    int views "조회 수"
  }
  comment |o--o{ comment : "post_id·parent_id (CASCADE)"
```

## 알림 (notification)

- `notification` — 알림
- `notification_actor` — 알림에 묶인 사람
- `notification_mute` — 끈 알림 종류

```mermaid
erDiagram
  notification {
    bigint id PK "알림 번호"
    bigint receiver_id FK "받는 회원 번호"
    varchar type "알림 종류"
    bigint post_id FK "관련 글 번호"
    bigint comment_id FK "관련 댓글 번호"
    bigint report_id FK "관련 신고 번호"
    varchar result "신고 결과"
    bigint last_actor_id FK "마지막 행동 회원 번호"
    int actor_count "묶인 인원 수"
    varchar group_key "묶음 키"
    timestamptz read_at "읽은 일자"
    timestamptz created_at "생성 일자"
    timestamptz updated_at "수정 일자"
  }
  notification_actor {
    bigint notification_id PK,FK "알림 번호"
    bigint actor_id PK,FK "행동 회원 번호"
    timestamptz created_at "생성 일자"
  }
  notification_mute {
    bigint member_id PK,FK "회원 번호"
    varchar type PK "끈 알림 종류"
    timestamptz created_at "생성 일자"
  }
  notification ||--o{ notification_actor : "notification_id (CASCADE)"
```

## 신고·문의 (moderation·inquiry)

- `report` — 신고
- `report_case` — 신고 사건
- `inquiry` — 문의·신고

```mermaid
erDiagram
  report {
    bigint id PK "신고 번호"
    bigint case_id FK "신고 사건 번호"
    bigint reporter_id FK "신고자 번호"
    varchar reason "신고 사유"
    varchar detail "신고 설명"
    timestamptz created_at "생성 일자"
  }
  report_case {
    bigint id PK "신고 사건 번호"
    varchar target_type "신고 대상 종류"
    bigint post_id FK "신고 글 번호"
    bigint comment_id FK "신고 댓글 번호"
    bigint target_author_id FK "대상 작성자 번호"
    varchar snapshot_title "신고 시점 제목"
    varchar snapshot_content "신고 시점 내용"
    varchar status "처리 상태"
    bigint handled_by FK "처리 관리자 번호"
    timestamptz handled_at "처리 일자"
    timestamptz created_at "생성 일자"
  }
  inquiry {
    bigint id PK
    bigint member_id FK
    varchar category "종류(버그 신고·계정 문의·사용자 신고·기타)"
    text content
    varchar target_handle "신고할 사람 블로그 주소(사용자 신고만)"
    bigint image_id FK "붙인 사진 번호"
    varchar status "상태(접수·답변 완료)"
    text answer "관리자 답변"
    bigint answered_by FK "답한 관리자 회원 번호"
    timestamptz answered_at "답한 일자"
    timestamptz created_at
  }
  report_case ||--o{ report : "case_id (RESTRICT)"
```

## 새 키우기 (nest)

- `nest` — 둥지(새 크기·연속 출석·장착한 배경과 둥지)
- `nest_point` — 둥지 포인트 기록(합이 잔액)
- `nest_item` — 산 둥지 꾸미기 물건

```mermaid
erDiagram
  nest {
    bigint member_id PK,FK
    smallint bird_level
    int streak_days
    int best_streak_days
    date last_attended_on
    varchar background
    varchar nest_type
    timestamptz created_at
    timestamptz updated_at
    boolean show_on_blog "내 둥지 프로필에 새를 보이나 (기본 꺼짐)"
  }
  nest_point {
    bigint id PK
    bigint member_id FK
    varchar reason
    int amount
    varchar source_key
    date earned_on
    timestamptz created_at
  }
  nest_item {
    bigint member_id PK,FK
    varchar item_code PK
    boolean equipped
    timestamptz bought_at
  }
```

## 통계·인프라 (stats·shared)

- `site_visit_daily` — 일별 방문자 수
- `blog_visit_daily` — 둥지별 일별 방문자 수
- `shedlock` — 배치 실행 잠금 (ShedLock JDBC)

```mermaid
erDiagram
  site_visit_daily {
    date visit_date PK "방문 일자"
    int visitors "방문자 수(회원·비회원)"
    int member_visitors "그중 로그인 회원 수"
    timestamptz updated_at
  }
  blog_visit_daily {
    bigint owner_id PK,FK "둥지 주인 회원 번호"
    date visit_date PK "방문 일자"
    int visitors "방문자 수(회원·비회원, 주인 본인 제외)"
    timestamptz updated_at
  }
  shedlock {
    varchar name PK
    timestamptz lock_until
    timestamptz locked_at
    varchar locked_by
  }
```
