## 로컬 테스트 브랜치를 만들어서 포스트맨으로 작동 시켜봤다

## 쉽게 지우는법
전체 검색/치환 열기
macOS: Cmd + Shift + R

##
    @Builder.Default
    private LocalDateTime createdAt = LocalDateTime.now();

    ?     private Long likeCount = 0L; 만 쓰면 안되는 이유?
    왜     @Builder.Default
    private Long likeCount = 0L; 로 써야함?

## 프린시퍼

    @PostMapping("/{id}/bookmark")
    public ResponseEntity<Void> bookmark(@AuthenticationPrincipal PrincipalDetails userDetails, @PathVariable Long id) {
        Long userId = userDetails.getUserId();
        bookmarkService.bookmark(userId, id);
        return ResponseEntity.ok().build();

```
    }
    @DeleteMapping("/{contentId}")
    public ResponseEntity<Void> delete(
            @AuthenticationPrincipal PrincipalDetails user,
            @PathVariable Long contentId
    )
```

## 몰랐던거
그 브랜치로 가서 pull을 한 다음에 merge를 해야 한다
나의 경우 그냥 냅다 머지함

## 고쳐줌
널이오면 안돼니까
    @Column(nullable = false)
    private String title;
    @Column(nullable = false)
    private String content;
    @Column(nullable = false)
    private int price;
    @Column(nullable = false)
    private int discountRate;
    @Column(nullable = false)
    private boolean isFree;

import popeye.popeyebackend.content.repository.ContentLikeRepository; 오류 고치기
= 패키지부분이 이상했었음

라이크레파지토리:
        if (alreadyLiked) {
            likeRepository.deleteByUserAndContent(user, content);
            content.decreaseLike();
        } else {
            ContentLike contentLike = ContentLike.builder()
                    .user(user)
                    .content(content).build();

            likeRepository.save(contentLike);
            content.increaseLike();
        }

프린시펄 디테일로 바꿈
    @PostMapping("/{id}/bookmark")
    public ResponseEntity<Void> bookmark(@AuthenticationPrincipal PrincipalDetails details, @PathVariable Long id) {
        Long userId = details.getUserId();
        bookmarkService.bookmark(userId, id);
        return ResponseEntity.ok().build();
    }

##
contentTest.http로 시험해본다


앞으로 할것들
1. 이부분에 대해서 고민해보기.... 구매내역이 저장되어있는 db를 찾아서 ...어디에 저장되는지 맞는 repository찾아서 {
        boolean hasPurchased = orderService.hasPurchased(userId, contentId); //유료서비스
        return contentService.getContentWithAccessControl(id, hasPurchased);
    }
이거 없이 db에서 불러오기 => 이런ㄴ식으로 고치기
```
    @GetMapping("/{userId}")
    public Object get(@PathVariable Long userId) {
        boolean hasPurchased = orderService.hasPurchased(userId, contentId); //유료서비스
        return contentService.getContentWithAccessControl(id, hasPurchased);
    }
```


2. create는 테스트했고, 실전용으로 (임시 아이디가 아니라 principalId가 되도록)
```

```
그리고 다른거

3.
long부분, 다시 자기페이지로 돌아올수있기
    @PostMapping
    public ResponseEntity<Long> create(@RequestBody ContentCreateRequest req) {
        Long id = contentService.createContent(1L, req); // 임시 userId
        return ResponseEntity.status(HttpStatus.CREATED).body(id);
    }

get부분
    @GetMapping("/{userId}")
    public Object get(@PathVariable Long userId) {
        boolean hasPurchased = orderService.hasPurchased(userId, contentId); //유료서비스
        return contentService.getContentWithAccessControl(id, hasPurchased);
    } //제작자랑 어드민은 자기꺼보게

4. 무료의 경우 가격정보, 할인정보는 보여줄필요 없음 -> null로 보여주겠지 ->     private int price;
    private int discountRate; 이부분 null 값처리(예외처리?)
   이 외에 null 값을 나올거 같은 경우를 찾아서 처리해야함

5. 
