#### assertions

```
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;
```

```java
IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));

// assertThrows(IllegalStateException.class, ()->memberService.join(member2));
//만약 ()->memberSerivce.join(member2) 메서드를 실행할 때 throws를 이용해IllegalStateException클래스를 발생시킵니다.

assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원.");
```

