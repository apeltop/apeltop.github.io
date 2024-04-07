---
title: GCP Cloud BigQuery Invalid table-valued function EXTERNAL_QUERY Failed to connect 에러 해결
author: sunshine@ptokos.com
categories: [gcp]
---

## 해결 방법
> Service Account 에서 Cloud SQL Client 권한을 추가

## 문제 상황
BigQuery 에서 Cloud SQL 에 연결 한 후 쿼리를 실행하려고하니 _Invalid table-valued function EXTERNAL_QUERY Failed to connect to the Cloud SQL instance_ 이 발생

![1.png](/assets/img/gcp/gcp-bigquery-connect-to-cloudsql-error-1.png)

![2.png](/assets/img/gcp/gcp-bigquery-connect-to-cloudsql-error-2.png)

## 해결 과정
Connection Info 에 있는 Service Account 를 확인한다.

![3.png](/assets/img/gcp/gcp-bigquery-connect-to-cloudsql-error-3.png)

IAM & Admin 에서 해당 Service Account 에 Cloud SQL Client 권한을 추가한다.

![4.png](/assets/img/gcp/gcp-bigquery-connect-to-cloudsql-error-4.png)
