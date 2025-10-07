#### NoSql
- flexible schema
    - application 레벨에서 스키마 관리가 필요
- 중복 허용 (join 회피)
    - application 레벨에서 중복된 데이터들이 모두 최신 데이터를 유지할 수 있도록 관리해야 함
- scale-out에 최적화
- ACID의 일부를 포기하고 high-throughput, low-latency 추구
    - 금융 시스템 처럼 일관성이 중요한 환경에선 조심스러움