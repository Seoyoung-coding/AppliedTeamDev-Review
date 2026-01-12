1. cicd 설정?
2. 

## Content entity는 상태+행동을 나타낸다
1. 상태 매서드를 추가했다
```

```
## ContentCreateRequest (요청 DTO)
사용자가 콘텐츠를 생성할 때 클라이언트 → 서버로 보내는 데이터

## ContentResponse (응답 DTO)
서버가 사용자에게 돌려주는 응답 전용 객체

## 순서
Content 엔티티
: 필드 / 생성 메서드 (create) / 상태 변경 (activate / deactivate) / 조회수 증가 (increaseViewCount)

ContentRepository
: findByIdAndStatus / findAllByStatus

ContentService
: 콘텐츠 생성 / 콘텐츠 단건 조회

ContentController
: POST /contents / GET /contents/{id}


## 유저랑 컨텐츠 가 입력이 필요한데 create 말고 @builder 로 하자

@Builder
public ContentBookmark(User user, Content content) {
   this.user = user;
   this.content = content;
}


bookmarkRepository.save(ContentBookmark.builder().user(user).content(content));


public void bookmark(Long userId, Long contentId) {
   User user = userRepository.findById(userId)
           .orElseThrow();
   Content content = contentRepository.findById(contentId)
           .orElseThrow(() -> new ContentError("컨텐츠를 찾을 수 없습니다."));


## 필드 추가 
private Integer price;

   @Builder
   public ContentBookmark(User user, Content content, Integer price) {
       this.user = user;
       this.content = content;
       this.price = price;
   }
}


ContentBookmark contentBookmark = ContentBookmark.builder()
       .user(user)
       .content(content)
       .price(price).build();


bookmarkRepository.save(contentBookmark);


## 컨트롤러 부분도 수정
@PostMapping("/{id}/bookmark")
public ResponseEntity<Void> bookmark(@AuthenticationPrincipal UserDetails userDetails, @PathVariable Long id) {
   Long userId = userDetails.getUserId();
   bookmarkService.bookmark(1L, id);
   return ResponseEntity.ok().build();
}
생성자 위에 build쓸수있다

## +
contentbookmarkdto는 필요없이 price 정보를 받아오면 된다? 
ContentBookmark contentBookmark = ContentBookmark.builder()
       .user(user)
       .content(content)
       .price(content.getPrice).build();


## 여기 추가하기 


public void unbookmark(Long userId, Long contentId) {
   User user = userRepository.findById(userId).orElseThrow();
   Content content = contentRepository.findById(contentId).orElseThrow();

public void increaseViewCount(){
   this.viewCount++;
}
