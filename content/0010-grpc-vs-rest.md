---
title: "MSA에서는 왜 REST 대신 gRPC를 사용할까요?"
preview: "서비스 간 통신에 익숙한 REST와 JSON 대신 gRPC와 Protobuf를 사용하는 이유는 무엇일까요?"
tags: [grpc, rest, protobuf, http2, msa]
---

MSA에서는 여러 서비스가 서로 네트워크를 통해 통신해요. 서비스 간 통신량이 많아지면 데이터 직렬화 비용과 네트워크 대역폭도 중요해져요.

이때 REST 대신 **gRPC**를 선택할 수 있어요. gRPC는 RPC(Remote Procedure Call) 프레임워크로, 원격 서버의 메서드를 호출하는 방식으로 통신해요.

**JSON 대신 Protobuf를 사용하는 이유**

JSON은 사람이 읽기 편한 텍스트 기반 포맷이에요. 데이터를 전송하려면 객체를 JSON 문자열로 변환하고, 받은 문자열을 다시 객체로 변환해야 해요.

반면 Protobuf는 미리 정의한 스키마를 기반으로 데이터를 바이너리로 인코딩해요.

```protobuf
message User {
  int32 id = 1;
  string name = 2;
  int32 age = 3;
}
```

필드 번호와 타입을 이용해 데이터를 표현하므로 일반적으로 JSON보다 전송 크기가 작고 직렬화 및 역직렬화도 효율적이에요.

단, Protobuf 역시 직렬화와 역직렬화 과정은 필요해요.

**HTTP/2 멀티플렉싱**

gRPC는 HTTP/2를 기반으로 통신해요.

HTTP/2의 멀티플렉싱은 하나의 TCP 연결에서 여러 Stream의 요청과 응답을 동시에 처리할 수 있는 기능이에요.

덕분에 여러 TCP 연결을 생성하는 부담을 줄이고 네트워크 자원을 효율적으로 사용할 수 있어요.

참고로 REST도 HTTP/2를 사용할 수 있으므로 멀티플렉싱이 gRPC만의 장점은 아니에요.

**.proto를 통한 계약 관리**

gRPC는 `.proto` 파일에 서비스와 요청 및 응답 구조를 정의해요.

```protobuf
service UserService {
  rpc GetUser (GetUserRequest) returns (GetUserResponse);
}

message GetUserRequest {
  int32 user_id = 1;
}

message GetUserResponse {
  string name = 1;
  int32 age = 2;
}
```

이 정의를 기반으로 클라이언트와 서버 코드를 생성할 수 있어요.

덕분에 서비스 간 데이터 구조와 통신 규격을 명확하게 관리할 수 있고, 타입 불일치나 스키마 변경 문제도 개발 단계에서 확인하기 쉬워요.

**결론**

gRPC를 사용하는 주요 이유는 다음과 같아요.

- **Protobuf**: 데이터 크기와 직렬화 비용 절감
- **HTTP/2**: 멀티플렉싱을 통한 효율적인 통신
- **.proto**: 서비스 간 명확한 계약과 코드 생성

일반적으로 외부 공개 API나 브라우저 통신에는 REST가 적합한 경우가 많고, 내부 서비스 간 고성능 통신이나 Streaming이 필요한 경우에는 gRPC가 좋은 선택지가 될 수 있어요.

## 참고

- [gRPC 공식 문서](https://grpc.io/docs/what-is-grpc/introduction/)
- [Protocol Buffers 공식 문서](https://protobuf.dev/overview/)
