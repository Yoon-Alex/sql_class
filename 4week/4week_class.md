# 4주차 

1-1. 2018년도 2월 아래 내용을 DAILY_REPORT_{이니셜}  INSERT 하는 프로시저를 만들어주세요.  

``` sql   
SELECT  YMD 
        , MAX(UV) UV
        , MAX(MAU) MAU
        , MAX(CUST) CUST
        , MAX(SALES_AMT) SALES_AMT
        , MAX(PAYMT_AMT) PAYMT_AMT
        , MAX(JOIN_MEM) JOIN_MEM
        , MAX(WITHDR_MEM) WITHDR_MEM
        , MAX(REST_MEM) REST_MEM
  FROM  ( 
        SELECT  '20180201' YMD
                , COUNT(DISTINCT CASE WHEN YMD = '20180201' THEN MEM_NO END) UV
                , COUNT(DISTINCT MEM_NO) MAU 
                , 0 CUST        
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM
          FROM  FS_WLOG 
         WHERE  1=1 
                AND YM = LEFT('20180201',6)
                AND YMD <= '20180201' 
         GROUP
            BY  '20180201'  
            
        UNION ALL 
        
        SELECT  YMD 
                , 0 UV        
                , 0 MAU 
                , COUNT(DISTINCT MEM_NO) CUST
                , SUM(SALES_AMT) SALES_AMT 
                , SUM(PAYMT_AMT) PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM        
          FROM  FS_SALE
         WHERE  1=1 
                AND YM = LEFT('20180201',6)
                AND YMD = '20180201'
         GROUP
            BY  YMD 
        
        UNION ALL 
        
        SELECT  REPLACE(JOIN_DT, '-', '') YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , COUNT(DISTINCT MEM_NO) JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM     
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND REPLACE(JOIN_DT, '-', '') = '20180201'
         GROUP
            BY  REPLACE(JOIN_DT, '-', '')
            
        UNION ALL 
        
        SELECT  WITHDR_DT YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , COUNT(DISTINCT MEM_NO) WITHDR_MEM
                , 0 REST_MEM     
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND WITHDR_DT = '20180201'
         GROUP
            BY  WITHDR_DT
            
        UNION ALL 
        
        SELECT  REST_DT YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , COUNT(DISTINCT MEM_NO) REST_MEM
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND REST_DT = '20180201'
         GROUP
            BY  REST_DT    
        ) A 
 GROUP
    BY  YMD 
;            
``` 

1-2. 테이블 생성 후, TRUNCATE 수행
``` sql
TRUNCATE TABLE DAILY_REPORT_JBY
;
CREATE TABLE DAILY_REPORT_JBY AS ( 
SELECT  YMD 
        , MAX(UV) UV
        , MAX(MAU) MAU
        , MAX(CUST) CUST
        , MAX(SALES_AMT) SALES_AMT
        , MAX(PAYMT_AMT) PAYMT_AMT
        , MAX(JOIN_MEM) JOIN_MEM
        , MAX(WITHDR_MEM) WITHDR_MEM
        , MAX(REST_MEM) REST_MEM
  FROM  ( 
        SELECT  '20180201' YMD
                , COUNT(DISTINCT CASE WHEN YMD = '20180201' THEN MEM_NO END) UV
                , COUNT(DISTINCT MEM_NO) MAU 
                , 0 CUST        
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM
          FROM  FS_WLOG_MST 
         WHERE  1=1 
                AND LEFT(YMD,6) = LEFT('20180201',6)
                AND YMD <= '20180201' 
         GROUP
            BY  '20180201'  
            
        UNION ALL 
        
        SELECT  YMD 
                , 0 UV        
                , 0 MAU 
                , COUNT(DISTINCT MEM_NO) CUST
                , SUM(SALES_AMT) SALES_AMT 
                , SUM(PAYMT_AMT) PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM        
          FROM  FS_SALE
         WHERE  1=1 
                AND YM = LEFT('20180201',6)
                AND YMD = '20180201'
         GROUP
            BY  YMD 
        
        UNION ALL 
        
        SELECT  REPLACE(JOIN_DT, '-', '') YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , COUNT(DISTINCT MEM_NO) JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM     
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND REPLACE(JOIN_DT, '-', '') = '20180201'
         GROUP
            BY  REPLACE(JOIN_DT, '-', '')
            
        UNION ALL 
        
        SELECT  WITHDR_DT YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , COUNT(DISTINCT MEM_NO) WITHDR_MEM
                , 0 REST_MEM     
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND WITHDR_DT = '20180201'
         GROUP
            BY  WITHDR_DT
            
        UNION ALL 
        
        SELECT  REST_DT YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , COUNT(DISTINCT MEM_NO) REST_MEM
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND REST_DT = '20180201'
         GROUP
            BY  REST_DT    
        ) A 
 GROUP
    BY  YMD 
)
```     
 
1-3. 프로시저 생성.
``` sql
CREATE PROCEDURE FSN.SP_DAILY_REPORT_JBY(
    IN V_STD_DT VARCHAR(8) -- 매개변수 
)
BEGIN    
    
DELETE 
  FROM  FSN.DAILY_REPORT_JBY
 WHERE  YMD = V_STD_DT
;
COMMIT; 

INSERT INTO FSN.DAILY_REPORT_JBY
SELECT  YMD 
        , MAX(UV) UV
        , MAX(MAU) MAU
        , MAX(CUST) CUST
        , MAX(SALES_AMT) SALES_AMT
        , MAX(PAYMT_AMT) PAYMT_AMT
        , MAX(JOIN_MEM) JOIN_MEM
        , MAX(WITHDR_MEM) WITHDR_MEM
        , MAX(REST_MEM) REST_MEM
  FROM  ( 
        SELECT  V_STD_DT YMD
                , COUNT(DISTINCT CASE WHEN YMD = V_STD_DT THEN MEM_NO END) UV
                , COUNT(DISTINCT MEM_NO) MAU 
                , 0 CUST        
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM
          FROM  FS_WLOG_MST 
         WHERE  1=1 
                AND LEFT(YMD,6) = LEFT(V_STD_DT,6)
                AND YMD <= V_STD_DT 
         GROUP
            BY  V_STD_DT  
            
        UNION ALL 
        
        SELECT  YMD 
                , 0 UV        
                , 0 MAU 
                , COUNT(DISTINCT MEM_NO) CUST
                , SUM(SALES_AMT) SALES_AMT 
                , SUM(PAYMT_AMT) PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM        
          FROM  FS_SALE
         WHERE  1=1 
                AND YM = LEFT(V_STD_DT,6)
                AND YMD = V_STD_DT
         GROUP
            BY  YMD 
        
        UNION ALL 
        
        SELECT  REPLACE(JOIN_DT, '-', '') YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , COUNT(DISTINCT MEM_NO) JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM     
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND REPLACE(JOIN_DT, '-', '') = V_STD_DT
         GROUP
            BY  REPLACE(JOIN_DT, '-', '')
            
        UNION ALL 
        
        SELECT  WITHDR_DT YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , COUNT(DISTINCT MEM_NO) WITHDR_MEM
                , 0 REST_MEM     
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND WITHDR_DT = V_STD_DT
         GROUP
            BY  WITHDR_DT
            
        UNION ALL 
        
        SELECT  REST_DT YMD
                , 0 UV        
                , 0 MAU 
                , 0 CUST
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , COUNT(DISTINCT MEM_NO) REST_MEM
          FROM  FS_MEMBER 
         WHERE  1=1 
                AND REST_DT = V_STD_DT
         GROUP
            BY  REST_DT    
        ) A 
 GROUP
    BY  YMD 
;
COMMIT;
END
```  
    

1-4. FOR문을 활용해서 2월 전체 데이터를 구해주세요. 
``` sql  
-- FOR 문
CREATE PROCEDURE LOOP_TABLE_JBY(
    IN ST_DD varchar(8),
    IN EN_DD varchar(8)    
)
BEGIN
    DECLARE done INTEGER DEFAULT 0; -- 반복문 변수
    DECLARE STD_DD varchar(8); -- 커서에서사용할변수선언

    DECLARE openCursor CURSOR FOR SELECT YMD FROM CM_CLDR WHERE YMD BETWEEN ST_DD AND EN_DD; -- 커서에서사용할 Select 테이블선언
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true; -- 반복문 핸들러 선언
      
    OPEN openCursor; -- 커서오픈
    
    read_loop: LOOP -- 반복문시작
    
        FETCH openCursor INTO STD_DD; -- 커서에서데이터가져옴
        
        IF done THEN
            LEAVE read_loop; -- 반복문종료시조건
        END IF;
        CALL FSN.SP_DAILY_REPORT_JBY(STD_DD);-- 프로시저 선언
        
    END LOOP read_loop;
    
    CLOSE openCursor;

END;  
;     
``` 

5. FOR 문 (프로시저)를 수행해주세요. 
``` sql  
-- 20180201부터 10일까지 데이터 INSERT문 수행. 
CALL LOOP_TABLE_JBY('20180201','20180210')
;     
``` 

2. 2. 2021년 5월 한달간 개인 추천란을 통해서 구매한 매출효과를 보고 싶습니다. 
* UV -> 구매전환인원 
* 구매전환 기준 : 해당 로그를 통해 접속하여 해당일 구매(24시)

``` sql
SELECT  A.YMD
        , COUNT(DISTINCT A.MEM_NO) UV 
        , COUNT(DISTINCT CASE WHEN B.PROD_CD IS NOT NULL THEN A.MEM_NO END) PUR
  FROM  ( 
        SELECT  A.*
          FROM  FSN.FS_WLOG_MST A
         WHERE  WPGE_ID = 11 
                AND YMD BETWEEN '20210501' AND '20210531'
        ) A 
  LEFT
  JOIN  (
        SELECT  *
          FROM  FSN.FS_SALE A
         WHERE  YM = '202105'
                AND OFF_YN = 'N'
        ) B 
    ON  A.YMD = B.YMD 
        AND A.MEM_NO = B.MEM_NO 
        AND A.SRCH_DT <= B.ORDER_DT 
        AND A.PROD_CD = B.PROD_CD 
 GROUP
    BY  A.YMD
;   
```

3. 70501 기획전을 봤던 회원들의 kpi 분석을 해주세요. 기간 : 202107 – 202112
* UV, PV, SV, 구매전환율 
``` sql 
WITH MART AS (     
SELECT  YMD
        , SESSON_ID 
        , MEM_NO 
  FROM  ( 
        SELECT  A.*
          FROM  FSN.FS_WLOG_MST A
         WHERE  1=1 
               AND YMD BETWEEN '20210701' AND '20211231'
                AND PLAN_SQ = '70501'
        ) A 
 GROUP
    BY  YMD
        , SESSON_ID 
        , MEM_NO 
) 
SELECT  A.YMD
        , COUNT(DISTINCT A.MEM_NO) UV 
        , COUNT(DISTINCT A.SESSON_ID) SV
        , SUM(1) PV
        , COUNT(DISTINCT CASE WHEN B.PROD_CD IS NOT NULL THEN A.MEM_NO END) PUR
  FROM  ( 
        SELECT  A.*
          FROM  FSN.FS_WLOG_MST A 
          JOIN  MART B 
            ON  A.SESSON_ID = B.SESSON_ID
                AND A.MEM_NO = B.MEM_NO 
        ) A 
  LEFT
  JOIN  (
        SELECT  *
          FROM  FSN.FS_SALE A
         WHERE  YM BETWEEN '202107' AND '202112' 
                AND OFF_YN = 'N'
        ) B 
    ON  A.YMD = B.YMD 
        AND A.MEM_NO = B.MEM_NO 
        AND A.SRCH_DT <= B.ORDER_DT 
        AND A.PROD_CD = B.PROD_CD         
 GROUP
    BY  A.YMD
; 

```

4-1. 테이블 구조 짜기. 
``` sql 
CREATE TABLE FSN.DAILY_REPORT_JBY AS (
SELECT  '20210101' YMD
        , COUNT(DISTINCT CASE WHEN YMD = '20210101' THEN MEM_NO END) UV
        , COUNT(DISTINCT MEM_NO) MAU
  FROM  ( 
        SELECT  A.*
          FROM  FSN.FS_WLOG_MST A         
         WHERE  1=1 
                AND LEFT(YMD, 6) = LEFT('20210101', 6)
                AND YMD <= '20210101'
        ) A 
 GROUP
    BY  '20210101'
)
;
```


4-2. 프로시저
``` sql 
CREATE PROCEDURE FSN.SP_MAU_JBY(
    IN V_STD_DT VARCHAR(8) -- 매개변수 
)
BEGIN    
    
DELETE 
  FROM  FSN.DAILY_REPORT_JBY
 WHERE  YMD = V_STD_DT
;
COMMIT; 

INSERT INTO FSN.DAILY_REPORT_JBY
SELECT  V_STD_DT YMD
        , COUNT(DISTINCT CASE WHEN YMD = V_STD_DT THEN MEM_NO END) UV
        , COUNT(DISTINCT MEM_NO) MAU
  FROM  ( 
        SELECT  A.*
          FROM  FSN.FS_WLOG_MST A         
         WHERE  1=1 
                AND LEFT(YMD, 6) = LEFT(V_STD_DT, 6)
                AND YMD <= V_STD_DT
        ) A 
 GROUP
    BY  V_STD_DT

;
COMMIT;
END; 
```

4-3. for문 프로시저
``` sql 
CREATE PROCEDURE LOOP_TABLE_JBY(
    IN ST_DD varchar(8),
    IN EN_DD varchar(8)    
)
BEGIN
    DECLARE done INTEGER DEFAULT 0; -- 반복문 변수
    DECLARE STD_DD varchar(8); -- 커서에서사용할변수선언

    DECLARE openCursor CURSOR FOR SELECT DS FROM CM_CLDR WHERE DS BETWEEN ST_DD AND EN_DD; -- 커서에서사용할 Select 테이블선언
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true; -- 반복문 핸들러 선언
      
    OPEN openCursor; -- 커서오픈
    
    read_loop: LOOP -- 반복문시작
        FETCH openCursor INTO STD_DD; -- 커서에서데이터가져옴
        
        IF done THEN
            LEAVE read_loop; -- 반복문종료시조건
        END IF;
        CALL FSN.SP_DAILY_REPORT_JBY(STD_DD);-- 프로시저 선언
        
    END LOOP read_loop;
    
    CLOSE openCursor;

END;  
;     
```

4-4. 20180201부터 10일까지 데이터 INSERT문 수행. 
``` sql 
CALL LOOP_TABLE_JBY('20210101','20210131')
```
