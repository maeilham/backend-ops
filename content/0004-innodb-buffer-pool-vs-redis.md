---
title: "InnoDB 버퍼 풀이 있는데도 Redis 캐시를 쓰는 이유는?"
preview: "InnoDB도 자주 읽은 데이터 페이지를 메모리에 캐시합니다. 그런데도 Redis를 따로 두는 이유는 둘이 캐시하는 대상과 줄여주는 비용이 다르기 때문입니다."
tags: [mysql, redis, cache, database]
---

둘은 **캐시하는 대상이 다릅니다.** InnoDB 버퍼 풀은 디스크에서 읽은 **데이터·인덱스 페이지**(기본 16KB)를 메모리에 올려 디스크 I/O를 줄입니다. Redis는 애플리케이션이 **가공을 끝낸 결과값**을 키-값으로 저장해 쿼리 실행 자체를 건너뜁니다.

**버퍼 풀이 줄여주지 못하는 비용**

페이지가 메모리에 있어도 아래 비용은 그대로 남습니다.

- SQL 파싱과 실행 계획 수립
- B+Tree 인덱스 탐색, row 조립, MVCC 처리
- 조인·정렬·집계 같은 계산
- 애플리케이션과 DB 사이의 커넥션과 네트워크 왕복

MySQL 8.0부터는 쿼리 캐시가 제거되어, DB가 쿼리 결과를 대신 캐시해주지 않습니다.

**Redis를 썼을 때 얻는 이득**

- **계산을 건너뜁니다.** 무거운 조인·집계 결과를 저장해두면 같은 요청은 키 조회 한 번으로 끝납니다.
- **DB 부하가 줄어듭니다.** 읽기 요청이 DB의 커넥션과 CPU까지 도달하지 않아 같은 DB로 더 많은 요청을 받을 수 있습니다.
- **DB 서버 밖에서 확장할 수 있습니다.** 버퍼 풀은 DB 서버 한 대의 메모리에 묶이지만, Redis는 노드를 늘리거나 클러스터로 구성할 수 있습니다.
- **값 단위로 캐시합니다.** row 하나를 읽어도 페이지 전체가 버퍼 풀을 차지하지만, Redis는 필요한 값만 저장합니다.
- **TTL과 다양한 자료구조를 쓸 수 있습니다.** 세션, 랭킹(sorted set), 카운터 같은 용도를 다룹니다.

가장 흔한 사용 방식은 **cache-aside**입니다.

```go
func GetUser(ctx context.Context, id int64) (*User, error) {
    key := fmt.Sprintf("user:%d", id)
    if v, err := rdb.Get(ctx, key).Result(); err == nil {
        return decode(v), nil // 캐시 히트: DB를 거치지 않음
    }
    u, err := db.QueryUser(ctx, id) // 캐시 미스: DB 조회
    if err != nil {
        return nil, err
    }
    rdb.Set(ctx, key, encode(u), 10*time.Minute) // TTL로 오래된 값 정리
    return u, nil
}
```

**주의할 점**

Redis는 공짜가 아닙니다. 원본과 캐시가 어긋나는 **일관성 문제**, 캐시가 한꺼번에 만료될 때 DB로 요청이 몰리는 **스탬피드**, 운영할 시스템이 하나 늘어나는 비용이 따라옵니다.

그래서 먼저 버퍼 풀이 충분한지 확인합니다. 아래 두 값으로 히트율(`1 - reads / read_requests`)을 볼 수 있고, 히트율이 높은데도 DB가 느리다면 병목은 I/O가 아니라 쿼리 실행이나 커넥션일 가능성이 큽니다.

```sql
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_read%';
-- Innodb_buffer_pool_read_requests: 버퍼 풀에 요청한 논리적 읽기 횟수
-- Innodb_buffer_pool_reads: 버퍼 풀에 없어 디스크에서 읽은 횟수
```

인덱스와 쿼리를 먼저 튜닝하고, 그래도 같은 결과를 반복해서 계산하는 부하가 남을 때 Redis를 도입하는 순서가 안전합니다.

## 참고

- [MySQL 공식 문서 — InnoDB Buffer Pool](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool.html)
- [MySQL 공식 문서 — What Is New in MySQL 8.0 (쿼리 캐시 제거)](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)
- [AWS 백서 — Database Caching Strategies Using Redis](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/welcome.html)
