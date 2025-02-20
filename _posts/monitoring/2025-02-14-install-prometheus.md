---
layout: post
title: CentOS에 Prometheus 설치하기
date: 2025-02-14 14:29:00 +0900
categories: [Monitoring]
tags: [monitoring, prometheus]
description: CentOS에 Prometheus 설치하기
toc: true
math: true
mermaid: true
---

## Prometheus란?
오픈소스 모니터링 및 알림 시스템으로, 시계열 데이터(Time Series Data)를 수집하고 저장하며
시스템 및 애플리케이션의 상태를 실시간으로 모니터링하는 데 사용됩니다.
또한 알림 설정을 통해 시스템 장애나 성능 저하를 빠르게 파악하고 대응할 수 있습니다.

---

## 주요 용어
**Metric**
- 모니터링 대상에서 수집되는 데이터로 이름과 값으로 구성되며, 레이블을 붙여 더 구체적으로 정의할 수 있습니다.
- 예) http_requests_total{method="GET", status="200"} = 1500
  - http_requests_total = 1500 : 메트릭
  - {method="GET", status="200"} : 레이블

**Label**
- 메트릭에 추가 정보를 붙이는 {key=value} 입니다. 동일한 메트릭이라도 레이블로 여러 차원을 표현할 수 있습니다.

**Endpoint**
- Prometheus가 데이터를 수집하기 위해 접근하는 URL 또는 네트워크 위치입니다.
- Prometheus가 주기적으로 이 엔드포인트를 "스크레이프(Scrape)"해서 데이터를 가져옵니다.

**Scrape**
- Prometheus가 설정된 주기마다 엔드포인트에서 메트릭을 끌어오는(Pull) 과정입니다.

**Time Series(시계열)**
- 시간에 따라 기록된 메트릭 데이터의 연속입니다. Prometheus는 모든 데이터를 시계열로 저장하며, 각 시계열은 메트릭 이름과 레이블 조합으로 고유하게 식별됩니다.

**TSDB(Time Series Database, 시계열 데이터베이스)**
- 시계열 데이터를 저장하는 로컬 데이터베이스입니다. TSDB는 이를 효율적으로 저장하고 쿼리할 수 있게 설계되었습니다. 기본적으로 디스크에 저장되며, 필요하면 외부 저장소 보관도 가능합니다.

**Exporter**
- Prometheus가 직접 접근할 수 없는 시스템(예: 데이터베이스, 하드웨어)의 메트릭을 수집해 Prometheus가 이해할 수 있는 형식으로 변환해주는 도구입니다.

---

## Prometheus 주요 특징
**시계열 데이터 수집**
- 시계열 데이터를 key-value 형태의 메트릭으로 저장합니다.

**강력한 쿼리 언어 (PromQL)**
- Prometheus Query Language(PromQL)를 사용해 저장된 데이터를 자유롭게 쿼리하고 분석할 수 있습니다.

**Pull 방식 데이터 수집**
- 에이전트를 설치해 데이터를 푸시(Push)하는 대신, Prometheus는 타겟 시스템에서 데이터를 끌어오는(Pull) 방식을 사용합니다. 이를 통해 모니터링 대상의 상태를 주기적으로 확인할 수 있습니다.

**서비스 디스커버리**
- 동적인 클라우드 환경에서 자동으로 모니터링 대상을 탐지합니다. Kubernetes, AWS, GCP 같은 환경과 통합이 뛰어나 새로 추가된 노드나 서비스를 바로 감지할 수 있습니다.

**경량화 및 독립성**
- 분산 저장소가 필요 없으며, 단일 바이너리로 실행 가능하여 유지보수가 용이합니다.

**확장 가능한 알림 시스템**
- Alertmanager라는 별도 컴포넌트를 통해 Slack, 이메일, PagerDuty 등으로 알림을 보낼 수 있고, 알림 규칙도 유연하게 설정 가능합니다.

**시각화 통합**
- Grafana 같은 외부 툴과 쉽게 연동되어 데이터를 대시보드로 시각화할 수 있습니다.

---

## Prometheus의 작동 방식
**1. Target에서 Matric 노출**
- Exporter, 애플리케이션 자체 엔드포인트 (/metrics) 등에서 데이터 노출

**2. Prometheus가 주기적으로 데이터 수집**
- 설정된 주기에 따라 타겟에서 데이터를 가져옴

**3. 시계열 데이터 저장**
- 수집한 데이터를 TSDB에 저장

**4. PromQL을 이용한 데이터 조회 및 분석**

---

## CentOS에 Prometheus 설치하기

**1. CentOS 업데이트**
```bash
sudo yum update -y
```

**2. Prometheus 사용자 생성**
```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```
**3. 디렉토리 만들기**
```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

**4. [Prometheus 다운로드](https://prometheus.io/download/)**
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.53.3/prometheus-2.53.3.linux-amd64.tar.gz
```

**5. 압축 풀기**
```bash
tar xvf prometheus*.tar.gz
```

**6. 압축 푼 디렉토리로 이동**
```bash
cd prometheus*/
```

**7. Prometheus 구성 파일 옮기기**
```bash
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

**8. 파일 권한 설정**
```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown prometheus:prometheus /var/lib/prometheus
```

**9. Prometheus 시스템 서비스 등록**
```bash
sudo vi /etc/systemd/system/prometheus.service
```

**10. 내용 추가**
```text
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries \
[Install]
WantedBy=multi-user.target
```

**11. 서비스 활성화 및 시작**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
sudo systemctl start prometheussystem
```

**12. 상태 확인**
```bash
sudo systemctl status prometheus
```
![img-description](/assets/img/post/monitoring/prometheus/서비스상태확인.png)

**13. Prometheus 웹 UI 접속**
<br>
설치가 완료되면 브라우저에서 http://<서버_IP>:9090 로 접속하여 Prometheus 웹 UI를 확인할 수 있습니다.

![img-description](/assets/img/post/monitoring/prometheus/설치완료.png)

---

## 설치간 발생한 문제
### 1. Prometheus 9090 port 충돌 문제
Prometheus는 default로 9090 port를 사용하는데 CentOS에서 이미 9090 port를 관리자 페이지로 사용 중이어서 port 충돌이 발생했습니다.
이를 해결하기 위해 Prometheus port를 9091로 변경하였습니다.

**1-1. Prometheus 서비스 파일 수정**
```bash
sudo vi /etc/systemd/system/prometheus.service
```
```text
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries \
--web.listen-address=0.0.0.0:9091 \ <- 추가된 부분
[Install]
WantedBy=multi-user.target
```

**1-2. prometheus.yml 파일 수정**
```bash
sudo vi /etc/prometheus/prometheus.yml
```
```text
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9091"] <-- 수정된 부분
```

**1-3. 방화벽 설정**
```bash
sudo firewall-cmd --zone=public --add-port=9091/tcp --permanent
sudo firewall-cmd --reload
```

### 2. Prometheus Permission denied 오류 발생
Prometheus 서비스 등록 중 Permission denied 오류가 발생하여 SELinux 로그를 확인해 본 결과
Prometheus 실행 파일(/usr/local/bin/prometheus)의 SELinux 컨텍스트가
unconfined_u:object_r:user_home_t:s0로 설정되어 있었습니다.
user_home_t는 사용자 홈 디렉터리에 주로 사용되기 때문에
/usr/local/bin 같은 시스템 디렉터리에 있는 실행 파일에는 적합하지 않습니다.

**2-1. SELinux 로그 확인**
```bash
sudo ausearch -m avc -ts recent
```

**2-2. Prometheus 실행 파일의 SELinux 보안 컨텍스트 수정**
```bash
sudo semanage fcontext -a -t bin_t "/usr/local/bin/prometheus"
```

**2-3. 변경 사항 적용**
```bash
sudo restorecon -v /usr/local/bin/prometheus
```

**2-3. 변경 사항 확인**
```bash
ls -Z /usr/local/bin/prometheus 
```




