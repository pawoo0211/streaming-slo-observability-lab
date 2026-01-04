# Streaming SLO & Observability Lab

Kafka / Flink / Hadoop(Iceberg) 기반 스트리밍 파이프라인에서  
**End-to-End Latency를 SLO로 정의하고 운영 판단 기준을 검증**하는 실험 프로젝트입니다.

본 프로젝트는 기능 구현이 아니라,  
**운영 환경에서 언제 시스템을 장애로 판단해야 하는지**를 기술적으로 증명하는 것을 목표로 합니다.

본 프로젝트의 도메인은 **eCommerce 상품(Product)**이며, 상품 변경 이벤트(가격/재고/상태/카테고리 등)의 스트리밍 처리에서 End-to-End Latency SLO와 운영 판단 기준을 검증한다.
---

## Background

Kafka / Flink 기반 스트리밍 시스템에서는 다음과 같은 문제가 반복적으로 발생합니다.

- Consumer Lag은 정상이나 실제 데이터 처리는 지연됨
- Sink 병목이나 Checkpoint 지연이 Lag 지표에 반영되지 않음
- 장애 판단이 경험이나 감에 의존함

이로 인해 운영자는 **문제가 발생한 이후에야** 장애를 인지하게 됩니다.

본 프로젝트는 Lag 중심 관측의 한계를 검증하고,  
**End-to-End Latency와 Processing Freshness를 운영 판단의 기준(SLO)**으로 삼는 접근을 실험합니다.

---

## Architecture

본 프로젝트는 **운영 판단이 실제로 결정되는 스트리밍 구간**에만 집중합니다.

### Data Flow

Kafka Producer → Kafka Topic → Flink Job (Stateful, Event Time) → Hadoop Iceberg Table → Prometheus / Grafana

### 구성 요소 설명

- **Kafka Producer**: 이벤트에 `event_time`, `ingest_time` 등을 포함해 전송합니다.
- **Kafka Topic**: 입력 이벤트 스트림을 수신합니다.
- **Flink Job**: 상태(state)를 사용하는 스트리밍 처리(Event Time + Watermark 기반)를 수행합니다.
- **Iceberg Table**: 처리 결과를 적재합니다. sink 병목 및 checkpoint 영향을 재현하기 위한 목적도 포함합니다.
- **Prometheus/Grafana**: Kafka/Flink/E2E 지표를 수집 및 시각화합니다.

Spark, Backend Application, MongoDB 등은  
운영 판단과 직접적인 관련이 없기 때문에 **의도적으로 제외**했습니다.

---

## Technology Stack

### Streaming / Data
- Apache Kafka
- Apache Flink
- Hadoop Iceberg

### Observability
- Prometheus
- Grafana

### Infrastructure
- Docker
- Docker Compose

---

## Observability Metrics

### Kafka Metrics
- Consumer Lag
- Throughput (records in / out)
- Broker Latency

### Flink Metrics
- Checkpoint Duration
- Checkpoint Failure Count
- Backpressure (Task Busy Time)
- Restart Count
- State Size

### End-to-End Metrics
- End-to-End Latency (p50 / p95 / p99)
- Processing Freshness (current time - event_time)

---

## SLO Definition

- **Latency SLO**: p95 End-to-End Latency < **5s** (10-minute rolling window)
- **Freshness SLO**: p99 Processing Freshness < **30s**
- **Reliability**: Processing success rate > **99.9%**

Consumer Lag은 참고 지표로만 사용하며,  
**SLO breach 판단에는 직접 사용하지 않습니다.**

---

## Failure & Stress Scenarios

다음 장애/부하 시나리오를 의도적으로 주입하여 Lag과 사용자 영향의 괴리를 검증합니다.

- Iceberg write 지연 발생
- Flink TaskManager 재시작
- Checkpoint timeout 유도

각 시나리오에서 다음을 관측합니다.

- Consumer Lag 변화 여부
- End-to-End Latency spike 발생 시점
- SLO breach 발생 시점과 운영 판단의 차이

---

## Operational Decisions

실험 결과를 기반으로 다음과 같은 운영 판단 기준을 도출합니다.

- Lag이 정상이어도 End-to-End Latency SLO가 깨지면 장애로 판단
- Checkpoint failure 증가 시 write path 또는 state 점검
- Sink 병목이 지속되면 parallelism 조정 또는 저장소 분리 검토

---

## Conclusion

스트리밍 시스템의 안정성은 Lag이 아니라  
**End-to-End Latency와 Processing Freshness 기준으로 판단되어야 합니다.**

본 프로젝트는 운영 판단을  
**자동화 가능한 기술 기준(SLO)**으로 정리하는 것을 목표로 합니다.
