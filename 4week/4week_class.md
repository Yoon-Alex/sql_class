# 4주차 

1. 2018년도 2월 아래 내용을 DAILY_REPORT_{이니셜}  INSERT 하는 프로시저를 만들어주세요.  

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
        SELECT  YMD 
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
                AND YM = '201802'
                AND YMD <= '20180201' 
         GROUP
            BY  YMD 
            
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
                AND YM = '201802'
                AND YMD = '20180201'
         GROUP
            BY  YMD 
        
        UNION ALL 
        
        SELECT  DATE_FORMAT(STR_TO_DATE(JOIN_DT, '%Y-%m-%d'), '%Y%m%d') YMD
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
                AND DATE_FORMAT(STR_TO_DATE(JOIN_DT, '%Y-%m-%d'), '%Y%m%d') = '20180201'
         GROUP
            BY  DATE_FORMAT(STR_TO_DATE(JOIN_DT, '%Y-%m-%d'), '%Y%m%d') 
            
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

2. 테이블 생성 후, TRUNCATE 수행
``` sql
TRUNCATE TABLE DAILY_REPORT_JBY

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
        SELECT  YMD 
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
                AND YM = '201802'
                AND YMD <= '20180201' 
         GROUP
            BY  YMD 
            
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
                AND YM = '201802'
                AND YMD = '20180201'
         GROUP
            BY  YMD 
        
        UNION ALL 
        
        SELECT  '20180201' YMD
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
            BY  '20180201'
            
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
 
3. 프로시저 생성.
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
        , CURRENT_DATE LOAD_DT
  FROM  ( 
        SELECT  YMD 
                , COUNT(DISTINCT CASE WHEN YMD = V_STD_DT THEN MEM_NO END) UV
                , COUNT(DISTINCT MEM_NO) MAU 
                , 0 CUST        
                , 0 SALES_AMT 
                , 0 PAYMT_AMT 
                , 0 JOIN_MEM
                , 0 WITHDR_MEM
                , 0 REST_MEM
          FROM  FS_WLOG 
         WHERE  1=1 
                AND YM = '201802'
                AND YMD <= V_STD_DT 
         GROUP
            BY  YMD 
            
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
                AND YM = '201802'
                AND YMD = V_STD_DT
         GROUP
            BY  YMD 
        
        UNION ALL 
        
        SELECT  V_STD_DT YMD
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
            BY  V_STD_DT
            
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
    

4. FOR문을 활용해서 2월 전체 데이터를 구해주세요. 
``` sql  
-- FOR 문
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

5. FOR 문 (프로시저)를 수행해주세요. 
``` sql  
-- 20180201부터 10일까지 데이터 INSERT문 수행. 
CALL LOOP_TABLE_JBY('20180201','20180210')
;     
``` 
