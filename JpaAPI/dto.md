# API 개발시 DTO를 사용해야 하는 이유

## 엔티티를 RequestBody에 직접 매핑하면 생기는 문제점

### Member 엔티티를 직접 받는경우

```java
@PostMapping("/api/v1/members")
public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```

- 엔티티에 @NotEmpty와 같은 API 검증을 위한 로직이 들어가게 된다.
- 엔티티가 변경되면 API 스펙이 변한다
- 한 엔티티에 각각의 API를 위한 모든 요청 요구사항을 담기가 어렵다.
- 실무에서는 엔티티를 API 스펙에 노출하면 안된다

### 엔티티 대신 DTO를 RequestBody에 매핑하는 경우

```java
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
    Member member = new Member();
    member.setName(request.getName());

    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```

- CreateMemberRequest를 만들어서 매핑하였다.
- 엔티티와 프레젠테이션 계층을 위한 로직을 분리할 수 있다.
- 엔티티와 API 스펙을 명확하게 분리할 수 있고, 엔티티가 변해도 API 스펙이 변하지 않는다.

---

## Reference

- [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard)
