# 현재 상태

## ContentBookmarkController
```
package popeye.popeyebackend.content.controller;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.*;
import popeye.popeyebackend.content.service.ContentBookmarkService;

@RestController
@RequestMapping("/api/contents")
@RequiredArgsConstructor
public class ContentBookmarkController {

    private final ContentBookmarkService bookmarkService;

    @PostMapping("/{id}/bookmark")
    public ResponseEntity<Void> bookmark(@AuthenticationPrincipal UserDetails userDetails, @PathVariable Long id) {
        Long userId = userDetails.getUserId();
        bookmarkService.bookmark(userId, id);
        return ResponseEntity.ok().build();
    }

    @DeleteMapping("/{id}/bookmark")
    public ResponseEntity<Void> unbookmark(@PathVariable Long id) {
        bookmarkService.unbookmark(1L, id);
        return ResponseEntity.noContent().build();
    }
}

```

## ContentController
```
package popeye.popeyebackend.content.controller;

import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.web.bind.annotation.*;
import popeye.popeyebackend.content.dto.request.ContentCreateRequest;
import popeye.popeyebackend.content.dto.response.ContentResponse;
import popeye.popeyebackend.content.service.ContentService;

@RestController
@RequestMapping("/api/contents")
@RequiredArgsConstructor
public class ContentController {

    private final ContentService contentService;

    @PostMapping
    public ResponseEntity<Long> create(@RequestBody ContentCreateRequest req) {
        Long id = contentService.createContent(1L, req); // 임시 userId
        return ResponseEntity.status(HttpStatus.CREATED).body(id);
    }

    @GetMapping("/{id}")
    public Object get(@PathVariable Long id) {
        boolean hasPurchased = paymentService.hasPurchased(userId, contentId); //결제부분
        return contentService.getContentWithAccessControl(id, hasPurchased);
    }
    @DeleteMapping("/{contentId}")
    public ResponseEntity<Void> delete(
            @AuthenticationPrincipal CustomUserDetails user,
            @PathVariable Long contentId
    ) {
        contentService.deleteContent(user.getUserId(), contentId);
        return ResponseEntity.noContent().build();
    }

}

```

## ContentLikeController
```
package popeye.popeyebackend.content.controller;

import jakarta.persistence.*;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import popeye.popeyebackend.content.domain.Content;
import popeye.popeyebackend.content.domain.ContentLike;
import popeye.popeyebackend.user.domain.User;

import java.time.LocalDateTime;

@Entity
@Table(name = "content_likes")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ContentLikeController {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "content_id", nullable = false)
    private Content content;

    private LocalDateTime createdAt;

    public static ContentLike create(User user, Content content) {
        ContentLike like = new ContentLike();
        like.user = user;
        like.content = content;
        like.createdAt = LocalDateTime.now();
        return like;
    }

    @PostMapping("/{id}/like")
    public ResponseEntity<Void> toggleLike(@PathVariable Long id) {
        likeService.toggleLike(1L, id); // 임시 userId
        return ResponseEntity.ok().build();
    }
}
```

## Domain - Content
```
package popeye.popeyebackend.content.domain;

import jakarta.persistence.*;
import lombok.Builder;
import lombok.Getter;
import popeye.popeyebackend.pay.domain.Order;
import popeye.popeyebackend.content.enums.ContentStatus;
import popeye.popeyebackend.user.domain.User;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.List;

@Entity
@Table(name = "contents")
@Getter
@Builder
public class Content {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private String content;

    private int price;

    private int discountRate;

    private boolean isFree;

    private Integer viewCount;

    private LocalDateTime createdAt =  LocalDateTime.now();

    private LocalDateTime modifiedAt;

    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private ContentStatus contentStatus = ContentStatus.INACTIVE;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "creator_id")
    private User creator;

    @OneToMany(mappedBy = "content")
    private List<ContentBookmark> bookmarks;

    @OneToMany(mappedBy = "content")
    private List<Order> orders;

    public void activate() {    // 컨텐츠 공개
        this.contentStatus = ContentStatus.ACTIVE;
        this.modifiedAt = LocalDateTime.now();
    }

    public void inactivate() {
        this.contentStatus = ContentStatus.INACTIVE;
        this.modifiedAt = LocalDateTime.now();
    }

    public boolean isActive() {
        return this.contentStatus.equals(ContentStatus.ACTIVE);
    }

    private long viewCount = 0;
    public void increaseViewCount() {
        this.viewCount++;
    }

    public void publish() {
        this.status = ContentStatus.ACTIVE;
        this.modifiedAt = LocalDateTime.now();
    }

    private long likeCount = 0;

    public void increaseLike() {
        this.likeCount++;
    }

    public void decreaseLike() {
        if (this.likeCount > 0) {
            this.likeCount--;
        }
    }
}
```

## Domain - ContentBookmark
```
package popeye.popeyebackend.content.domain;

import jakarta.persistence.*;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import popeye.popeyebackend.user.domain.User;

import java.time.LocalDateTime;

@Entity
@Table(name = "content_bookmarks")
@Getter
@NoArgsConstructor
public class ContentBookmark {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "bookmark_id")
    private Long id;

    private Integer price;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id") // BIGINT(FK)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "content_id") // BIGINT(FK)
    private Content content;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt = LocalDateTime.now();

    @Builder
    public ContentBookmark(User user, Content content, Integer price) {
        this.user = user;
        this.content = content;
        this.price = price;
    }
}
```

## ContentLike
```
package popeye.popeyebackend.content.domain;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.NoArgsConstructor;
import popeye.popeyebackend.user.domain.User;

import java.time.LocalDateTime;

@Entity
@Table(name = "content_likes")
@Getter
@NoArgsConstructor
public class ContentLike {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "like_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "content_id")
    private Content content;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt = LocalDateTime.now();

    public static ContentLike create(User user, Content content) {
        ContentLike like = new ContentLike();
        like.user = user;
        like.content = content;
        like.createdAt = LocalDateTime.now();
        return like;
    }

}
```

## service - ContentBookmarkService
```
package popeye.popeyebackend.content.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import popeye.popeyebackend.content.domain.Content;
import popeye.popeyebackend.content.domain.ContentBookmark;
import popeye.popeyebackend.content.exception.ContentError;
import popeye.popeyebackend.content.repository.ContentBookmarkRepository;
import popeye.popeyebackend.content.repository.ContentRepository;
import popeye.popeyebackend.user.domain.User;
import popeye.popeyebackend.user.repository.UserRepository;

@Service
@RequiredArgsConstructor
@Transactional
public class ContentBookmarkService {

    private final ContentBookmarkRepository bookmarkRepository;
    private final ContentRepository contentRepository;
    private final UserRepository userRepository;

    public void bookmark(Long userId, Long contentId) {
        User user = userRepository.findById(userId)
                .orElseThrow();
        Content content = contentRepository.findById(contentId)
                .orElseThrow(() -> new ContentError("컨텐츠를 찾을 수 없습니다."));

        if (bookmarkRepository.existsByUserAndContent(user, content)) {
            throw new IllegalStateException("이미 북마크됨");
        }
        ContentBookmark contentBookmark = ContentBookmark.builder()
                .user(user)
                .content(content)
                .price(content.getPrice()).build();

        bookmarkRepository.save(contentBookmark);
    }

    public void unbookmark(Long userId, Long contentId) {
        User user = userRepository.findById(userId).orElseThrow();
        Content content = contentRepository.findById(contentId).orElseThrow();

        bookmarkRepository.deleteByUserAndContent(user, content);
    }
}
```

## service - ContentLikeService
```
package popeye.popeyebackend.content.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import popeye.popeyebackend.content.domain.Content;
import popeye.popeyebackend.content.domain.ContentLike;
import popeye.popeyebackend.content.exception.ContentError;
import popeye.popeyebackend.content.repository.ContentLikeRepository;
import popeye.popeyebackend.content.repository.ContentRepository;
import popeye.popeyebackend.user.domain.User;
import popeye.popeyebackend.user.repository.UserRepository;

@Service
@RequiredArgsConstructor
@Transactional
public class ContentLikeService {

    private final ContentLikeRepository likeRepository;
    private final ContentRepository contentRepository;
    private final UserRepository userRepository;

    public void toggleLike(Long userId, Long contentId) {
        User user = userRepository.findById(userId).orElseThrow();

        Content content = contentRepository.findById(contentId)
                .orElseThrow(() -> new ContentError("컨텐츠를 찾을 수 없습니다."));

        boolean alreadyLiked =
                likeRepository.existsByUserAndContent(user, content);

        if (alreadyLiked) {
            likeRepository.deleteByUserAndContent(user, content);
            content.decreaseLike();
        } else {
            likeRepository.save(ContentLike.create(user, content));
            content.increaseLike();
        }
    }
}
```

## service - ContentService
```
package popeye.popeyebackend.content.service;

import jakarta.transaction.Transactional;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import popeye.popeyebackend.content.domain.Content;
import popeye.popeyebackend.content.dto.request.ContentCreateRequest;
import popeye.popeyebackend.content.dto.response.ContentResponse;
import popeye.popeyebackend.content.dto.response.FullContentResponse;
import popeye.popeyebackend.content.dto.response.PreviewContentResponse;
import popeye.popeyebackend.content.enums.ContentStatus;
import popeye.popeyebackend.content.repository.ContentRepository;
import popeye.popeyebackend.user.domain.User;
import popeye.popeyebackend.user.repository.UserRepository;

@Service
@RequiredArgsConstructor
@Transactional
public class ContentService {
    private final ContentRepository contentRepository;
    private final UserRepository userRepository;

    public Long createContent(Long userId, ContentCreateRequest req) {
        User creator = userRepository.findById(userId).orElseThrow();

        Content content = Content.create( //build 패턴
                creator,
                req.getTitle(),
                req.getBody(),
                req.getPrice(), //null 처리하기
                req.getDiscountRate(), //여기도
                req.isFree()
        );

        return contentRepository.save(content).getId();
    }

    public ContentResponse getContent(Long contentId) {
        Content content = contentRepository
                .findByIdAndContentStatus(contentId, ContentStatus.ACTIVE)
                .orElseThrow();

        content.increaseViewCount();
        return ContentResponse.from(content);
    }

    @Transactional(readOnly = true)
    public Object getContentWithAccessControl(Long contentId, boolean hasPurchased) {
        Content c = contentRepository.findByIdAndStatus(contentId, ContentStatus.ACTIVE).orElseThrow();
        c.increaseViewCount();

        if (c.isFree() || hasPurchased) {
            return FullContentResponse.from(c);
        }
        return PreviewContentResponse.from(c);
    }

    public void deleteContent(Long userId, Long contentId) {
        Content content = contentRepository.findById(contentId)
                .orElseThrow();

        if (!content.getCreator().getId().equals(userId)) {
            throw new IllegalStateException("작성자만 삭제할 수 있습니다.");
        }

        content.inactivate(); // 실제 삭제 아님 (비공개 처리)
    }

}
```
