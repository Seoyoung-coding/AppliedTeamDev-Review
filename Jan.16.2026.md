해야 할 일

## 1. 콘텐츠 조회 로직 정리 (구매 여부 판단 방식 개선)
1. 구매 내역이 저장된 테이블 확인 (orders, payments, purchases 등 실제 엔티티 확인)

2. 정확한 Repository 찾기
예: OrderRepository.existsByUserIdAndContentId(...)

3. 구매 여부 판단 책임을 Service로 이동
- Controller는 userId, contentId만 전달
- Service 내부에서 구매 여부 조회

## 2. Create API: 임시 userId → 실전용 principalId 전환

## 3. Create 이후 “자기 페이지로 돌아가기” 처리

## 4. Get API 접근 제어 규칙 명확화
요구 사항
- 제작자: 항상 전체 내용 조회 가능
- 관리자: 항상 전체 내용 조회 가능
- 일반 사용자: 구매 → 전체
미구매 → 미리보기만

## 5. 무료 콘텐츠의 가격/할인 정보 처리
현재 문제
private int price;
private int discountRate;
= 무료 콘텐츠인데 의미 없는 값 노출 가능

int → null 불가

## 6. 유료 콘텐츠 미구매 시 “미리보기 처리”
현재 문제
유료 콘텐츠 전체를 가져온 뒤 나눠야 하는 구조

고민 포인트
미리보기 전용 컬럼
previewContent
본문 일부 substring
비추천 (유지보수/보안 문제)

## 7. Global Exception 정리
해야 할 일
@ControllerAdvice
예외 타입 분리
ContentNotFound
AccessDenied
AlreadyPurchased
HTTP 상태 코드 명확화

## 8. 가격 변동 알림 기능 (추후 기능)
개념 정리
가격 변경 시:
북마크/관심 사용자 조회
알림 발송 (이메일 / 푸시 / 내부 알림)
선행 작업
가격 변경 이력 테이블
관심 콘텐츠 테이블

## 9. Soft Delete / Hard Delete 기준 정하기


소프트삭제의 경우 밴이랑 inactive나눠야함, 활성화시키면 active, 이넘타입에 softdeleted 하나 추가해서 리팩토링하기
