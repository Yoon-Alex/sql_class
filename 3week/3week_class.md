
# 3주차 

0. 총주문(취반품 제외한) 기준 첫구매 데이터를 구하고 싶습니다.  
__MIN( )__ 집계 함수를 이용하여 첫구매 리스트를 추출하는 쿼리를 작성해 주세요. 
``` sql   
WITH FST_ORD AS (   
SELECT  MEM_NO 
        , MIN(YMD) MN_ORD_DT
  FROM  FS_SALE     
 GROUP
    BY  MEM_NO
)
SELECT  A.*
  FROM  FS_SALE A 
  JOIN  FST_ORD B 
    ON  A.MEM_NO = B.MEM_NO 
        AND A.YMD = MN_ORD_DT 
;            
``` 


1. 2018년도 첫구매로 진행했던 오프라인 상품 순위 100위와 온라인 상품 순위 100위를 함께 확인하고 싶어요.   
첫구매는 총주문 기준 / UNION ALL 을 활용해서 문제 해결  

``` sql
WITH FST_ORD AS (   
-- 첫구매 리스트만 사용하고 싶다면.  
SELECT  MEM_NO 
        , MIN(YMD) MN_ORD_DT
  FROM  FS_SALE     
 GROUP
    BY  MEM_NO
)   
SELECT  A.*     
        , '오프라인'    
  FROM  (   
        SELECT  PROD_CD     
                , SUM(SALES_QTY) QTY    
                , ROW_NUMBER() OVER(ORDER BY SUM(SALES_QTY) DESC) RNK   
          FROM  FS_SALE A   
          JOIN  FST_ORD B   
            ON  A.MEM_NO = B.MEM_NO     
                AND A.YMD = B.MN_ORD_DT   
         WHERE  1=1     
                AND YY = '2018'
                AND OFF_YN = 'Y'    
         GROUP  
            BY  PROD_CD 
        ) A 
 WHERE  RNK <= 100
    
UNION ALL   
    
SELECT  A.*     
        , '온라인' 
  FROM  (   
        SELECT  PROD_CD     
                , SUM(SALES_QTY) QTY    
                , ROW_NUMBER() OVER(ORDER BY SUM(SALES_QTY) DESC) RNK   
          FROM  FS_SALE A   
          JOIN  FST_ORD B   
            ON  A.MEM_NO = B.MEM_NO     
                AND A.YMD = B.MN_ORD_DT   
         WHERE  1=1     
                AND YY = '2018'
                AND OFF_YN = 'N'    
         GROUP  
            BY  PROD_CD 
        ) A 
 WHERE  RNK <= 100
;  
``` 

2. 10위 이후의 상품은 ‘ETC’로 표현하고 싶습니다.    
``` sql
WITH FST_ORD AS (   
-- 첫구매 리스트만 사용하고 싶다면.  
SELECT  MEM_NO 
        , MIN(YMD) MN_ORD_DT
  FROM  FS_SALE     
 GROUP
    BY  MEM_NO
)   
SELECT  CASE WHEN RNK > 10 THEN 'ETC' ELSE PROD_CD END PROD_CD
        , CASE WHEN RNK > 10 THEN 'OVER10' ELSE RNK END RNK
        , SUM(QTY) QTY 
        , '오프라인'    
  FROM  (   
        SELECT  PROD_CD     
                , SUM(SALES_QTY) QTY    
                , ROW_NUMBER() OVER(ORDER BY SUM(SALES_QTY) DESC) RNK   
          FROM  FS_SALE A   
          JOIN  FST_ORD B   
            ON  A.MEM_NO = B.MEM_NO     
                AND A.YMD = B.MN_ORD_DT   
         WHERE  1=1     
                AND YY = '2018'
                AND OFF_YN = 'Y'    
         GROUP  
            BY  PROD_CD 
        ) A 
 WHERE  RNK <= 100
 GROUP
 	BY  CASE WHEN RNK > 10 THEN 'ETC' ELSE PROD_CD END 
        , CASE WHEN RNK > 10 THEN 'OVER10' ELSE RNK END 
        
UNION ALL   
    
SELECT  CASE WHEN RNK > 10 THEN 'ETC' ELSE PROD_CD END PROD_CD
        , CASE WHEN RNK > 10 THEN 'OVER10' ELSE RNK END RNK
        , SUM(QTY) QTY 
        , '온라인' 
  FROM  (   
        SELECT  PROD_CD     
                , SUM(SALES_QTY) QTY    
                , ROW_NUMBER() OVER(ORDER BY SUM(SALES_QTY) DESC) RNK   
          FROM  FS_SALE A   
          JOIN  FST_ORD B   
            ON  A.MEM_NO = B.MEM_NO     
                AND A.YMD = B.MN_ORD_DT   
         WHERE  1=1     
                AND YY = '2018'
                AND OFF_YN = 'N'    
         GROUP  
            BY  PROD_CD 
        ) A 
 WHERE  RNK <= 100
 GROUP
 	BY  CASE WHEN RNK > 10 THEN 'ETC' ELSE PROD_CD END 
        , CASE WHEN RNK > 10 THEN 'OVER10' ELSE RNK END 
```     
 
3. 2019년 1월에 첫구매로 구매한 상품 리스트를 상품 카테고리 별로 나타내주세요. 어떤 카테고리의 상품이 잘팔렸나요? FS_PROD    
``` sql
WITH FST_ORD AS (   
SELECT  MEM_NO 
        , MIN(YMD) MN_ORD_DT
  FROM  FS_SALE     
 GROUP
    BY  MEM_NO      
)
SELECT  B.CL_NM     
        , PROD_CTGR1    
        , SUM(QTY) QTY  
  FROM  (   
        SELECT  PROD_CD     
                , SUM(SALES_QTY) QTY    
          FROM  FS_SALE A   
          JOIN  FST_ORD B   
            ON  A.MEM_NO = B.MEM_NO     
                AND A.YMD = B.MN_ORD_DT   
         WHERE  1=1     
                AND YM = '201901'   
         GROUP  
            BY  PROD_CD 
        ) A     
  LEFT  
  JOIN  FS_PROD B   
    ON  A.PROD_CD = B.PROD_CD   
 GROUP  
    BY  B.CL_NM     
        , PROD_CTGR1    
 ORDER  
    BY  3 DESC  
```  
    

4. '03479f39279fa516','edd1c63f5cfac0dc','04019ccd24df042f' 상품은 당시 첫구매 쿠폰을 제공한 상품입니다.   
이 상품을 해당월(2019년 01월) 구매했던 회원들이 그 후 1년간 (2019년 2월 ~ 2020년 1월)까지의 거래액, 구매객을 확인하고 싶습니다.    
즉, 2017년 01월에 시행한 캠페인의 효과를 LTV의 개념으로 측정하고 싶습니다.     

``` sql    
SELECT  COUNT(DISTINCT A.MEM_NO)    
        , SUM(SALES_AMT)    
  FROM  FS_SALE A   
  JOIN  (       
        SELECT  DISTINCT MEM_NO 
          FROM  FS_SALE     
         WHERE  1=1     
                AND YM = '201901'   
                AND PROD_CD IN ('03479f39279fa516','edd1c63f5cfac0dc','04019ccd24df042f')   
        ) B     
    ON  A.MEM_NO = B.MEM_NO     
 WHERE  YM BETWEEN '201902' AND '202001'
;     
``` 

5. VVIP 회원의 2018년도 각 회원의 구매액과 구매건수를 구해주세요.  
분석은?    
``` sql
SELECT  A.MEM_NO    
        , COUNT(DISTINCT ORDER_NO)  
        , SUM(SALES_AMT)    
        , SUM(PAYMT_AMT)    
        , SUM(SALES_QTY)            
  FROM  FS_SALE A   
  JOIN  FS_VVIP B   
    ON  A.MEM_NO = B.MEM_NO     
 WHERE  YM >= '201801'  
 GROUP  
    BY  A.MEM_NO  
``` 
    
6. 지금 브랜드의 첫구매 이후 재구매 전환이 일어나도록 유도하려고 합니다. 
기존 회원의 첫구매 후 두번째 구매까지 걸리는 평균 소요시간(일)을 연령별로 나타내어 캠페인 기획에 활용하려 합니다. 
FS_ORD_RNK : 주문번호, 구매자를 구매 순서(RNK)에 따라서 정리한 테이블  
위 테이블을 활용해서 mem_no 별 2번째 구매까지 걸린 시간을 나타내는 데이터를 추출해주세요. 
``` sql    
SELECT  A.* 
        , TIMESTAMPDIFF(DAY, FST_DT, SEC_DT) N2N1
  FROM  (   
        SELECT  A.MEM_NO    
                , MAX(CASE WHEN ORD_RNK = 1 THEN ORDER_DT END) FST_DT   
                , MAX(CASE WHEN ORD_RNK = 2 THEN ORDER_DT END) SEC_DT   
          FROM  FS_ORD_RNK A     
          JOIN  (   
                SELECT  MEM_NO  
                        , COUNT(DISTINCT ORDER_NO) ORD_CNT  
                  FROM  FS_SALE        
                 GROUP  
                    BY  MEM_NO  
                HAVING  COUNT(DISTINCT ORDER_NO) >= 2   
                ) B     
            ON  A.MEM_NO = B.MEM_NO     
         WHERE  ORD_RNK <= 2    
         GROUP  
            BY  A.MEM_NO    
        ) A    
```     

7. 2021년도 1월 1일 기준으로 작년(2020년도) 한해 모든 회원의 Recency, Frequency, Monetary을 구해주세요.  
최근성 : 마지막 구매일   
빈도 : 구매빈도   
금액 : 구매금액       
``` sql        
SELECT  MEM_NO  
        , TIMESTAMPDIFF(DAY, MAX(ORDER_DT), STR_TO_DATE('20210101', '%Y%m%d')) R    
        , COUNT(DISTINCT ORDER_NO) F    
        , SUM(PAYMT_AMT) M  
  FROM  FS_SALE     
 WHERE  1=1       
        AND YY = '2020'    
 GROUP  
    BY  MEM_NO  
``` 

8. 7. 위 결과를 활용해서 고객의 Life Time Value (LTV)를 산출하고자 합니다.  
각각 R, F, M의 점수로 순위를 매긴 데이터를 추출하는 쿼리를 작성해주세요.    
``` sql    
SELECT  A.*     
        , ROW_NUMBER() OVER(ORDER BY R ASC) R_RNK  
        , ROW_NUMBER() OVER(ORDER BY F DESC) F_RNK  
        , ROW_NUMBER() OVER(ORDER BY M DESC) M_RNK  
  FROM  (   
        SELECT  MEM_NO  
                , TIMESTAMPDIFF(DAY, MAX(ORDER_DT), STR_TO_DATE('20210101', '%Y%m%d')) R     
                , COUNT(DISTINCT ORDER_NO) F    
                , SUM(PAYMT_AMT) M  
          FROM  FS_SALE     
         WHERE  1=1     
                AND YY = '2020'    
         GROUP  
            BY  MEM_NO  
        ) A  
``` 

9. 홈(2번 페이지)에서 바로 이탈하는 회원들을 대상으로 구매 전환을 유도하기 위해 쿠폰 캠페인을 진행하려고 합니다.  
2020년 2월 1일 홈에서 이탈한 회원 리스트를 추출해주세요.    
``` sql    
SELECT  DISTINCT MEM_NO 
  FROM  (   
        SELECT  A.* 
                , ROW_NUMBER() OVER(PARTITION BY SESSION_ID ORDER BY SRES_NO DESC) RNK  
          FROM  FS_WLOG_MST A   
         WHERE  YM = '20180201'   
        ) A     
 WHERE  1=1     
        AND WPGE_ID = '2'   
        AND RNK = 1     
        AND SRES_NO = '1'
```         

10. 2020년 5월 일자별 DAU를 구해주세요. 
``` sql    
SELECT  SUBSTR(SESSON_ID, 1,8) YMD 
        , COUNT(DISTINCT MEM_NO) DAU 
  FROM  FSN.FS_WLOG_MST
 WHERE  DATE_FORMAT(SRCH_DT, '%Y%m')= '202005'
 GROUP
    BY  SUBSTR(SESSON_ID, 1,8) 
```       


11. 2020년 1월 일자별 PV를 구해주세요.  
``` sql    
SELECT  SUBSTR(SESSON_ID, 1,8) YMD 
        , SUM(1)
  FROM  FSN.FS_WLOG_MST  
 GROUP
    BY  SUBSTR(SESSON_ID, 1,8)
```       


12. 2020년 1월 일자별 UV, PV, SV를 구해주세요.  
``` sql    
SELECT  SUBSTR(SESSON_ID, 1,8) YMD 
        , COUNT(DISTINCT MEM_NO) UV
        , SUM(1) PV 
        , COUNT(DISTINCT SESSON_ID) SV 
  FROM  FSN.FS_WLOG_MST  
 WHERE  DATE_FORMAT(SRCH_DT, '%Y%m')= '202001'
 GROUP
    BY  SUBSTR(SESSON_ID, 1,8)
```       


13. 2020년 1월, 세션 당 페이지에 머문시간을 알고싶습니다. (세션 당 평균 체류시간)
``` sql    
SELECT  COUNT(DISTINCT SESSON_ID) SV 
        , SUM(DURATION) DURATION
        , ROUND(SUM(DURATION) / COUNT(DISTINCT SESSON_ID)) AVG_DUR
  FROM  ( 
        SELECT  TIMESTAMPDIFF(SECOND ,SRCH_DT, AFT_SRCH_DT) DURATION
                , A.*
          FROM  ( 
                SELECT  A.*
                        , LEAD(SRCH_DT, 1) OVER(PARTITION BY SESSON_ID ORDER BY SERS_NO) AFT_SRCH_DT
                  FROM  FSN.FS_WLOG_MST A 
                 WHERE  DATE_FORMAT(SRCH_DT, '%Y%m')= '202001'
                ) A 
        ) A
```       


14. 페이지 당 체류시간, -> 어떤 페이지에서 가장 오래 머물러 있었나요? 
``` sql    
      -- 어떤 페이지에서 가장 오래 머물러 있었나요? 
SELECT  WPGE_NM 
        , SUM(1) PV
        , SUM(DURATION) DURATION
        , ROUND(SUM(DURATION) / SUM(1)) AVG_DUR
  FROM  ( 
        SELECT  TIMESTAMPDIFF(SECOND ,SRCH_DT, AFT_SRCH_DT) DURATION
                , A.*
          FROM  ( 
                SELECT  A.*
                        , LEAD(SRCH_DT, 1) OVER(PARTITION BY SESSON_ID ORDER BY SERS_NO) AFT_SRCH_DT
                  FROM  FSN.FS_WLOG_MST A 
                 WHERE  DATE_FORMAT(SRCH_DT, '%Y%m')= '202001'
                ) A 
        ) A
 GROUP
    BY  WPGE_NM
```       



15. 2020년 한해 어떤 페이지에서 가장 이탈이 많이 발생하는지 확인해주세요 .
``` sql    
SELECT  WPGE_NM 
        , SUM(1)
  FROM  ( 
        SELECT  A.*
                , ROW_NUMBER() OVER(PARTITION BY SESSON_ID ORDER BY SERS_NO DESC) RNK
          FROM  FSN.FS_WLOG_MST A 
         WHERE  DATE_FORMAT(SRCH_DT, '%Y')= '2020'
        ) A         
 WHERE  RNK = 1 
 GROUP
    BY  WPGE_NM 
```       



16. 2020년 5월 기준으로, 아무런 행동을 하지 않고 홈에 접속한 뒤, 바로 이탈한 회원 수(UV)를 구해주세요. 
``` sql    
SELECT  COUNT(DISTINCT MEM_NO) DAU 
        , COUNT(DISTINCT CASE WHEN RNK = 1 AND WPGE_NM = '홈' THEN MEM_NO END) 이탈
        , ROUND(COUNT(DISTINCT CASE WHEN RNK = 1 AND WPGE_NM = '홈' THEN MEM_NO END) / COUNT(DISTINCT MEM_NO), 2) PCT
  FROM  ( 
        SELECT  A.*
                , ROW_NUMBER() OVER(PARTITION BY SESSON_ID ORDER BY SERS_NO DESC) RNK
          FROM  FSN.FS_WLOG_MST A 
         WHERE  DATE_FORMAT(SRCH_DT, '%Y%m')= '202005'
        ) A         
```       
