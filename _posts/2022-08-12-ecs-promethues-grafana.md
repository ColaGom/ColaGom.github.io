---
layout: post
title: "Spring on AWS ECS Fargate, Prometheus, Grafana, ECS discovery service를 활용한 모니터링 환경 구축"
categories: [Dev, DevOps]
tags: [aws, ecs, prometheus, grafana, monitoring]

---

# 들어가며

본 글에서는 AWS ECS에 대한 모니터링 시스템을 Prometheus, Grafana를 활용하여 구축하는 방법에 대해 정리한다.

> ECS에 포함되는 많은 container중 spring boot micrometer(prometheus-registry)에 대한 모니터링을 구축하는 방법에 대해서만 정리한다.
>

# 사전요구사항

- prometheus, grafana에 대한 기본지식
- Spring boot application on **ECS fargate**
  - 2~5 instance
  - **micrometer** 설정 작업완료

# 문제점

- spring boot application이 포함된 service는 ELB를 통해 외부접근이 가능하다.
- 동일한 VPC에 monitoring instance를 따로 만들어서 구동중인 모든 **spring instance에** 대한 **prometheus metric**을 수집하고자 한다면 어떻게 구현해야될까?

> 단순히 실행중인 모든 **spring container**의 private ip를 직접 **prometheus static_config** 값으로 사용하면 **ECS 작업**이 재실행되면 private ip는 동적으로 재할당되므로 지속적인 수집이 불가능하다.
>

# ECS Discovery service?

단순 EC2 instance의 경우 **EC2 Discovery service**를 활용하여 원하는 EC2 instance 목록을 획득할 수 있었는데 ECS의 fargate instance는 딱히 방법이 떠오르지않아 리서치를 하다가 발견한 서비스!!!

설정값에 따라 원하는 **ECS container목록**을 획득 할 수 있으며 이미 dockerhub에 image도 배포 되어있었다.

# 미리보기

![preview](/assets/img/220812-1-1.png)

1. **ECS discovery**를 사용해서 **prometheus targets**를 획득하여 prometheus scrape format **file_sd_configs** format으로 저장
2. prometheus server에서는 획득한 promethus targets에 대한 지표를 수집한다.
3. grafana에 prometheus datasource를 등록하고 원하는 dashboard를 통해 원하는 지표를 확인한다.

# 구축

> EC2 환경에서 docker-compose를 활용하여 작업
>

### EC2

- 수집대상 클러스터와 동일한 VPC에 위치
  - 또는, VPC 피어링 설정(서로 다른 VPC에 위치한 private ip에 접근)이 되어있어야 한다.
- 보안그룹
  - Grafana(3000), Promethues(9090) port inbound open

### EC2 IAM role 설정

- **ECS discovery service**에 필요하며 config를 통해 Access ID, Key값을 설정해도 가능.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ecs:Describe*",
                "ecs:List*"
            ],
            "Resource": "*"
        }
    ]
}
```

### Docker compose yml

```yaml
version: '3.3'

networks:
  monitor-net:
    driver: bridge

services:
  ecs-discovery:
    image: tkgregory/prometheus-ecs-discovery
    volumes:
      - ./output:/output
    environment:
      - AWS_REGION=$AWS_REGION
    command:
      - '--config.write-to=/output/ecs_file_sd.yml'
      - '--config.cluster=$AWS_CLUSTER'
    networks:
      - monitor-net

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus:/etc/prometheus
      - ./output:/output
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    restart: unless-stopped
    ports:
      - "9090:9090"
    expose:
      - 9090
    networks:
      - monitor-net

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    expose:
      - 3000
    networks:
      - monitor-net
```

### prometheus yml

```yaml
global:
  scrape_interval:     5s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'ecs-discovery'
    file_sd_configs:
      - files:
          - /output/ecs_file_sd.yml
        refresh_interval: 1m
```

# Quick start

[https://github.com/ColaGom/docker-monitoring-sample](https://github.com/ColaGom/docker-monitoring-sample)

```bash
git clone https://github.com/ColaGom/docker-monitoring-sample
cd docker-monitoring-sample
vim .env //setup your env
docker-compose up
```

# 참조

[https://github.com/teralytics/prometheus-ecs-discovery](https://github.com/teralytics/prometheus-ecs-discovery)

[Configuration | Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)
