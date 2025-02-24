---
layout: post
title: Prometheus Exporter
date: 2025-02-20 15:45:00 +0900
categories: [Monitoring]
tags: [Monitoring, Prometheus]
description: Prometheus Exporter에 대해 알아보고, CentOS에 Node Exporter 설치하기
toc: true
---

## Prometheus Exporter란?
Prometheus는 강력한 모니터링 및 경고 시스템으로, 다양한 메트릭을 수집하여 시계열 데이터를 저장하고 분석할 수 있습니다.
하지만 Prometheus 자체는 기본적으로 애플리케이션의 내부 메트릭을 직접 가져오는 방식(Pull)을 사용하기 때문에,
외부 시스템이나 애플리케이션의 메트릭을 수집하려면 추가적인 도구가 필요합니다. 
이를 해결하는 것이 **Prometheus Exporter**입니다.

---

## Exporter를 사용하는 이유
Exporter는 Prometheus가 특정 시스템의 상태를 모니터링 할 수 있도록 
메트릭을 표준화된 형식으로 변환하여 HTTP 엔드포인트(/metrics)를 통해 데이터를 노출합니다.

- **다양한 시스템 모니터링**: Prometheus가 직접 지원하지 않는 외부 애플리케이션 및 시스템의 상태를 모니터링할 수 있습니다.

- **일관된 데이터 포맷 제공**: 모든 메트릭을 Prometheus가 이해할 수 있는 포맷으로 변환하여 제공합니다.

- **유연한 확장성**: 필요한 경우 새로운 Exporter를 추가하여 모니터링 범위를 쉽게 확장 가능합니다.

- **별도 에이전트 설치 필요 없음**: Exporter는 독립적으로 실행되며, 기존 애플리케이션에 직접적인 영향을 주지 않습니다.

---

## 대표적인 Exporter
다양한 Exporter가 존재하며, 사용자의 환경에 맞춰 적절한 Exporter를 선택하여 사용할 수 있습니다.

| 카테고리                  | Exporter                                  | 설명                                           |
|-----------------------|-------------------------------------------|----------------------------------------------|
| 운영 체제                 | Node Exporter                             | 리눅스 및 유닉스 기반 서버의 CPU, 메모리, 디스크 사용량 등을 수집     |
|| Window Exporter       | Windows 시스템의 CPU, 메모리, 네트워크 정보 수집         |
| 데이터베이스                | MySQL Exporter                            | MySQL/MariaDB의 성능 및 상태 정보 제공                 |
|| PostgreSQL Exporter   | PostgreSQL의 쿼리 성능 및 리소스 사용량 모니터링          |
|| MongoDB Exporter      | MongoDB 클러스터 상태 및 성능 모니터링                 |
|| Redis Exporter        | Redis의 키 개수, 메모리 사용량, 커넥션 수 등의 메트릭 수집     |
| 메시지 브로커               | Kafka Exporter                            | Apache Kafka의 소비자 그룹, 파티션, 오프셋 모니터링          |
|| RabbitMQ Exporter     | RabbitMQ 큐 상태, 메시지 처리량 정보 제공              |
| 웹 서버 및 <br> 애플리케이션 서버 | Nginx Exporter                            | Nginx의 연결 상태 및 요청 처리량 모니터링                   |
|| Apache Exporter       | Apache HTTP 서버의 요청 및 트래픽 모니터링             |
| 클라우드 및 기타             | AWS CloudWatch Exporter                   | AWS CloudWatch 메트릭을 Prometheus에서 활용 가능하도록 변환 |
|| Blackbox Exporter     | HTTP, TCP, ICMP 프로토콜을 사용하여 외부 서비스의 가용성 확인 |
|| Process Exporter      | 특정 프로세스의 CPU 및 메모리 사용량 추적                 |
|| JMX Exporter          | Java 애플리케이션(JVM) 모니터링                     |

---

## Node Exporter 설치하기

**1. CentOS 업데이트**
```bash
sudo yum update -y
```

**2. Node Exporter 사용자 생성**
```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

**3. [Node Exporter 다운로드](https://github.com/prometheus/node_exporter/releases)**
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```

**4. 압축 풀기**
```bash
tar -xvf node_exporte*.tar.gz
```

**5. 압축 푼 디렉토리로 이동**
```bash
cd node_exporte*/
```

**6. Node Exporter 구성 파일 옮기기**
```bash
sudo mv node_exporter /usr/local/bin/
```

**7. 실행 권한 설정**
```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

**8. Node Exporter 서비스 파일 생성**
```bash
sudo vi /etc/systemd/system/node_exporter.service
```
```text
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

**9. Prometheus와 연동 - prometheus.yml 파일 수정**
```bash
sudo vi /etc/prometheus/prometheus.yml
```
```text
scrape_configs: 
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

**10. Node Exporter 서비스 활성화 및 시작**
```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

---

## Node Exporter 설치 확인

설치가 완료되면 브라우저에서 http://<서버_IP>:9100 로 접속하여 Node Exporter 실행 정보를 확인할 수 있습니다.

![img-description](/assets/img/post/monitoring/prometheusexporter/설치완료.png)

위와 같은 화면이 보인다면 Node Exporter가 정상적으로 설치된 것입니다. <br>
화면에 보이는 Metrics를 클릭시 Node Exporter가 수집하는 메트릭 정보를 확인할 수 있습니다.

![img-description](/assets/img/post/monitoring/prometheusexporter/NodeExporterMetrics.png)

다음으로 Prometheus 웹 UI에 접속하면 Node Exporter의 EndPoint와 Metrics가 추가된걸 확인할 수 있습니다.

![img-description](/assets/img/post/monitoring/prometheusexporter/Endpoint.png)
![img-description](/assets/img/post/monitoring/prometheusexporter/PrometheusWeb.png)
![img-description](/assets/img/post/monitoring/prometheusexporter/MetricsExplorer.png)



