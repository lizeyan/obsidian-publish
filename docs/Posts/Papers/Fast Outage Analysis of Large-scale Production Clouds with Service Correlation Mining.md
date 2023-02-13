---
share: true
tags:
  - Paper Notes
  - ICSE
  - "2021"
---



> Cloud-based services are surging into popularity in recent years. However, outages, i.e., severe incidents that always impact multiple services, can dramatically affect user experience and incur a severe economic loss. Locating the root cause service, i.e., the service where the propagation of anomaly originates, is a crucial step to mitigate the impact of the outage. In current industrial practice, this is generally performed in a bootstrap manner, which largely depends on human efforts. A candidate service that directly causes the outage is identiﬁed ﬁrst, and the suspected root cause may be traced back manually from service to service during diagnosis until the actual root cause is found. Unfortunately, production clouds typically contain a large number of interdependent services. Such a manual root cause analysis is often time-consuming and labor-intensive. In this work, we propose COT, the ﬁrst correlation-based outage triage approach by constructing a global view of service correlations. COT mines the correlations of the performance indicators collected from hundreds of services. After learning from historical outages, COT can infer the root cause of emerging ones accordingly. We implement COT and evaluate it on a real-world dataset containing one year of data collected from a production Cloud A, one of the representative cloud computing platforms around the world. Our experimental results show that COT can reach a triage accuracy of 82.1∼83.5%, which outperforms the state-of-the-art triage approach by 28.0∼29.7%.
>