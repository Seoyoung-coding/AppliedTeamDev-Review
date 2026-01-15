> 해야할일 목록 : contentTest.http로 시험해본다

앞으로 할것들

이부분에 대해서 고민해보기.... 구매내역이 저장되어있는 db를 찾아서 ...어디에 저장되는지 맞는 repository찾아서 { boolean hasPurchased = orderService.hasPurchased(userId, contentId); //유료서비스 return contentService.getContentWithAccessControl(id, hasPurchased); } 이거 없이 db에서 불러오기 => 이런ㄴ식으로 고치기
    @GetMapping("/{userId}")
    public Object get(@PathVariable Long userId) {
        boolean hasPurchased = orderService.hasPurchased(userId, contentId); //유료서비스
        return contentService.getContentWithAccessControl(id, hasPurchased);
    }
create는 테스트했고, 실전용으로 (임시 아이디가 아니라 principalId가 되도록)

그리고 다른거

long부분, 다시 자기페이지로 돌아올수있기 @PostMapping public ResponseEntity create(@RequestBody ContentCreateRequest req) { Long id = contentService.createContent(1L, req); // 임시 userId return ResponseEntity.status(HttpStatus.CREATED).body(id); }

get부분 @GetMapping("/{userId}") public Object get(@PathVariable Long userId) { boolean hasPurchased = orderService.hasPurchased(userId, contentId); //유료서비스 return contentService.getContentWithAccessControl(id, hasPurchased); } //제작자랑 어드민은 자기꺼보게

무료의 경우 가격정보, 할인정보는 보여줄필요 없음 -> null로 보여주겠지 -> private int price; private int discountRate; 이부분 null 값처리(예외처리?) 이 외에 null 값을 나올거 같은 경우를 찾아서 처리해야함





✅ C-01 콘텐츠 발행

콘텐츠를 생성하고 게시하는 기능

크리에이터가 제목 / 내용 / 가격(10원 단위) / 할인율 / 무료 여부를 설정하여 콘텐츠를 생성

콘텐츠는 생성 시 INACTIVE(비공개) 상태

게시(Publish) 시 ACTIVE(공개) 상태로 전환

공개된 콘텐츠만 조회 가능

콘텐츠 생성 시간 / 수정 시간 자동 관리

👉 즉,
“콘텐츠 하나를 DB에 만들고, 공개 상태로 전환해서 사용자에게 보여줄 수 있는 기능”

✅ C-02 접근 제어 (유료 / 무료 콘텐츠)

콘텐츠 접근 권한에 따른 노출 제어 기능

무료 콘텐츠

로그인 / 구매 여부와 상관없이 전체 내용 조회 가능

유료 콘텐츠

구매하지 않은 사용자 → 미리보기만 노출

콘텐츠 일부만 반환

구매한 사용자 → 전체 내용 조회 가능

접근 제어 판단은 Service에서 수행

Controller는 단순히 “구매 여부(boolean)”만 전달

👉 즉,
“돈 안 낸 사람은 미리보기만, 돈 낸 사람은 전체 콘텐츠를 볼 수 있는 구조”

🟡 C-03 관심 / 추천 (북마크)

사용자가 콘텐츠를 ‘찜’하고 관리하는 기능

구현된 기능

사용자가 콘텐츠를 북마크(찜)

동일 콘텐츠 중복 북마크 방지

사용자가 북마크한 콘텐츠 해제 가능

아직 남은 기능

내가 북마크한 콘텐츠 목록 조회

👉 즉,
“이 콘텐츠 마음에 들어서 저장해두는 기능 (즐겨찾기)”

✅ C-04 조회수

콘텐츠 조회 시 조회수를 기록하는 기능

콘텐츠 상세 조회 시마다 조회수 +1 증가

무료 / 유료 구분 없이 조회하면 증가

조회수는 콘텐츠 자체 속성으로 관리

👉 즉,
“사람들이 이 콘텐츠를 몇 번 봤는지 세는 기능”
