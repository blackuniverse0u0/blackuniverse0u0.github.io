# 포트폴리오 업데이트 가이드

이 문서는 GitHub Pages (al-folio) 포트폴리오를 더 구체적이고 전문적으로 만들기 위한 단계별 가이드입니다.

---

## 📋 목차

1. [현재 상태 분석](#현재-상태-분석)
2. [우선순위별 업데이트 항목](#우선순위별-업데이트-항목)
3. [상세 업데이트 가이드](#상세-업데이트-가이드)
4. [Jekyll/Liquid 문법 기초](#jekylliquid-문법-기초)
5. [로컬 테스트 방법](#로컬-테스트-방법)

---

## 현재 상태 분석

### ✅ 완료된 것들
- [x] 3개의 프로젝트 페이지 (상세 설명 포함)
- [x] 3개의 기술 블로그 포스트
- [x] About 페이지 기본 작성
- [x] CV에 프로젝트 추가 (resume.json)

### ❌ 아직 안 된 것들
- [ ] 프로필 사진 (`assets/img/prof_pic.jpg`)
- [ ] 소셜 미디어 링크 (GitHub, LinkedIn 등)
- [ ] 뉴스/공지사항 (`_news/`)
- [ ] 프로젝트 대표 이미지들
- [ ] Resume.json 개인정보 업데이트
- [ ] 네비게이션 메뉴 정리
- [ ] Publications (논문이 있다면)

---

## 우선순위별 업데이트 항목

### 🔥 High Priority (지금 바로)

#### 1. 프로필 사진 추가
**위치**: `assets/img/prof_pic.jpg`

**방법**:
```bash
# 1. 프로필 사진을 준비 (정방형이 좋음, 최소 400x400px)
# 2. 파일을 prof_pic.jpg로 저장
# 3. assets/img/ 폴더에 복사

cp /path/to/your/photo.jpg assets/img/prof_pic.jpg

# Git에 추가
git add assets/img/prof_pic.jpg
git commit -m "Add profile picture"
git push
```

**팁**:
- 정방형 사진이 가장 좋음
- 배경이 깔끔한 사진 추천
- 파일 크기: 500KB 이하

---

#### 2. 소셜 미디어 링크 설정
**위치**: `_config.yml` (200-250번 줄 근처)

**현재 상태 확인**:
```bash
# _config.yml에서 social 섹션 찾기
grep -n "github_username" _config.yml
```

**수정 방법**:
`_config.yml` 파일을 열고 다음 섹션을 찾아 수정:

```yaml
# Social integration (약 200번 줄 근처)
# -----------------------------------------------------------------------------

github_username: blackuniverse0u0  # GitHub 사용자명
gitlab_username: # your GitLab user name
twitter_username: # your Twitter handle
mastodon_username: # your mastodon instance+username (e.g., @username@instance.com)
linkedin_username: # your LinkedIn user name
telegram_username: # your Telegram user name
scholar_userid: # your Google Scholar ID
semanticscholar_id: # your Semantic Scholar ID
whatsapp_number: # your WhatsApp number (full phone number in international format. e.g., +12024561111)
orcid_id: # your ORCID ID
medium_username: # your Medium username
quora_username: # your Quora username
publons_id: # your ID on Publons
lattes_id: # your ID on Lattes (Brazilian Lattes CV)
osf_id: # your OSF ID
research_gate_profile: # your ResearchGate profile name
scopus_id: # your Scopus ID
blogger_url: # your Blogger URL
work_url: # work page URL
keybase_username: # your Keybase username
wikidata_id: # your Wikidata ID
wikipedia_id: # your Wikipedia ID (Case sensitive)
dblp_url: # your DBLP profile URL
stackoverflow_id: # your StackOverflow ID
kaggle_id: # your Kaggle ID
lastfm_id: # your Last.fm ID
spotify_id: # your Spotify ID
pinterest_id: # your Pinterest ID
unsplash_id: # your Unsplash ID
instagram_id: # your Instagram ID
facebook_id: # your Facebook ID
youtube_id: # your YouTube channel ID (not the channel name!)
discord_id: # your Discord ID (e.g., 18-digit unique numerical ID)
zotero_username: # your Zotero username
wechat_qr: # filename of your wechat qr-code saved as an image (e.g., wechat-qr.png)
```

**작성 예시**:
```yaml
github_username: blackuniverse0u0
linkedin_username: joonhyun-shin-robotics  # LinkedIn URL의 마지막 부분
twitter_username: your_twitter
scholar_userid: YOUR_GOOGLE_SCHOLAR_ID
```

**Google Scholar ID 찾는 방법**:
1. Google Scholar 프로필 접속
2. URL 확인: `https://scholar.google.com/citations?user=XXXXX`
3. `XXXXX` 부분이 scholar_userid

---

#### 3. Resume.json 개인정보 업데이트
**위치**: `assets/json/resume.json`

**현재 문제**: Albert Einstein 샘플 데이터가 남아있음

**수정 방법**:

```json
{
  "basics": {
    "name": "신준현",  // 또는 "Joonhyun Shin"
    "label": "Robotics Engineer",
    "image": "",
    "email": "your.email@example.com",
    "phone": "+82-10-XXXX-XXXX",  // 선택사항
    "url": "https://blackuniverse0u0.github.io",
    "summary": "Robotics engineer specializing in perception, planning, and control for autonomous systems. Experience in quadruped locomotion, semantic navigation, and production AI deployment.",
    "location": {
      "city": "Seoul",
      "countryCode": "KR",
      "region": "Seoul"
    },
    "profiles": [
      {
        "network": "GitHub",
        "username": "blackuniverse0u0",
        "url": "https://github.com/blackuniverse0u0"
      },
      {
        "network": "LinkedIn",
        "username": "joonhyun-shin",
        "url": "https://linkedin.com/in/joonhyun-shin"
      }
    ]
  },

  "work": [
    {
      "name": "회사명 또는 연구실",
      "position": "Robotics Engineer / Researcher",
      "url": "https://company-website.com",
      "startDate": "2023-01-01",
      "endDate": "",  // 현재 진행중이면 비워두기
      "summary": "Developed autonomous navigation systems for urban robots",
      "highlights": [
        "Implemented BEV-MPPI planner with 95% success rate",
        "Deployed AI tracking system on defense robots",
        "Trained quadruped locomotion policies with PPO"
      ]
    }
    // 더 추가 가능
  ],

  "education": [
    {
      "institution": "대학교 이름",
      "location": "Seoul, South Korea",
      "url": "https://university.ac.kr",
      "area": "Robotics / Computer Science / Mechanical Engineering",
      "studyType": "Bachelor / Master / PhD",
      "startDate": "2018-03-01",
      "endDate": "2022-02-28",
      "score": "3.8/4.0",  // 선택사항
      "courses": [
        "Robot Dynamics and Control",
        "Computer Vision",
        "Reinforcement Learning"
      ]
    }
  ],

  "skills": [
    {
      "name": "Robotics & Control",
      "level": "Advanced",
      "icon": "fa-solid fa-robot",
      "keywords": [
        "Model Predictive Control",
        "MPPI",
        "Inverse Kinematics",
        "SLAM",
        "ROS/ROS2"
      ]
    },
    {
      "name": "Machine Learning",
      "level": "Advanced",
      "icon": "fa-solid fa-brain",
      "keywords": [
        "Deep Reinforcement Learning",
        "Computer Vision",
        "PyTorch",
        "JAX",
        "Semantic Segmentation"
      ]
    },
    {
      "name": "Programming",
      "level": "Advanced",
      "icon": "fa-solid fa-code",
      "keywords": [
        "Python",
        "C++",
        "Docker",
        "Git",
        "Linux"
      ]
    }
  ],

  // projects 섹션은 이미 업데이트됨
  "projects": [
    // ... 이미 작성된 3개 프로젝트
  ]
}
```

---

### ⚡ Medium Priority (이번 주 내로)

#### 4. 뉴스/공지사항 추가
**위치**: `_news/` 폴더

**작성 방법**:

```bash
# 새 뉴스 파일 생성
cd _news/
```

**파일명 규칙**: `announcement_N.md` (N은 숫자)

**예시 1: 인라인 공지** (`_news/announcement_4.md`):
```markdown
---
layout: post
date: 2024-12-20 07:59:00-0400
inline: true
related_posts: false
---

Started working on SideWalker project - autonomous navigation for urban sidewalks using BEV-MPPI! 🚀
```

**예시 2: 긴 공지** (`_news/announcement_5.md`):
```markdown
---
layout: post
title: Deployed AI Tracking System on Defense Robots
date: 2024-11-15 16:11:00-0400
inline: false
related_posts: false
---

Successfully deployed the Gremsy Box AI tracking system on defense robots!

Key achievements:
- <100ms detection-to-control latency
- TensorRT optimization: 15 FPS → 45 FPS
- Docker-based deployment for easy updates
- WebRTC streaming with sub-200ms latency

This system is now in production use for autonomous surveillance missions.
```

**예시 3: 프로젝트 완료** (`_news/announcement_6.md`):
```markdown
---
layout: post
date: 2024-10-01 15:59:00-0400
inline: true
related_posts: false
---

Completed quadruped locomotion RL training! Achieved 1.8 m/s walking speed with JAX-accelerated PPO 🦿
```

**정렬 순서**: 날짜가 최신인 것이 위에 표시됩니다.

---

#### 5. 프로젝트 이미지 개선

**현재 상태**:
- `assets/img/sidewalker.png` ✅
- `assets/img/gremsy_box.png` ✅
- `assets/img/locomotion_rl.png` ✅ (MuJoCo 배너)

**개선 방안**:

**Option 1: 실제 프로젝트 스크린샷 사용**
```bash
# 각 프로젝트의 대표 이미지를 찾아서 복사
# 예: SideWalker의 BEV 시각화 스크린샷

# projects/ 디렉토리에서 좋은 이미지 찾기
find projects/perception_planning_control/SideWalker -name "*.png" -o -name "*.jpg"

# 선택한 이미지를 assets/img로 복사
cp projects/perception_planning_control/SideWalker/path/to/good_image.png \
   assets/img/sidewalker.png

# Git에 추가
git add assets/img/sidewalker.png
git commit -m "Update SideWalker project image"
```

**Option 2: 여러 이미지를 프로젝트 페이지에 추가**

프로젝트 마크다운 파일 내에 이미지 추가:

```markdown
<!-- _projects/sidewalker.md -->

## Results

### BEV Semantic Map
![BEV Map](../assets/img/projects/sidewalker_bev.png)

### Navigation in Action
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/sidewalker_nav1.jpg" title="Navigation 1" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/sidewalker_nav2.jpg" title="Navigation 2" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
```

**이미지 준비 가이드**:
- 대표 이미지 (프로젝트 카드용): 16:9 비율, 1200x675px
- 상세 이미지 (프로젝트 페이지 내): 다양한 크기 가능
- 파일 형식: JPG (사진), PNG (스크린샷, 다이어그램)
- 파일 크기: 각 이미지 1MB 이하

---

#### 6. 네비게이션 메뉴 정리

**위치**: `_pages/` 폴더의 각 파일 front matter

**현재 메뉴 확인**:
```bash
# 현재 네비게이션에 표시되는 페이지 확인
grep -r "nav: true" _pages/
```

**메뉴 순서 조정**:

각 페이지의 `nav_order` 값을 수정:

```yaml
# _pages/about.md
---
nav_order: 1  # 가장 왼쪽
---

# _pages/projects.md
---
nav_order: 2
---

# _pages/blog.md
---
nav_order: 3
---

# _pages/cv.md
---
nav_order: 4
---

# _pages/publications.md (논문이 있다면)
---
nav_order: 5
---
```

**메뉴에서 제거하기**:
```yaml
# 예: teaching 페이지를 메뉴에서 숨기기
---
nav: false  # true를 false로 변경
---
```

---

### 🎨 Nice to Have (여유 있을 때)

#### 7. 블로그 포스트 추가

**위치**: `_posts/`

**파일명 규칙**: `YYYY-MM-DD-title-with-hyphens.md`

**템플릿**:

```markdown
---
layout: post
title: "제목은 여기에"
date: 2024-12-31 10:00:00
description: 짧은 요약 (1-2줄)
tags: robotics ai control planning
categories: research  # 또는 engineering, tutorial
giscus_comments: true  # 댓글 기능 (선택)
---

## 서론

블로그 내용 시작...

## 본론

### 소제목 1

내용...

### 소제목 2

#### 코드 예시

```python
def example_function():
    print("Hello, World!")
```

#### 이미지 추가

![설명]({{ '/assets/img/blog/image.png' | relative_url }})

또는 (al-folio 전용):

{% include figure.liquid path="assets/img/blog/image.png" class="img-fluid rounded z-depth-1" %}

#### 수식 (LaTeX)

인라인 수식: $x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$

블록 수식:

$$
\mathbf{x}_{t+1} = f(\mathbf{x}_t, \mathbf{u}_t)
$$

#### 표

| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |

## 결론

마무리...

```

**블로그 아이디어**:
1. "역기구학 구현: 이론부터 실전까지"
2. "MuJoCo 시뮬레이션 시작하기"
3. "ROS2와 Docker를 사용한 로봇 개발 환경 구축"
4. "Semantic Segmentation 모델 학습 A-Z"
5. "강화학습 Reward Engineering 실전 팁"

---

#### 8. Publications 추가 (논문이 있다면)

**위치**: `_bibliography/papers.bib`

**BibTeX 형식으로 작성**:

```bibtex
@article{shin2024sidewalker,
  abbr={Preprint},
  title={SideWalker: Semantic Navigation for Urban Sidewalks with BEV-MPPI},
  author={Shin, Joonhyun and Others},
  journal={arXiv preprint arXiv:2401.XXXXX},
  year={2024},
  url={https://arxiv.org/abs/2401.XXXXX},
  html={https://project-page.com},
  pdf={sidewalker.pdf},
  code={https://github.com/blackuniverse0u0/perception_planning_control},
  abstract={We present SideWalker, an autonomous navigation system...},
  preview={sidewalker_preview.png},
  selected={true}  # 중요한 논문은 true
}
```

**PDF 추가**:
```bash
# PDF를 assets/pdf/ 폴더에 추가
cp /path/to/paper.pdf assets/pdf/sidewalker.pdf
```

**프리뷰 이미지 추가**:
```bash
# 논문 썸네일을 assets/img/publication_preview/ 에 추가
cp /path/to/thumbnail.png assets/img/publication_preview/sidewalker_preview.png
```

---

## 상세 업데이트 가이드

### A. 프로필 사진 변경

**단계별**:

1. **사진 준비**
   - 정방형 (1:1 비율) 권장
   - 최소 400x400px, 권장 800x800px
   - JPG 또는 PNG

2. **파일 배치**
   ```bash
   # 현재 디렉토리 확인
   pwd  # .../blackuniverse0u0.github.io

   # 사진 복사
   cp ~/Pictures/my_photo.jpg assets/img/prof_pic.jpg
   ```

3. **원형으로 만들기 (선택)**

   `_pages/about.md`에서:
   ```yaml
   profile:
     image_circular: true  # false를 true로 변경
   ```

4. **커밋**
   ```bash
   git add assets/img/prof_pic.jpg _pages/about.md
   git commit -m "Update profile picture"
   git push
   ```

---

### B. 소셜 미디어 아이콘 활성화

**_config.yml 수정 후 확인**:

```bash
# 로컬에서 테스트
bundle exec jekyll serve

# 브라우저에서 http://localhost:4000 접속
# About 페이지 하단에 소셜 아이콘이 표시되는지 확인
```

**아이콘이 안 나타난다면**:

`_pages/about.md` 확인:
```yaml
---
social: true  # 이게 true인지 확인
---
```

---

### C. 프로젝트 카테고리 추가

**현재**: `display_categories: [robotics]`

**여러 카테고리로 분류하고 싶다면**:

1. **_pages/projects.md 수정**:
   ```yaml
   display_categories: [perception, locomotion, deployment]
   ```

2. **각 프로젝트 파일의 category 수정**:
   ```yaml
   # _projects/sidewalker.md
   category: perception

   # _projects/locomotion_rl.md
   category: locomotion

   # _projects/gremsy_box.md
   category: deployment
   ```

**효과**: 프로젝트 페이지에서 카테고리별로 그룹화되어 표시됩니다.

---

## Jekyll/Liquid 문법 기초

### 1. Front Matter (YAML)

모든 페이지/포스트의 맨 위에 `---`로 감싸진 메타데이터:

```yaml
---
layout: post       # 사용할 레이아웃
title: 제목
date: 2024-12-31
tags: [tag1, tag2]  # 배열은 []로
key: value         # 단일 값
---

여기부터 본문 시작
```

### 2. 변수 출력

```liquid
{{ variable }}           # 변수 출력
{{ "문자열" }}          # 문자열 출력
{{ site.title }}        # _config.yml의 값
{{ page.title }}        # 현재 페이지의 front matter 값
```

### 3. 필터

```liquid
{{ "hello" | capitalize }}              # "Hello"
{{ "2024-12-31" | date: "%Y년 %m월" }}  # "2024년 12월"
{{ "/assets/img/pic.png" | relative_url }}  # 상대 경로로 변환
```

### 4. 조건문

```liquid
{% if page.image %}
  <img src="{{ page.image }}">
{% else %}
  <img src="default.png">
{% endif %}
```

### 5. 반복문

```liquid
{% for post in site.posts %}
  <h2>{{ post.title }}</h2>
{% endfor %}
```

### 6. 주석

```liquid
{% comment %}
이 부분은 렌더링되지 않음
{% endcomment %}
```

### 7. Include (재사용 가능한 조각)

```liquid
{% include figure.liquid
   path="assets/img/pic.png"
   title="설명"
   class="img-fluid" %}
```

---

## 로컬 테스트 방법

### 1. Jekyll 서버 실행

```bash
# 터미널에서 프로젝트 디렉토리로 이동
cd ~/blackuniverse0u0.github.io

# Jekyll 서버 시작
bundle exec jekyll serve

# 출력 예시:
# Server address: http://127.0.0.1:4000/
# Server running... press ctrl-c to stop.
```

### 2. 브라우저에서 확인

```
http://localhost:4000
```

**실시간 업데이트**:
- 파일을 수정하고 저장하면 자동으로 재빌드됩니다
- 브라우저를 새로고침하면 변경사항이 반영됩니다

**예외**: `_config.yml` 수정 시에는 서버 재시작 필요:
```bash
# Ctrl+C로 서버 중지
# 다시 시작
bundle exec jekyll serve
```

### 3. 빌드 에러 확인

에러가 발생하면 터미널에 표시됩니다:

```
Liquid Exception: Liquid syntax error...
```

**자주 발생하는 에러**:
- YAML front matter 문법 오류 (`:` 뒤에 공백 필요)
- Liquid 태그 닫기 누락 (`{% endif %}` 등)
- 잘못된 날짜 형식 (YYYY-MM-DD HH:MM:SS)

---

## 파일 구조 빠른 참조

```
blackuniverse0u0.github.io/
│
├── _config.yml              # 사이트 전역 설정 ⭐
│
├── _pages/                  # 주요 페이지 ⭐
│   ├── about.md            # 홈페이지
│   ├── projects.md         # 프로젝트 목록
│   ├── cv.md               # 이력서
│   └── blog.md             # 블로그 목록
│
├── _projects/               # 프로젝트 마크다운 ⭐
│   ├── sidewalker.md
│   ├── locomotion_rl.md
│   └── gremsy_box.md
│
├── _posts/                  # 블로그 포스트 ⭐
│   ├── 2024-12-15-bev-mppi-path-planning.md
│   └── 2024-11-20-quadruped-rl-from-scratch.md
│
├── _news/                   # 뉴스/공지 ⭐
│   ├── announcement_1.md
│   └── announcement_2.md
│
├── assets/
│   ├── img/                 # 이미지 ⭐
│   │   ├── prof_pic.jpg    # 프로필 사진
│   │   ├── sidewalker.png  # 프로젝트 이미지
│   │   └── blog/           # 블로그 이미지
│   │
│   ├── json/                # JSON 데이터
│   │   └── resume.json     # CV 데이터 ⭐
│   │
│   └── pdf/                 # PDF 파일
│       └── papers/          # 논문 PDF
│
├── _bibliography/           # 출판물
│   └── papers.bib          # BibTeX 파일
│
└── _data/                   # 기타 데이터
    └── repositories.yml    # GitHub 리포지토리 목록
```

---

## 체크리스트

작업 완료 후 아래 체크리스트로 확인:

### 기본 정보
- [ ] 프로필 사진 업데이트 (`assets/img/prof_pic.jpg`)
- [ ] 소셜 미디어 링크 추가 (`_config.yml`)
- [ ] Resume.json 개인정보 업데이트
- [ ] About 페이지 내용 확인

### 콘텐츠
- [ ] 최소 3개의 뉴스/공지사항 (`_news/`)
- [ ] 프로젝트 이미지 확인 (실제 스크린샷으로 교체)
- [ ] 블로그 포스트 3개 이상
- [ ] CV 정보 완성

### 네비게이션
- [ ] 메뉴 순서 확인
- [ ] 불필요한 페이지 숨김 처리
- [ ] 모든 링크 작동 확인

### 테스트
- [ ] 로컬에서 빌드 성공 (`bundle exec jekyll serve`)
- [ ] 모바일 레이아웃 확인
- [ ] 모든 이미지 로드 확인
- [ ] GitHub Actions 빌드 성공

---

## 다음 단계

1. **우선순위 높은 항목부터 하나씩 처리**
   - 프로필 사진
   - 소셜 링크
   - Resume.json

2. **로컬에서 테스트**
   ```bash
   bundle exec jekyll serve
   ```

3. **문제없으면 커밋 & 푸시**
   ```bash
   git add .
   git commit -m "Update profile and resume"
   git push
   ```

4. **GitHub에서 빌드 확인**
   - https://github.com/blackuniverse0u0/blackuniverse0u0.github.io/actions

5. **배포 확인**
   - https://blackuniverse0u0.github.io

---

## 도움말

### 자주 하는 실수

1. **YAML 문법 오류**
   ```yaml
   # 잘못된 예
   title:My Title  # ❌ 콜론 뒤에 공백 필요

   # 올바른 예
   title: My Title  # ✅
   ```

2. **날짜 형식**
   ```yaml
   # 잘못된 예
   date: 2024/12/31  # ❌

   # 올바른 예
   date: 2024-12-31 10:00:00  # ✅
   ```

3. **이미지 경로**
   ```markdown
   <!-- 잘못된 예 -->
   ![Image](assets/img/pic.png)  <!-- ❌ 슬래시 누락 -->

   <!-- 올바른 예 -->
   ![Image](/assets/img/pic.png)  <!-- ✅ 맨 앞에 / -->
   <!-- 또는 -->
   ![Image]({{ '/assets/img/pic.png' | relative_url }})  <!-- ✅ Liquid -->
   ```

### 추가 리소스

- **al-folio 공식 문서**: https://github.com/alshedivat/al-folio
- **Jekyll 문서**: https://jekyllrb.com/docs/
- **Liquid 문법**: https://shopify.github.io/liquid/
- **Markdown 가이드**: https://www.markdownguide.org/

### 질문이 있을 때

1. 로컬에서 먼저 테스트
2. 에러 메시지 전체 복사
3. GitHub Issues 검색: https://github.com/alshedivat/al-folio/issues

---

**마지막 업데이트**: 2024-12-31
**작성자**: Claude Code

