# 캐시와 검증 헤더

## 1. 캐시 
### 목적 
- 캐시는 웹 리소스를 로컬에 저장하여 빠르게 접근할 수 있게 하는 기술
- 이를 통해 네트워크 트래픽을 줄이고, 로딩 시간을 단축

### 동작 방식
- 클라이언트가 웹 리소스를 요청할 때, 캐시된 버전이 있는지 먼저 확인
- 캐시된 버전이 최신이면, 서버에 다시 요청할 필요 없이 즉시 해당 리소스를 제공

### 캐시가 없을 떄
1. 클라이언트가 서버에 이미지를 요청
2. 서버는 이미지의 HTTP 헤더(0.1M) + 바디(1.0M)를 합쳐 1.1M정도 용량의 데이터를 응답
3. 클라이언트는 이미지 응답을 받아 사용
4. 클라이언트가 이미지를 또 요청하면 1~3번 반복
- #### 문제점
  - 동일한 이미지를 요청하는데 네트워크를 통해 같은 데이터를 또 다운받는다
  - 용량이 클 수록 비용이 커지고 브라우저의 로딩속도가 느려진다
    - 인터넷 네트워크는 매우 느리고 비쌈

### 캐시 적용
1. 클라이언트가 서버에 이미지를 요청
2. 서버가 헤더에 `cache-control:max-age=60` 속성을 넣어주어 이미지를 응답
3. 클라이언트가 이미지 응답을 받아 사용
4. 클라이언트가 다시 요청할 떄, 캐시를 조회
5. 캐시가 존재하는 60초 이내의 경우 해당 캐시에서 자료를 가져옴
- #### 장점
  - 캐시 가능 시간동안 네트워크를 사용하지 않아도 됨
  - 비싼 네트워크 사용량을 줄일수 있음
  - 브라우저 로딩 속도가 매우 빠름
  
---

## 2. 검증 헤더와 조건부 요청
### 캐시 시간 초과
- 캐시 유효 시간이 초과해서 서버에 다시 요청
  1. 서버 데이터가 변경 됨
  2. 데이터가 변경 되지 않고 기존과 같음

### 검증 헤더
- 캐시 만료후에 서버 데이터가 변경되지 않았다면 저장했던 캐시를 재사용 가능
- 이 때, 클라이언트/서버의 데이터가 바뀌지않고 같다는것을 확일 할 때 사용하는 것이 검증 헤더

</br>
</br>

## Last-Modified 와 if-modified-since
#### 동작 방식
- 클라이언트가 서버에 데이터 요청
- 서버가 `Last-Modified: 2023년 11월 16일 10:00:00`를 헤더에 추가해서 데이터 응답
- 클라이언트가 응답 결과를 캐시에 저장할 때 데이터 최종 수정일도 같이 저장
- 캐시 만료 후, 다시 요청할 때 `if-modified-since: 2023년 11월 16일 10:00:00` 를 요청헤더에 담아서 서버에 요청
- 서버에서 상태코드는 `304 Not Modified`로 변경된것이 없다는 것을 알림
  - Body가 없으므로 헤더만 전송해서 네트워크 비용 부담이 줄어듬
- 데이터가 변경 되었다면 `200 OK` 와 새로운 `Last-Modified` 그리고 Body를 함께 보냄

#### 단점
- 1초 미만 단위 캐시 조정 불가
- 날짜 기반 로직 사용
- 데이터를 수정해 날짜가 다르지만, 같은 데이터를 수정해 데이터 결과가 똑같은 경우
    - test.txt 파일의 내용을 A → B로 수정했지만, 다시 B → A로 수정한 경우
- 서버에서 별도의 캐시 로직을 관리하고 싶은 경우
    - 스페이스나 주석처럼 크게 영향이 없는 변경에서 캐시를 유지하고 싶은 경우 

</br>
</br>

## ETag 와 If-None-Match
서버에서 완전히 캐시를 컨트롤하고 싶은 경우 `ETag` 를 사용하면 된다

**ETag (Entity Tag)**
- `ETag: "v1.0"`, `ETag: "kmskmskms"`
- 데이터가 변경되면 이 이름을 바꾸어서 변경한다 (Hash를 다시 생성)
- `ETag:"aaaa"` → `ETag:"bbbb"`
- 단순하게 `ETag`만 보내서 같으면 유지하고 바르면 다시 받는다 

#### 동작 방식
- 클라이언트가 서버에 데이터 요청
- 서버가 헤더에 `ETag`를 추가해서 응답
- 클라이언트에 `ETag`도 함게 캐시에 저장
- 캐시 만료후 `If-None-Match:aaaa` 를 요청 헤더에 작성해서 요청
- 서버에서 데이터가 변경되지 않았을 경우 `ETag`는 동일, `If-None-Match`는 실패로 `304 Not Modified`를 응답
  - 이 때, Body는 없다
- 클라이언트는 응답 결과를 재사용하고 헤더 데이터를 갱신해 캐시에 저장