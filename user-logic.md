## (controller - UserController)
### 크리에이터 권한 신청 및 승격
- `@AuthenticationPrincipal UserDetails userDetails` 로그인된 사용자를 인증함
- Mapping을 하는 4가지 방법
| 메서드 | 의미 |
|------|------|
| GET | 조회 |
| POST | 새로 생성 |
| PUT | 전체 수정 |
| PATCH | 부분 수정 |
  - `PatchMapping("/me/creator")`
    - User 전체를 바꾸는 게 아니라 role(권한)만 변경하기 때문에 Patch (부분수정)
    - `"/me/creator"`는 로그인한 자신, 바꾸려는 상태 => "나를 크리에이터 상태로 바꾼다"
- `@AuthenticationPrincipal UserDetails userDetails` = "이 요청을 보낸 로그인된 사용자의 정보를 자동으로 주입해"
    - Spring Security가 토큰 검사 / 로그인 여부 확인 / 인증 객체 생성 후에 userDetails에 넣어줌

### 내 프로필 정보 조회 / 수정
- 둘은 비슷한 뼈대를 가지고 있다
- (getMyProfile) 조회는 get `UserProfileResponse response = userService.getMyProfile(userDetails.getUsername());`
- (updateProfile) 수정은 patch `@RequestBody UpdateProfileRequest request`

### 
