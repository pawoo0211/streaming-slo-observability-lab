# Streaming SLO & Observability Lab
Kafka / Flink / Hadoop(Iceberg) 기반 스트리밍 파이프라인에서  
End-to-End Latency를 SLO로 정의하고 운영 판단 기준을 검증하는 실험 프로젝트입니다.

---

## 1. Background

Kafka/Flink 기반 스트리밍 시스템에서는 Consumer Lag이 정상임에도 불구하고  
실제 데이터 처리 지연이나 사용자 영향이 발생하는 경우가 많습니다.

본 프로젝트는 다음 질문에서 출발합니다.

- Lag이 정상일 때도 시스템을 장애로 판단해야 하는 시점은 언제인가?
- 운영 판단 기준을 정책이 아니라 **기술적 지표(SLO)**로 정의할 수 있는가?

이를 위해 Lag 중심 관측의 한계를 검증하고,  
**End-to-End Latency 기반 SLO**를 운영 판단 기준으로 실험합니다.

---

## 2. Architecture

