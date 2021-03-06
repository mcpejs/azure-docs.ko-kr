---
title: SQL-Azure Monitor 로그 쿼리 참고 자료 | Microsoft Docs
description: Azure Monitor에서 로그 쿼리를 작성하는 데 SQL에 익숙한 사용자를 위한 도움말입니다.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/21/2018
ms.openlocfilehash: c76ab145fd2fdd077075b345ecac9c6a473f2369
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75365192"
---
# <a name="sql-to-azure-monitor-log-query-cheat-sheet"></a>SQL-Azure Monitor 로그 쿼리 참고 자료 

아래 표는 SQL에 익숙한 사용자가 Azure Monitor에서 로그 쿼리를 작성하기 위한 Kusto 쿼리 언어를 알아보는 데 도움이 됩니다. Azure Monitor 로그 쿼리에서 일반적인 시나리오와 이에 해당하는 시나리오를 해결하기 위한 T-SQL 명령을 살펴보십시오.

## <a name="sql-to-azure-monitor"></a>SQL-Azure Monitor

설명                             |SQL 쿼리                                                                                          |Azure Monitor 로그 쿼리
----------------------------------------|---------------------------------------------------------------------------------------------------|----------------------------------------
테이블에서 모든 데이터 선택            |`SELECT * FROM dependencies`                                                                       |<code>dependencies</code>
테이블에서 특정 열 선택    |`SELECT name, resultCode FROM dependencies`                                                        |<code>dependencies <br>&#124; project name, resultCode</code>
테이블에서 100개의 레코드 선택         |`SELECT TOP 100 * FROM dependencies`                                                               |<code>dependencies <br>&#124; take 100</code>
Null 평가                         |`SELECT * FROM dependencies WHERE resultCode IS NOT NULL`                                          |<code>dependencies <br>&#124; where isnotnull(resultCode)</code>
문자열 비교: 같음             |`SELECT * FROM dependencies WHERE name = "abcde"`                                                  |<code>dependencies <br>&#124; where name == "abcde"</code>
문자열 비교: 하위 문자열            |`SELECT * FROM dependencies WHERE name like "%bcd%"`                                                   |<code>dependencies <br>&#124; where name contains "bcd"</code>
문자열 비교: 와일드 카드             |`SELECT * FROM dependencies WHERE name like "abc%"`                                                |<code>dependencies <br>&#124; where name startswith "abc"</code>
날짜 비교: 최근 1일             |`SELECT * FROM dependencies WHERE timestamp > getdate()-1`                                         |<code>dependencies <br>&#124; where timestamp > ago(1d)</code>
날짜 비교: 날짜 범위             |`SELECT * FROM dependencies WHERE timestamp BETWEEN '2016-10-01' AND '2016-11-01'`                 |<code>dependencies <br>&#124; where timestamp between (datetime(2016-10-01) .. datetime(2016-10-01))</code>
부울 비교                      |`SELECT * FROM dependencies WHERE !(success)`                                                      |<code>dependencies <br>&#124; where success == "False" </code>
정렬                                    |`SELECT name, timestamp FROM dependencies ORDER BY timestamp asc`                                  |<code>dependencies <br>&#124; order by timestamp asc </code>
Distinct                                |`SELECT DISTINCT name, type  FROM dependencies`                                                    |<code>dependencies <br>&#124; summarize by name, type </code>
그룹화, 집계                   |`SELECT name, AVG(duration) FROM dependencies GROUP BY name`                                       |<code>dependencies <br>&#124; summarize avg(duration) by name </code>
열 별칭, 확장                  |`SELECT operation_Name as Name, AVG(duration) as AvgD FROM dependencies GROUP BY name`             |<code>dependencies <br>&#124; summarize AvgD=avg(duration) by operation_Name <br>&#124; project Name=operation_Name, AvgD</code>
측정별 상위 n 레코드                |`SELECT TOP 100 name, COUNT(*) as Count FROM dependencies GROUP BY name ORDER BY Count asc`        |<code>dependencies <br>&#124; summarize Count=count() by name <br>&#124; top 100 by Count asc</code>
Union                                   |`SELECT * FROM dependencies UNION SELECT * FROM exceptions`                                        |<code>union dependencies, exceptions</code>
통합: 조건 사용                  |`SELECT * FROM dependencies WHERE value > 4 UNION SELECT * FROM exceptions WHERE value < 5`                |<code>dependencies <br>&#124; where value > 4 <br>&#124; union (exceptions <br>&#124; where value < 5)</code>
Join                                    |`SELECT * FROM dependencies JOIN exceptions ON dependencies.operation_Id = exceptions.operation_Id`|<code>dependencies <br>&#124; join (exceptions) on operation_Id == operation_Id</code>


## <a name="next-steps"></a>다음 단계

- [Azure Monitor에서 로그 쿼리 작성에 대한 강의를 진행합니다.](get-started-queries.md)
