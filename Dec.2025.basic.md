인수조건 맞추기
auth 로 바꾸기
질의 응답 게시판 개발을 위한 대략적인 계획을 세웠음 도메인별 기능에 대해 세부사항/사용자 스토리/인수 조건을 나누어 작성함 인당 파트를 나눠 작성했으며 남은 부분은 빨리 해결한 사람이 맡아 채워넣음 다음은 오늘 활동에 대한 예시중 하나임

### Question (질문)

- 질문 작성
    - 사용자가 질문 작성 버튼을 통해 질문 작성 화면에 접근한다
    - 질문 제목과 내용을 입력할 수 있는 입력 영역을 확인한다
    - 질문 제목을 입력한다
    - 질문 내용을 입력한다
    - 등록 버튼을 누른다
    - 입력한 질문이 서버에 저장된다
    - 질문 목록에 새 질문이 등록된다

사용자 스토리
     - 사용자로서 궁금한 내용을 다른 사용자들에게 질문하기 위해, 질문을 작성하고 등록하고 싶다

인수 조건
    - [ ]  로그인하지 않은 사용자는 질문 작성 버튼이 보이지 않는다
    - [ ]  로그인한 사용자만 질문을 작성할 수 있다
    - [ ]  질문 제목이 비어 있으면 ‘제목이 비어있음’ 에러 메시지가 표시 된다
    - [ ]  질문 내용이 비어 있으면 ‘내용이 비어있음’ 에러 메시지가 표시 된다
    - [ ]  질문 등록 버튼 클릭 시 질문이 서버에 저장된다
    - [ ]  질문 등록 후 질문 목록에 새 질문이 즉시 반영된다
    - [ ]  질문 작성 완료 후 입력 폼이 초기화된다

1. 오류 해결
Domain
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id", nullable = false)
private User user;
=

@Builder
public Question(String title, String content, User user) {
    this.title = title;
    this.content = content;
    this.user = user;
    this.viewCount = 0;
}
=

@PrePersist
protected void onCreate() {
    this.createdAt = LocalDateTime.now();
}

@PreUpdate
protected void onUpdate() {
    this.updatedAt = LocalDateTime.now();
}
=

repository
public interface QuestionRepository extends JpaRepository<Question, Long> {
    // 특정 유저의 질문 목록을 페이징하여 조회
    Page<Question> findByUser_Id(Long userId, Pageable pageable);
}
=

controller
package com.example.qnaboard.controller.question;

import com.example.qnaboard.dto.question.request.QuestionCreateRequest;
import com.example.qnaboard.dto.question.request.QuestionUpdateRequest;
import com.example.qnaboard.dto.question.response.QuestionResponse;
import com.example.qnaboard.dto.user.response.MyQuestionSummaryResponse;
import com.example.qnaboard.security.CustomUserDetails;
import com.example.qnaboard.service.quesiton.QuestionService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/questions")
@RequiredArgsConstructor
public class QuestionController {

    private final QuestionService questionService;

    // 1. 등록
    @PostMapping
    public ResponseEntity<QuestionResponse> register(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @Valid @RequestBody QuestionCreateRequest request
    ) {
        QuestionResponse response =
                questionService.createQuestion(userDetails.getUserId(), request);

        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    // 2. 전체 목록 조회
    @GetMapping
    public Page<QuestionResponse> list(@PageableDefault Pageable page) {
        return questionService.getQuestionList(page);
    }

    // 3. 상세 조회
    @GetMapping("/{id}")
    public QuestionResponse getDetail(@PathVariable Long id) {
        return questionService.getQuestionDetail(id);
    }

    // 4. 수정
    @PutMapping("/{id}")
    public ResponseEntity<QuestionResponse> update(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @PathVariable Long id,
            @Valid @RequestBody QuestionUpdateRequest request
    ) {

        QuestionResponse updated = questionService.updateQuestion(userDetails.getUserId(), id, request);

        return ResponseEntity.ok(updated);
    }

    // 5. 삭제
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @PathVariable Long id
    ) {
        questionService.deleteQuestion(userDetails.getUserId(), id);
        return ResponseEntity.noContent().build();
    }

    // 6. 내가 쓴 글 조회
    @GetMapping("/userPost")
    public Page<MyQuestionSummaryResponse> getUserPost(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @PageableDefault Pageable page
    ) {
        return questionService.getMyQuestions(userDetails.getUserId(), page);
    }
}
service
package com.example.qnaboard.service.quesiton;

import com.example.qnaboard.domain.question.Question;
import com.example.qnaboard.domain.user.User;
import com.example.qnaboard.dto.question.request.QuestionCreateRequest;
import com.example.qnaboard.dto.question.request.QuestionUpdateRequest;
import com.example.qnaboard.dto.question.response.QuestionResponse;
import com.example.qnaboard.dto.user.response.MyQuestionSummaryResponse;
import com.example.qnaboard.exception.QuestionErrorCode;
import com.example.qnaboard.exception.UserErrorCode;
import com.example.qnaboard.exception.common.BusinessException;
import com.example.qnaboard.repository.question.QuestionRepository;
import com.example.qnaboard.repository.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional
public class QuestionService {

    private final QuestionRepository questionRepository;
    private final UserRepository userRepository;

    // 질문 등록
    public QuestionResponse createQuestion(
            Long userId,
            QuestionCreateRequest request
    ) {
        // 작성자 유저 확인
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new BusinessException(UserErrorCode.USER_NOT_FOUND));

        // 질문 엔티티 생성 및 빌드
        Question question = Question.builder()
                .title(request.title())
                .content(request.content())
                .user(user)
                .build();
        Question saved = questionRepository.save(question);

        return new QuestionResponse(
                saved.getId(),
                saved.getTitle(),
                saved.getContent(),
                saved.getCreatedAt(),
                new QuestionResponse.UserResponse(
                        user.getId(),
                        user.getUsername()
                )
        );
    }

    // 질문 전체 목록 조회
    @Transactional(readOnly = true)
    public Page<QuestionResponse> getQuestionList(Pageable pageable) {
        return questionRepository.findAll(pageable)
                .map(question -> new QuestionResponse(
                        question.getId(),
                        question.getTitle(),
                        question.getContent(),
                        question.getCreatedAt(),
                        new QuestionResponse.UserResponse(
                                question.getUser().getId(),
                                question.getUser().getUsername()
                        )
                ));
    }

    // 질문 상세 조회
    @Transactional
    public QuestionResponse getQuestionDetail(Long questionId) {
        Question question = questionRepository.findById(questionId)
                .orElseThrow(() -> new BusinessException(QuestionErrorCode.QUESTION_NOT_FOUND));

        return new QuestionResponse(
                question.getId(),
                question.getTitle(),
                question.getContent(),
                question.getCreatedAt(),
                new QuestionResponse.UserResponse(
                        question.getUser().getId(),
                        question.getUser().getUsername()
                )
        );
    }

    // 질문 수정
    public QuestionResponse updateQuestion(
            Long userId,
            Long questionId,
            QuestionUpdateRequest request
    ) {
        Question question = questionRepository.findById(questionId)
                .orElseThrow(() -> new BusinessException(QuestionErrorCode.QUESTION_NOT_FOUND));

        // 본인 소유 확인
        validateOwner(question, userId);
        question.update(request.title(), request.content());

        return new QuestionResponse(
                question.getId(),
                question.getTitle(),
                question.getContent(),
                question.getCreatedAt(),
                new QuestionResponse.UserResponse(
                        question.getUser().getId(),
                        question.getUser().getUsername()
                )
        );

    }

    // 질문 삭제
    public void deleteQuestion(
            Long userId,
            Long questionId
    ) {
        Question question = questionRepository.findById(questionId)
                .orElseThrow(() -> new BusinessException(QuestionErrorCode.QUESTION_NOT_FOUND));

        // 본인 소유 확인
        validateOwner(question, userId);
        questionRepository.delete(question);
    }

    // MyQuestion
    @Transactional(readOnly = true)
    public Page<MyQuestionSummaryResponse> getMyQuestions(Long userId, Pageable pageable) {
        return questionRepository.findByUser_Id(userId, pageable)
                .map(question -> new MyQuestionSummaryResponse(
                        question.getId(),
                        question.getTitle(),
                        question.getCreatedAt().toString()
                ));
    }

    // 검증
    private void validateOwner(Question question, Long userId) {
        // 엔티티의 유저 ID와 로그인한 유저 ID 비교
        if (!question.getUser().getId().equals(userId)) {
            throw new BusinessException(QuestionErrorCode.UNAUTHORIZED_USER);
        }
    }
}
2. 기능 추가
레파지토리에 추가 = 제목 내 키워드 포함된 질문 검색 Page<Question> findByTitleContainingOrContentContaining(String title, String content, Pageable pageable);
서비스에 추가 = 키워드로 검색하기
@Transactional(readOnly = true)
public Page<QuestionResponse> getQuestionList(String keyword, Pageable pageable) {
    Page<Question> questions;
    if (keyword != null && !keyword.isBlank()) {    // keyword가 비어있지 않으면 검색, 비어있으면 전체 조회
        questions = questionRepository.findByTitleContainingOrContentContaining(keyword, keyword, pageable);
    } else {
        questions = questionRepository.findAll(pageable);
    }
    return questions.map(this::convertToQuestionResponse);
}
정렬 시키는법 컨트롤러가 받은 Pageable 파라미터가 자동으로 처리함 인기순: 사용자가 /api/questions?sort=viewCount,desc로 요청하면 JPA가 내부적으로 ORDER BY viewCount DESC 답변 많은 순: Question 엔티티에 commentCount 필드를 추가했다면, 동일하게 ?sort=commentCount,desc로 요청 정렬용 필드 작성
@Column(nullable = false)
private int commentCount = 0; // 답변 개수
public void updateCommentCount(int count) {
    this.commentCount = count;
}

1. 오류 해결
Domain
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id", nullable = false)
private User user;
=

@Builder
public Question(String title, String content, User user) {
    this.title = title;
    this.content = content;
    this.user = user;
    this.viewCount = 0;
}
=

@PrePersist
protected void onCreate() {
    this.createdAt = LocalDateTime.now();
}

@PreUpdate
protected void onUpdate() {
    this.updatedAt = LocalDateTime.now();
}
=

repository
public interface QuestionRepository extends JpaRepository<Question, Long> {
    // 특정 유저의 질문 목록을 페이징하여 조회
    Page<Question> findByUser_Id(Long userId, Pageable pageable);
}
=

controller
package com.example.qnaboard.controller.question;

import com.example.qnaboard.dto.question.request.QuestionCreateRequest;
import com.example.qnaboard.dto.question.request.QuestionUpdateRequest;
import com.example.qnaboard.dto.question.response.QuestionResponse;
import com.example.qnaboard.dto.user.response.MyQuestionSummaryResponse;
import com.example.qnaboard.security.CustomUserDetails;
import com.example.qnaboard.service.quesiton.QuestionService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/questions")
@RequiredArgsConstructor
public class QuestionController {

    private final QuestionService questionService;

    // 1. 등록
    @PostMapping
    public ResponseEntity<QuestionResponse> register(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @Valid @RequestBody QuestionCreateRequest request
    ) {
        QuestionResponse response =
                questionService.createQuestion(userDetails.getUserId(), request);

        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    // 2. 전체 목록 조회
    @GetMapping
    public Page<QuestionResponse> list(@PageableDefault Pageable page) {
        return questionService.getQuestionList(page);
    }

    // 3. 상세 조회
    @GetMapping("/{id}")
    public QuestionResponse getDetail(@PathVariable Long id) {
        return questionService.getQuestionDetail(id);
    }

    // 4. 수정
    @PutMapping("/{id}")
    public ResponseEntity<QuestionResponse> update(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @PathVariable Long id,
            @Valid @RequestBody QuestionUpdateRequest request
    ) {

        QuestionResponse updated = questionService.updateQuestion(userDetails.getUserId(), id, request);

        return ResponseEntity.ok(updated);
    }

    // 5. 삭제
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @PathVariable Long id
    ) {
        questionService.deleteQuestion(userDetails.getUserId(), id);
        return ResponseEntity.noContent().build();
    }

    // 6. 내가 쓴 글 조회
    @GetMapping("/userPost")
    public Page<MyQuestionSummaryResponse> getUserPost(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @PageableDefault Pageable page
    ) {
        return questionService.getMyQuestions(userDetails.getUserId(), page);
    }
}
service
package com.example.qnaboard.service.quesiton;

import com.example.qnaboard.domain.question.Question;
import com.example.qnaboard.domain.user.User;
import com.example.qnaboard.dto.question.request.QuestionCreateRequest;
import com.example.qnaboard.dto.question.request.QuestionUpdateRequest;
import com.example.qnaboard.dto.question.response.QuestionResponse;
import com.example.qnaboard.dto.user.response.MyQuestionSummaryResponse;
import com.example.qnaboard.exception.QuestionErrorCode;
import com.example.qnaboard.exception.UserErrorCode;
import com.example.qnaboard.exception.common.BusinessException;
import com.example.qnaboard.repository.question.QuestionRepository;
import com.example.qnaboard.repository.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional
public class QuestionService {

    private final QuestionRepository questionRepository;
    private final UserRepository userRepository;

    // 질문 등록
    public QuestionResponse createQuestion(
            Long userId,
            QuestionCreateRequest request
    ) {
        // 작성자 유저 확인
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new BusinessException(UserErrorCode.USER_NOT_FOUND));

        // 질문 엔티티 생성 및 빌드
        Question question = Question.builder()
                .title(request.title())
                .content(request.content())
                .user(user)
                .build();
        Question saved = questionRepository.save(question);

        return new QuestionResponse(
                saved.getId(),
                saved.getTitle(),
                saved.getContent(),
                saved.getCreatedAt(),
                new QuestionResponse.UserResponse(
                        user.getId(),
                        user.getUsername()
                )
        );
    }

    // 질문 전체 목록 조회
    @Transactional(readOnly = true)
    public Page<QuestionResponse> getQuestionList(Pageable pageable) {
        return questionRepository.findAll(pageable)
                .map(question -> new QuestionResponse(
                        question.getId(),
                        question.getTitle(),
                        question.getContent(),
                        question.getCreatedAt(),
                        new QuestionResponse.UserResponse(
                                question.getUser().getId(),
                                question.getUser().getUsername()
                        )
                ));
    }

    // 질문 상세 조회
    @Transactional
    public QuestionResponse getQuestionDetail(Long questionId) {
        Question question = questionRepository.findById(questionId)
                .orElseThrow(() -> new BusinessException(QuestionErrorCode.QUESTION_NOT_FOUND));

        return new QuestionResponse(
                question.getId(),
                question.getTitle(),
                question.getContent(),
                question.getCreatedAt(),
                new QuestionResponse.UserResponse(
                        question.getUser().getId(),
                        question.getUser().getUsername()
                )
        );
    }

    // 질문 수정
    public QuestionResponse updateQuestion(
            Long userId,
            Long questionId,
            QuestionUpdateRequest request
    ) {
        Question question = questionRepository.findById(questionId)
                .orElseThrow(() -> new BusinessException(QuestionErrorCode.QUESTION_NOT_FOUND));

        // 본인 소유 확인
        validateOwner(question, userId);
        question.update(request.title(), request.content());

        return new QuestionResponse(
                question.getId(),
                question.getTitle(),
                question.getContent(),
                question.getCreatedAt(),
                new QuestionResponse.UserResponse(
                        question.getUser().getId(),
                        question.getUser().getUsername()
                )
        );

    }

    // 질문 삭제
    public void deleteQuestion(
            Long userId,
            Long questionId
    ) {
        Question question = questionRepository.findById(questionId)
                .orElseThrow(() -> new BusinessException(QuestionErrorCode.QUESTION_NOT_FOUND));

        // 본인 소유 확인
        validateOwner(question, userId);
        questionRepository.delete(question);
    }

    // MyQuestion
    @Transactional(readOnly = true)
    public Page<MyQuestionSummaryResponse> getMyQuestions(Long userId, Pageable pageable) {
        return questionRepository.findByUser_Id(userId, pageable)
                .map(question -> new MyQuestionSummaryResponse(
                        question.getId(),
                        question.getTitle(),
                        question.getCreatedAt().toString()
                ));
    }

    // 검증
    private void validateOwner(Question question, Long userId) {
        // 엔티티의 유저 ID와 로그인한 유저 ID 비교
        if (!question.getUser().getId().equals(userId)) {
            throw new BusinessException(QuestionErrorCode.UNAUTHORIZED_USER);
        }
    }
}
2. 기능 추가
레파지토리에 추가 = 제목 내 키워드 포함된 질문 검색 Page<Question> findByTitleContainingOrContentContaining(String title, String content, Pageable pageable);
서비스에 추가 = 키워드로 검색하기
@Transactional(readOnly = true)
public Page<QuestionResponse> getQuestionList(String keyword, Pageable pageable) {
    Page<Question> questions;
    if (keyword != null && !keyword.isBlank()) {    // keyword가 비어있지 않으면 검색, 비어있으면 전체 조회
        questions = questionRepository.findByTitleContainingOrContentContaining(keyword, keyword, pageable);
    } else {
        questions = questionRepository.findAll(pageable);
    }
    return questions.map(this::convertToQuestionResponse);
}
정렬 시키는법 컨트롤러가 받은 Pageable 파라미터가 자동으로 처리함 인기순: 사용자가 /api/questions?sort=viewCount,desc로 요청하면 JPA가 내부적으로 ORDER BY viewCount DESC 답변 많은 순: Question 엔티티에 commentCount 필드를 추가했다면, 동일하게 ?sort=commentCount,desc로 요청 정렬용 필드 작성
@Column(nullable = false)
private int commentCount = 0; // 답변 개수
public void updateCommentCount(int count) {
    this.commentCount = count;
}
