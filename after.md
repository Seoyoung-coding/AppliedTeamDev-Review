## exceptions
discount, price 부분

## 빌드 패턴

## 유료 서비스
“이 유저가 이 콘텐츠를 구매했는가?”


## service내에서와 exception처리의 차이점
Service에서 예외를 던지는 기준은 “이게 비즈니스 규칙 위반이냐, 아니면 시스템 실패냐”다.
예를들어 User 조회 실패의 경우, 비즈니스 규칙 위반이 아니라 시스템/요청 오류다.
```
User user = userRepository.findById(userId)
        .orElseThrow();
```

## contentController
```
@GetMapping("/{id}")
public ContentResponse read(@PathVariable Long id) {
return contentService.getContent(id);}
```
에서 read() 부분을 고쳐야 하는 이유는 접근 제어 없는 단순 조회 전용이기 때문이다.
```
 @GetMapping("/{id}")
    public Object get(@PathVariable Long id) {
        boolean hasPurchased = paymentService.hasPurchased(userId, contentId); //결제부분
        return contentService.getContentWithAccessControl(id, hasPurchased);
    }
```
get()으로 받아

## 북마크 목록 조회기능 만들기
북마크 엔티티에는 이 3개만 있으면 충분: user / content / createdAt

##
