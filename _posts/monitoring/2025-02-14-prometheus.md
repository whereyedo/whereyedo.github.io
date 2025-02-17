---
layout: post
title: CentOS에 Prometheus 설치하기
date: 2025-02-14 14:29:00 +0900
categories: [Monitoring]
tags: [monitoring, prometheus]
description: CentOS에 Prometheus를 설치하여 서버 모니터링하기
toc: true
math: true
mermaid: true
---
## Prometheus란?
Prometheus는 오픈소스 시스템 모니터링 및 경보 툴로, 시계열 데이터베이스(TSDB)를 기반으로 동작합니다. 
클라우드 네이티브 애플리케이션 및 서버의 성능을 모니터링하고 알림을 설정하는 데 유용합니다.

## Prometheus 주요 특징
- 다차원 데이터 모델: key-value 형태의 레이블(Label)을 사용하여 시계열 데이터를 저장합니다.
- 효율적인 데이터 수집: 푸시 방식이 아닌 풀(Pull) 방식으로 메트릭을 수집하며, 다양한 Exporter를 통해 데이터를 가져올 수 있습니다.
- 강력한 쿼리 언어 (PromQL): Prometheus 자체 쿼리 언어인 PromQL을 사용하여 데이터를 분석하고, 시각화 및 경고 설정을 할 수 있습니다.
- 독립적 운영 가능: 분산 저장소가 필요 없으며, 단일 바이너리로 실행 가능하여 유지보수가 용이합니다.
- 다양한 통합 지원: Grafana, Alertmanager, Kubernetes 등과 쉽게 연동할 수 있습니다.
- 알림 시스템 제공: 특정 조건에 따라 Alertmanager와 함께 알람을 설정할 수 있습니다.

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

**1-2. 내용 추가**
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




