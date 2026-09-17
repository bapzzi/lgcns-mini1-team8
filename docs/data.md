# 테이블 명세 · ERD 초안 (2026-09-18, 팀 확정 전)

팀장 초안입니다. 9/18 회의에서 확인한 뒤 확정합니다. 바꾸려면 `CONTRIBUTING.md`의 「규격을 바꿔야 할 때」를 따릅니다.

## 공통 규칙 [팀장 안]
- DB = MySQL. 테이블 · 칸 이름은 영문 snake_case, 테이블 이름은 단수
- 기본키는 `BIGINT AUTO_INCREMENT`, 일시는 `DATETIME`(한국 시간)
- 코드값(직무 · 고용형태 등)은 코드 표를 따로 만들지 않고 `VARCHAR` 코드로 둔다. 테이블 수를 줄이기 위해서다
- 외부 원본에서 온 행은 `source` + 원본 번호로 유일하게 묶어, 수집을 다시 돌려도 중복이 생기지 않게 한다

## 미정이 반영된 곳
| 미정 | 반영 방식 |
|---|---|
| 로그인 방식 A · B | `app_user`에 A안 칸(`login_id` · `password_hash`)과 B안 칸(`google_sub`)을 둘 다 두었다. 확정되면 안 쓰는 쪽을 지운다 |
| 블로그 수집 · 본문 저장 | `blog_post.content_text`는 본문을 저장할 때만 쓴다 |
| 추천률 계산 | 저장하지 않고 조회할 때 계산한다고 가정. 저장이 필요해지면 칸을 추가한다 |
| 공고 수집 경로 | `job_posting.source`로 출처를 구분해 경로가 바뀌어도 표는 그대로 쓴다 |

## 관계
| 부모 | 자식 | 관계 | 뜻 |
|---|---|---|---|
| app_user | user_employment_type | 1 : N | 사용자가 원하는 고용형태 여러 개 |
| app_user | user_skill | 1 : N | 보유 기술 · 키우고 싶은 기술 |
| tech_tag | user_skill | 1 : N | |
| company | job_posting | 1 : N | 기업의 공고 |
| company | blog_post | 1 : N | 기업의 기술블로그 글 |
| job_posting | job_posting_tag | 1 : N | 공고의 지원자격 · 우대 기술 |
| tech_tag | job_posting_tag | 1 : N | |
| blog_post | blog_post_tag | 1 : N | 글의 기술 태그 |
| tech_tag | blog_post_tag | 1 : N | |
| app_user | ai_guide | 1 : N | 사용자별 AI 가이드 |
| blog_post | ai_guide | 1 : N | |
| job_posting | ai_guide | 1 : N (선택) | 공고 기준 가이드만 연결, 프로필 기준은 비움 |
| app_user | bookmark | 1 : N | |
| job_posting | bookmark | 1 : N | |

## 테이블 (11개)

### app_user · 사용자와 프로필
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK | user_id | BIGINT | |
| UK | login_id | VARCHAR(50) NULL | A안 아이디 |
| | password_hash | VARCHAR(255) NULL | A안 암호화한 비밀번호 |
| UK | google_sub | VARCHAR(100) NULL | B안 Google 계정 식별값 |
| | name | VARCHAR(50) | |
| | job_group_code | VARCHAR(20) NULL | 희망 직무. 02 입력 전에는 비어 있음 → 비어 있으면 02로 보냄 |
| | career_years | INT NULL | 경력 년수 |
| | profile_version | INT | 프로필 저장마다 +1. AI 가이드 재사용 기준 |
| | created_at · updated_at | DATETIME | |

프로필을 따로 떼지 않고 사용자 표에 넣었다. 사용자 1명에 프로필 1개라서다.

### user_employment_type · 원하는 고용형태
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK · FK | user_id | BIGINT | → app_user |
| PK | employment_type_code | VARCHAR(20) | 정규직 · 계약직 · 인턴 |

### user_skill · 보유 기술과 키우고 싶은 기술
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK · FK | user_id | BIGINT | → app_user |
| PK · FK | tech_tag_id | BIGINT | → tech_tag |
| | skill_type | VARCHAR(10) | HAVE 보유 / WANT 키우고 싶은 |

기본키가 (사용자, 기술)이라 같은 기술을 보유와 키우고 싶은 쪽에 동시에 넣을 수 없다.

### tech_tag · 기술명 사전
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK | tech_tag_id | BIGINT | |
| UK | name | VARCHAR(50) | 표준 이름. 예 Kafka |
| | aliases | VARCHAR(255) | 다른 표기, 쉼표로 구분. 예 kafka,카프카 |
| | category | VARCHAR(30) | Backend · Frontend · Data 등 |

### company · 기업
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK | company_id | BIGINT | |
| | name | VARCHAR(100) | 우아한형제들 |
| | career_url | VARCHAR(255) | 채용 사이트 |
| | blog_feed_url | VARCHAR(255) NULL | 블로그 수집 주소(방식 미정) |

### job_posting · 채용 공고
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK | job_posting_id | BIGINT | |
| FK | company_id | BIGINT | → company |
| UK① | source | VARCHAR(30) | 수집 출처 |
| UK① | source_posting_id | VARCHAR(50) | 원본 공고 번호 |
| | title | VARCHAR(200) | |
| | job_group_code | VARCHAR(20) | 직무 |
| | employment_type_code | VARCHAR(20) | 고용형태 |
| | min_career_years · max_career_years | INT NULL | 경력 조건 |
| | deadline_at | DATETIME NULL | 비어 있으면 상시 채용 |
| | original_url | VARCHAR(255) | 원본 공고 |
| | requirement_text | TEXT | `[지원자격]` 구역 |
| | preferred_text | TEXT | `[우대사항]` 구역 |
| | first_seen_at · last_seen_at | DATETIME | 처음 · 마지막으로 수집에서 본 날. 마지막 확인일로 마감 판정 |

### job_posting_tag · 공고의 기술
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK · FK | job_posting_id | BIGINT | → job_posting |
| PK · FK | tech_tag_id | BIGINT | → tech_tag |
| PK | section | VARCHAR(10) | REQUIRED 지원자격 / PREFERRED 우대사항 |

추천 제외(지원자격 기술 미보유)와 우대 기술 m / n 계산이 이 표에서 나온다.

### blog_post · 기술블로그 글
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK | blog_post_id | BIGINT | |
| FK | company_id | BIGINT | → company |
| UK | url | VARCHAR(255) | 원문 주소. 중복 수집 방지 |
| | title | VARCHAR(200) | |
| | category | VARCHAR(50) | 블로그 카테고리 |
| | published_at | DATETIME | |
| | excerpt | VARCHAR(500) | 화면에 보여 줄 앞부분 발췌 |
| | content_text | MEDIUMTEXT NULL | AI 입력 · 태그 추출용 본문. 저장 여부 미정 |
| | collected_at | DATETIME | |

### blog_post_tag · 글의 기술
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK · FK | blog_post_id | BIGINT | → blog_post |
| PK · FK | tech_tag_id | BIGINT | → tech_tag |

### ai_guide · 사용자별 AI 가이드
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK | ai_guide_id | BIGINT | |
| FK | user_id | BIGINT | → app_user |
| FK | blog_post_id | BIGINT | → blog_post |
| FK | job_posting_id | BIGINT NULL | → job_posting. 비어 있으면 프로필 기준 |
| | mode | VARCHAR(10) | POSTING 공고 기준 / PROFILE 프로필 기준 |
| | profile_version | INT | 만들 때의 프로필 버전 |
| | section1_text · section2_text · section3_text | TEXT | 가이드 3칸 |
| | created_at | DATETIME | |

재사용 조회 기준 = 사용자 + 글 + mode + 공고 + 프로필 버전. MySQL은 NULL이 들어간 유일 제약을 중복으로 보지 않으므로, 저장 전에 서비스에서 먼저 조회한다.

### bookmark · 북마크
| 키 | 칸 | 형식 | 설명 |
|---|---|---|---|
| PK | bookmark_id | BIGINT | |
| FK · UK② | user_id | BIGINT | → app_user |
| FK · UK② | job_posting_id | BIGINT | → job_posting. 같은 공고 중복 저장 불가 |
| | created_at | DATETIME | 최근 저장 순 정렬 |

## 줄일 수 있는 곳 (팀 판단)
- `user_employment_type` → `app_user`에 쉼표 문자열 한 칸으로 합치면 10개. 대신 고용형태로 거르는 조회가 지저분해진다
- `blog_post_tag` · `job_posting_tag` → 기술 이름 문자열 칸으로 합치면 9개. 대신 추천 제외 · 글 연결 계산을 코드에서 문자열로 처리해야 한다
