# Kong Entity

- Kong의 entity는 기본적으로 유연하며, 만들고자 하는 시스템 구성에 따라 다양하게 사용할 수 있다

## Entity와 일반적인 용도

> Kong Gateway의 Request 흐름은 <br>
> Request -> Consumer (Group) -> Route -> Service -> (upstream)
의 구조를 가진다
- Rate Limit Plugin은 어디에 붙이냐에 따라서 적용 범위가 바뀐다
- 다른 Plugin도 붙이는 위치에 따라서 작동이 바뀐다
- Consumer
  - 소비자로써 Group을 만들어 속하도록 할 수 있다
- Route
  - domain/header등 라우팅의 정보들이 포함된다
- Service
  - path를 하나의 서비스로 설정한다
  - upstream을 통해 게이트의 요청을 받을 lb를 지정할 수 있다
  - RateLimit을 이 곳에 붙이고, ACL을 Service 설정하는게 일반적으로 경제적이다
     - 기본 Ratelimit을 선언하고, 예외 Consumer에 대한 RateLimit entity를 추가하는 형식
     - ACL을 통해 허가를 받은 Consumer만 받아주는 형식이 Gateway의 일반적인 컨셉에 알맞다
- 실제 엔진의 처리는 `Request → Route 매칭 → Service 식별 → (Auth 플러그인 확인) → Consumer 식별 → Upstream`순이다