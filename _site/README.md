# SanghyeungLee.github.io

## 로컬에서 실행하기

### 1. 의존성 설치

```bash
bundle install
```

만약 권한 문제가 발생하면:
```bash
bundle install --path vendor/bundle
```

### 2. 로컬 서버 실행

```bash
bundle exec jekyll serve
```

또는 Jekyll이 전역으로 설치되어 있다면:
```bash
jekyll serve
```

### 3. 브라우저에서 확인

서버가 실행되면 다음 주소에서 사이트를 확인할 수 있습니다:
- http://localhost:4000

### 참고사항

- Jekyll이 설치되어 있지 않다면 먼저 설치해야 합니다:
  ```bash
  gem install jekyll bundler
  ```
- 파일을 수정하면 자동으로 새로고침됩니다 (LiveReload)
- 서버를 중지하려면 터미널에서 `Ctrl+C`를 누르세요