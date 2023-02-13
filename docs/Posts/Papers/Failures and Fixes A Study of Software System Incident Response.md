---
share: true
tags:
  - Paper Notes
  - ICSME
  - "2020"
---


## Introduction

通过对interview和public的incident response document的分析，得到了一些observation。关于故障是如何发生，检测，分析和处理的。

## Thinking and Conclusion

故障如何发生

Observation 1: Test cases often fail to detect defects that lead to incidents only when (possibly rare) combinations of events or system states coincide

Observation 2: Testing environments and other preproduction environments often do not capture all aspects of the production environment.

Observation 3: When scaling limits are not well known, tested or monitored for they are discovered when they are exceeded

Observation 4: Conﬁguration changes are just as risky as code changes, but are often not tested and deployed with the same care

检测

Observation 5: Generic catch all monitoring and notiﬁcation are important but tend to be trailing metrics, leading to late detection.

Observation 6: Pre-determined, threshold based detection is fragile and incomplete.

Observation 7: Monitoring, notiﬁcation and other support systems may not themselves be as well tested or monitored as primary systems

排障

Observation 9: Architectural complexity has an operational (including investigative) cost

故障恢复

Observation 10: Addressing the original root cause is not always sufﬁcient mitigation; the mitigation does not always cascade the way the failure did.

Observation 11: Changes made in the context of incident response have the potential to make issues worse (and are made with fewer precautions than typical).