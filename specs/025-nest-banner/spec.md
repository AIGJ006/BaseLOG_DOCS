# Feature Specification: 둥지 배너

**Feature Branch**: `025-nest-banner`

**Created**: 2026-10-15

**Status**: Implemented (BaseLOG PR #133 — dev)

**Input**: 민서 요청(2026-10-15) 둥지 머리말에 넓은 배너 사진 한 장을 두고, 설정 > 프로필에서 고르고 자르게 해 달라. 나민서 개인 확장 — 공통 스펙과 원문 docs/는 바꾸지 않는다(constitution I).

## Clarifications

### Session 2026-10-15

- Q: 어디에 보이나? → A: 둥지(`/@handle`) 맨 위, 머리말(닉네임·주소·소개) 위의 넓은 띠. 프로필 사진은 배너 아래 끝에 살짝 걸치고, 글자는 배너 밖(아래)에 둔다. 배너 위에 글자를 올리지 않는다.
- Q: 모양은? → A: 어느 화면에서나 가로 4:1 그대로. 자를 때 고른 부분이 그대로 보인다. 모서리는 둥글다.
- Q: 배너가 없으면? → A: 연한 초록 기본 배경(`--color-brand-soft`)만 칠한다.
- Q: 어디서 바꾸나? → A: 설정 > 프로필 탭의 [배너] 칸. 내 둥지면 배너 오른쪽 아래 [배너 바꾸기]가 설정(`/settings`, 기본 탭 프로필)으로 보낸다. 남의 둥지에는 없다.
- Q: 자르기는? → A: 프로필 사진 자르기 화면을 같이 쓴다. 비율만 4:1로 받아서 1500×375로 다시 그린다. 결과는 WebP, 못 만드는 브라우저는 JPEG.
- Q: 언제 올리나? → A: 프로필 사진과 같다. 고르고 자른 뒤 [저장]을 누를 때 올리고 연결한다. [저장] 전에는 연결하지 않는다.
- Q: 몇 장? → A: 회원당 지금 배너 한 장. 바꾸면 이전 배너는 떼고, 뗀 사진은 정리 작업이 7일 뒤 지운다.

## User Scenarios & Testing

### User Story 1 - 둥지에서 배너 보기 (P1)

1. **Given** 배너가 있는 둥지, **When** 둥지를 열면, **Then** 머리말 위에 4:1 배너 사진이 보이고 닉네임은 배너 아래에 있다.
2. **Given** 배너가 없는 둥지, **Then** 연한 초록 띠만 보인다(사진 없음).
3. **Given** 내 둥지, **Then** 배너 오른쪽 아래에 [배너 바꾸기]. 누르면 설정 > 프로필로 간다. 남의 둥지에는 이 단추가 없다.

### User Story 2 - 설정에서 배너 바꾸기 (P1)

1. **Given** 설정 > 프로필, **Then** [배너] 칸에 지금 배너 미리보기와 "둥지 맨 위에 넓게 보여요. 고른 사진에서 가로 4:1로 보일 영역을 골라요".
2. **When** [사진 고르기]로 사진을 고르면, **Then** 4:1 자르기 화면. 위치·확대를 고르고 [이 영역 사용]을 누르면 미리보기가 새 배너로 바뀐다.
3. **When** [저장], **Then** 배너를 올리고 연결한 뒤 "저장했어요". 둥지에 새 배너가 보인다.
4. **When** [배너 지우기] 뒤 [저장], **Then** 기본 배경으로 돌아간다. [배너 지우기]는 배너가 있거나 새로 자른 것이 있을 때만 보인다.
5. **Given** 올리기에 실패하면, **Then** 저장하지 않고 "배너를 올리지 못했어요. 다시 시도해 주세요".

### User Story 3 - 잘못된 사진 막기 (P2)

1. 고를 수 있는 파일은 jpg·png·gif·webp, 10MB 이하(자르기 전 검사, 프로필 사진과 같음).
2. 올릴 때 형식이 WebP·JPEG가 아니면 400 `UNSUPPORTED_IMAGE_TYPE`, 2MB를 넘으면 400 `IMAGE_TOO_LARGE`, 썸네일 칸을 보내면 400 `THUMBNAIL_NOT_ALLOWED`.
3. 완료 확인에서 정확히 1500×375가 아니면 400 `details.reason` = `BANNER_SIZE_INVALID`.
4. 완료 전 사진·프로필 사진·남의 배너를 `bannerImageId`로 보내면 400 칸 오류 `INVALID_BANNER_IMAGE` "사용할 수 없는 사진이에요".

## Requirements

- **FR-001** 둥지 머리말 위에 배너 띠를 둔다. 비율 4:1, 사진은 `object-fit: cover`. 없으면 `--color-brand-soft` 배경만.
- **FR-002** 프로필 사진은 배너 아래 끝에 걸친다(위로 2.5rem 올림, 배경색 테두리). 닉네임·주소·소개는 배너 밖에 둔다.
- **FR-003** 내 둥지(`isMe`)에서만 [배너 바꾸기] 링크(`/settings`)를 보인다.
- **FR-004** 설정 > 프로필에서 고르기 → 4:1 자르기(1500×375) → [저장] 때 올리고 `PATCH /api/me/profile {bannerImageId}`로 연결한다. `bannerImageId: null`은 기본 배경으로.
- **FR-005** 사진 용도 `BANNER`. 올리기는 WebP·JPEG, 2MB 이하, 썸네일 없음. 완료 확인은 정확히 1500×375.
- **FR-006** 붙일 수 있는 사진은 그 회원이 올린 `BANNER`이면서 완료 확인을 통과한 것(`width IS NOT NULL`)뿐이다.
- **FR-007** 지금 배너는 회원당 한 장. 붙일 때 "떼기 → 붙이기" 순서로 하고, 프로필 저장 트랜잭션 안에서 회원 행을 잠근 뒤 한다. 같은 사진을 다시 보내면 아무것도 하지 않는다.
- **FR-008** 뗀 배너는 정리 작업이 7일 뒤(`detached-ttl` P7D) 지운다. 탈퇴 정리는 기존 사진 단계가 배너도 함께 뗀다.
- **FR-009** 내 프로필 응답과 둥지 머리말 응답에 배너 주소를 넣는다. 둥지 머리말은 media `BannerImageService`로만 만든다(image 테이블을 직접 읽지 않음).
- **FR-010** 색은 tokens.css 변수만 쓴다.

## 데이터 (data-model)

- **§1 V14__member_banner.sql** — `member`에 칸을 두지 않는다. 프로필 사진과 같이 `image` 행의 용도와 연결 상태로 지금 배너를 정한다.
  - `ck_image_purpose`를 지우고 `('POST', 'PROFILE', 'BANNER')`로 다시 만든다(`NOT VALID` 뒤 `VALIDATE`).
  - 부분 유일 인덱스 `uq_image_banner_current ON image (uploader_id) WHERE purpose = 'BANNER' AND status = 'ATTACHED' AND detached_at IS NULL`.
- **§2 지금 배너** — 위 인덱스 조건의 행 하나. 떼면 `detached_at = now()`.
- **§3 설정** `blog.image.banner-width` 1500, `banner-height` 375, `banner-max-bytes` 2097152(최대 10MB까지 설정 가능).

## API (contracts)

| 요청 | 바뀐 점 |
|---|---|
| `POST /api/images/presign` | `purpose: "BANNER"` 추가. WebP·JPEG, 2MB 이하, `thumbContentType`·`thumbSize` 받지 않음 |
| `POST /api/images/{imageId}/complete` | `BANNER`면 정확히 1500×375, 아니면 400 `BANNER_SIZE_INVALID` |
| `GET /api/me/profile` | `bannerImageId`, `bannerImageUrl` (없으면 `null`) |
| `PATCH /api/me/profile` | `bannerImageId` 칸. 숫자면 붙이기, `null`이면 떼기, 보내지 않으면 그대로. 잘못된 사진은 400 `INVALID_BANNER_IMAGE` |
| `GET /api/members/{handle}` | `bannerImageUrl` (없으면 `null`) |

화면 흐름: presign → 저장소에 PUT → complete → `PATCH /api/me/profile {bannerImageId}`.

## 시험

- 백엔드: `BannerImageIT`(presign 형식·크기·썸네일 거절, complete 1500×375만 통과, 붙이면 내 프로필·둥지 머리말에 주소·바꾸면 이전 배너 떼기, null이면 기본 배경, 완료 전·프로필 사진·남의 배너 400, 탈퇴 정리), `ProfileUpdateRequestTest`(번호·null·안 보냄 구분), `BlogHeaderFriendIT`(머리말 칸에 `bannerImageUrl`), `FlywayBaselineIntegrationTest`(`uq_image_banner_current` 회원당 한 장, 프로필 사진과 따로), `ImageHeaderReaderTest`(`banner-1500x375.webp`·`banner-1500x376.webp`).
- 화면: `BlogPage.test.tsx`(기본 배경 띠, 배너는 머리말 위·닉네임은 밖, 내 둥지만 [배너 바꾸기]), `SettingsPage.test.tsx`(고르기 → 1500×375 자르기 → [저장] 때 올리고 연결, [배너 지우기]는 null, 올리기 실패면 저장 안 함), `ProfileImageCropper.test.tsx`(4:1 자르기·배너 크기로 그리기).
