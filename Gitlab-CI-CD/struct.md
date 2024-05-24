# Cấu trúc chung của một file `.gitlab-ci.yml`

Nhìn chung cấu trúc của    `.gitlab-ci.yml` giống với một file `docker compose`. Nó cũng là file yml

1. Ví dụ file `.gitlab-ci.yml`

```yml
image: docker:24.0.7

services:
  - docker:24.0.7-dind

stages:
  - build
  - test
  - release

before_script:
  - docker version
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

build:
  stage: build
  script:
    - docker pull $CI_REGISTRY_IMAGE:latest || true
    - docker build --cache-from $CI_REGISTRY_IMAGE:latest --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  before_script:
    - docker compose version
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
  script:
    - docker compose up -d
    - sleep 15
    - docker compose exec -T app npm run test

release:
  variables:
    GIT_STRATEGY: none
  stage: release
  script:
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
```

## Cấu trúc và ý nghĩa của file `gitlab-ci.yml`

### 1. Định nghĩa version docker.
```yml
image: docker:24.0.7
```
Phải định nghĩa rõ version để hệ thống hoạt động ổn định, tuyệt đối không dùng tag là latest vì lỡ nhiều năm về sau version thay đổi lại không tương thích với hệ thống
### 2. Services:
```yml
services:
  - docker:24.0.7-dind
```
`service docker:24.0.7` -> đây là môi trường docker trong image docker:24.0.7 ở trên nó dùng để thiết lập môi trường chạy các lệnh docker trong  `image: docker:24.0.7`.
### 3. Stages
```yml
stages:
  - build
  - test
  - release
```  
Các stages cần thực hiện, nó được thực hiện tuần tự

###  4. `before_script`
```yml
before_script:
  - docker version
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
```
Đây là các `script` sẽ được chạy trước khi thực hiện các `stage`. Có thể định nghĩa `before_script` riêng cho từ stage. Nếu được đặt ở ngoài như trường hợp này nghĩa là nó được áp dụng cho tất cả các `stage`. Khi viết `before_script` riêng ta có thể ghi đè lại các `before_script` chung.
    - `- docker version` câu lệnh để lấy version docker
    - `- docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY`: lệnh để login vào `registry`. 

`$CI_REGISTRY_USER`, `$CI_REGISTRY_PASSWORD`, `$CI_REGISTRY` là các variable của `gitlab CI-CD`
    
`-u $CI_REGISTRY_USER` là tùy chọn tên user
`-p $CI_REGISTRY_PASSWORD` là tùy chọn nhập mật khẩu
`$CI_REGISTRY` là tên của registry của gitlab là: `registry.gitlab.com`

### 5. Các `stage`
Trong ví dụ có 3 stage được thực hiện tuần tự gồm: `build`, `test`, `release`. Thay vì việc phải build, test thủ công bằn các câu lệnh thì ta sẽ thực hiện gộp các dòng lệnh vào và ném cho CI-CD của git lab thực hiện.
#### 5.1 build
```yml
build:
  stage: build
  script:
    - docker pull $CI_REGISTRY_IMAGE:latest || true
    - docker build --cache-from $CI_REGISTRY_IMAGE:latest --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```
Các lệnh trên bao gồm:
    1. pull `image` của dụ án (là tag `latest` mới nhất)
    2. build `image` vừa pull và tạo 1 `tag` khác cho `image` đó `$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA ` nghĩa là sẽ tạo một tag của image với tên tag trùng với `commit hash id` của commit mới nhất.
    3. Push `image` `$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA` lên registry
- Việc tạo ra một image với tag `$CI_COMMIT_SHA` là không bắt buộc nhưng nó giúp chúng ta dễ dàng lưu lại các image cũ để lỡ may một ngày đẹp trời hệ thống sập mình còn có image cũ để deploy back về version cũ.

#### 5.2 test
```yml 
test:
  stage: test
  before_script:
    - docker compose version
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
  script:
    - docker compose up -d
    - sleep 15
    - docker compose exec -T app npm run test
```
Đây là bước kiểm tra `test` trước khi chuyển sang bước `release` nếu pass test.
`before_script` ở đây sẽ ghi đè `before_script` ở trước đó.
Vì khi khởi động test ta cần connect đến db, tạo dựng môi trường nên ta cần phải `sleep 15` nghĩa là đợi 15s rồi mới chạy script tiếp theo


#### 5.3 release
```yml
release:
  variables:
    GIT_STRATEGY: none
  stage: release
  script:
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
```
ta có khai báo biến môi trường `$GIT_STRATEGY` với giá trị none, ý bảo là không cần clone source code vào bên trong `Gitlab Runner`

[Bài này tham khảo của](https://viblo.asia/p/automation-test-voi-docker-va-gitlab-ci-yMnKMv2DZ7P). Anh này có series khá hay về  `Docker, CICD`