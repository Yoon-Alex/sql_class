# 집계, JOIN 문제 모음

1. BBY 서비스 2018년도 여성 가입자 추이를 월별로 확인하고 싶습니다.   

#### HINT 
* 조건(WHERE): 여성, 2018년도   
* 집계 기준(GROUP BY): 월별   
* 년도와, 월별 기준은 LEFT함수를 사용하여 계산  

```sql 
SELECT  LEFT(JOIN_DT, 6) YM
        , COUNT(DISTINCT MEM_NO) MEM_CNT
  FROM  BBY.BABY_MEM A
 WHERE  GD = '여자'
        AND LEFT(JOIN_DT, 4) = '2018'
 GROUP  
    BY  LEFT(JOIN_DT, 6) 
;
```

2. BBY / CASE WHEN 구문을 활용해서 각 월마다의 남성, 여성회원 가입자 추이를 확인해주세요. 

#### HINT 
* 조건(WHERE): 없음 
* 집계 기준(GROUP BY): 월별 
```sql
SELECT  LEFT(JOIN_DT, 6) YM
        , COUNT(DISTINCT CASE WHEN GD = '여자' THEN MEM_NO END) FEMALE
        , COUNT(DISTINCT CASE WHEN GD = '남자' THEN MEM_NO END) MALE
  FROM  BBY.BABY_MEM A 
 GROUP
    BY  LEFT(JOIN_DT, 6) 
;
```

3. 2018년도 전체 매출 중에서 GRADE 변수가 'NEW 프리미엄'에 해당하는 회원의 매출 비중이 어느정도 되는지 확인하고 싶습니다. 

2018년도 GRADE 별 매출을 구해주세요. 

#### HINT 
* 조건(WHERE): 2018년도  
* 집계 기준(GROUP BY): GRADE
```sql
SELECT  GRADE
        , SUM(SALES_AMT) SALES_AMT 
  FROM  BBY.BABY_SALES A
 WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018' 
 GROUP 
    BY  GRADE
;
```

4. BABY_PROD 테이블을 SALES 테이블과 JOIN 하여 2018년도에는 각 카테고리(CTGR) 별로 매출을 어느정도 차지하는지 확인하고 싶습니다. 
* 조건(WHERE): 2018년도  

```sql
SELECT  CTGR 
        , SUM(SALES_AMT) AMT
  FROM  BBY.BABY_SALES A 
  LEFT 
  JOIN  BBY.BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018' 
 GROUP 
    BY  CTGR 
;
```


5. BABY_PROD 테이블을 SALES 테이블과 JOIN 하여 2018년도 각 카테고리(CTGR) 별로 주문단가, 상품단가를 구해주세요.  
* 조건(WHERE): 2018년도  

```sql
SELECT  CTGR 
        , SUM(SALES_AMT) AMT
        , COUNT(DISTINCT ORDER_NO) ORD_CNT 
        , SUM(QTY) QTY 
        , SUM(SALES_AMT)/COUNT(DISTINCT ORDER_NO) 주문단가
        , SUM(SALES_AMT)/SUM(QTY) 상품단가         
  FROM  BBY.BABY_SALES A 
  LEFT 
  JOIN  BBY.BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'     
 GROUP 
    BY  CTGR 
;
```

6. BABY_MEM , BABY_SALES 테이블을 활용해서 2018년도 가입월을 기준으로 하는 코호트 분석을 진행해주세요.

* 조건(WHERE): 가입일 = 2018년도 / 구매월 = 2018년도 
* 집계 기준(GROUP BY): 가입월, 구매월 
```sql
SELECT  LEFT(JOIN_DT, 6) JOIN_MM
        , DATE_FORMAT(ORDER_DT, '%Y%m') YM 
        , COUNT(DISTINCT A.MEM_NO) MEM_CNT 
  FROM  BBY.BABY_SALES A
  LEFT 
  JOIN  BBY.BABY_MEM B 
    ON  A.MEM_NO = B.MEM_NO 
 WHERE  1=1 
        AND LEFT(JOIN_DT, 4) = '2018'
        AND DATE_FORMAT(ORDER_DT, '%Y') = '2018'
 GROUP 
    BY  LEFT(JOIN_DT, 6) 
        , DATE_FORMAT(ORDER_DT, '%Y%m')
;
```

7. FS_SALE , FS_VVIP 테이블을 활용해서 VVIP회원의 2018년도 구매특성을 구매지표(객단가, 구매단가, 상품단가)를 통해 분석하고 싶습니다.  
* 조건(WHERE): 구매월 = 2018년도 
```sql
SELECT  SUM(SALES_AMT) AMT 
        , COUNT(DISTINCT ORDER_NO) ORD_CNT 
        , COUNT(DISTINCT A.MEM_NO) MEM_CNT 
        , SUM(SALES_QTY) QTY 
  FROM  FSN.FS_SALE A 
  JOIN  FSN.FS_VVIP B 
    ON  A.MEM_NO = B.MEM_NO 
 WHERE  1=1 
        AND YY = '2018' 
        AND CAN_YN = 'N' 
;
```

8. 2018년도 가입한 FS_MEMBER 채널별 회원들이 2018년도 '온라인'에서는 얼만큼 매출을 발생시켰는지 확인하고 싶습니다. 
```sql
SELECT  B.JOIN_CHANNEL 
        , SUM(SALES_AMT) AMT 
  FROM  FSN.FS_SALE A 
  LEFT 
  JOIN  FSN.FS_MEMBER B 
    ON  A.MEM_NO = B.MEM_NO 
 WHERE  OFF_YN = 'N' -- 온라인 
        AND CAN_YN = 'N' 
        AND YY = '2018' -- 매출 발생년도 
        AND LEFT(REPLACE(JOIN_DT,'-',''),4) = '2018'
 GROUP 
    BY  B.JOIN_CHANNEL
;
```

9. 오프라인 매장의 지역별 매출 비중을 알고 싶습니다. 
* FS_SALE, FS_STORE 테이블을 이용해서 2017년 1월, 2018년도 1월 지역(시도)별 매출을 비교해주세요. 
```sql 
SELECT  SIDO 
        , YM 
        , SUM(SALES_AMT) AMT 
  FROM  FSN.FS_SALE A 
  LEFT 
  JOIN  FSN.FS_STORE B 
    ON  A.STORE_ID = B.STORE_ID 
 WHERE  CAN_YN = 'N'
        AND YM IN ('201801','201701')
        AND OFF_YN = 'Y' 
 GROUP 
    BY  SIDO 
        , YM 
 ORDER 
    BY  2, 1 
; 
```

10. 이번 시즌 메인롤링 배너에 2534 회원들이 가장 많이 사는 TOP 5 상품을 광고하려고 합니다. 해당조건의 상품리스트를 구해주시고, 상품 특성 정보를 함께 보여주세요. 
```sql
SELECT  A.* 
        , B.*
  FROM  ( 
        SELECT  PROD_CD 
                , SUM(SALES_QTY) QTY 
                , ROW_NUMBER() OVER(ORDER BY SUM(SALES_QTY) DESC) RNK 
          FROM  FSN.FS_SALE A 
          JOIN  FSN.FS_MEMBER B 
            ON  A.MEM_NO = B.MEM_NO 
         WHERE  CAN_YN = 'N'
                AND (2023 - BIRTH_YY) BETWEEN 25 AND 34
         GROUP 
            BY  PROD_CD 
        ) A 
  LEFT 
  JOIN  FSN.FS_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 WHERE  RNK <= 5
; 
```

11. 2018년도 2월, 일별 UV, PV를 나타내주세요. 
* 2018년도 2월의 MAU랑 비교해주세요. 
```sql
SELECT  YMD 
        , COUNT(DISTINCT MEM_NO) UV 
        , SUM(1) PV
  FROM  FSN.FS_WLOG 
 WHERE  YM = '201802'
 GROUP
    BY  YMD

-- MAU 
SELECT  YM 
        , COUNT(DISTINCT MEM_NO) UV 
        , SUM(1) PV
  FROM  FSN.FS_WLOG 
 WHERE  YM = '201802'
 GROUP
    BY  YM
; 
```

12. 2018년도 2월 1일, 이탈이 가장 많은 페이지 번호(WPGE_ID)는 무엇인가요!? 

```sql
SELECT  WPGE_ID 
        , SUM(1)
  FROM  ( 
        SELECT  *
                , ROW_NUMBER() OVER(PARTITION BY SESSION_ID ORDER BY SRCH_DT DESC) RN
          FROM  FSN.FS_WLOG 
         WHERE  YMD = '20180201'
        ) A 
 WHERE  RN = 1 
 GROUP  
    BY  WPGE_ID
; 
```

13. 2017년도 vs 2018년도의 각 연도별 거래액, 구매자수, 주문건수, 구매상품 수를 나타내주세요. 

``` sql 
SELECT  LEFT(YM,4) YY   	
        , SUM(PAYMT_AMT) 거래액   
        , COUNT(DISTINCT MEM_NO) 구매자수   
        , COUNT(DISTINCT ORDER_NO) 주문건수 	
        , SUM(SALES_QTY) 판매상품수  	
  FROM  FS_SALE     	
 WHERE  1=1     	
        AND YY BETWEEN '2017' AND '2018'    	
        AND CAN_YN ='N' 	
 GROUP  	
    BY  LEFT(YM,4) 	
```	


14. 인천에 살고있거나 인천 매장에서 구매행동을 보였던 적이 있는 회원을 
타겟으로 추출하고 싶습니다. MEM_NO 만 구해주세요. 
* UNION ALL, JOIN 사용	
``` sql
SELECT  DISTINCT MEM_NO 
  FROM  (  
        SELECT  DISTINCT MEM_NO 
          FROM  FSN.FS_MEMBER
         WHERE  ADDR LIKE '%인천%'
         
        UNION ALL 
        
        SELECT  DISTINCT MEM_NO 
          FROM  FSN.FS_SALE A 
          LEFT 
          JOIN  FSN.FS_STORE B 
            ON  A.STORE_ID = B.STORE_ID   
         WHERE  SIDO = '인천'
        ) A    
``` 

15. 온라인에서 가장 많이 팔린 제품 10개와		
오프라인에서 가장 많이 팔린 제품 10개를 구해주세요. 		
* UNION ALL이 아닌, ROW_NUMBER 만 사용. 		
* HINT (PARTITION BY로 OFF_YN 구분)		
``` sql
SELECT  *		
  FROM  ( 		
        SELECT  PROD_CD 		
                , CASE WHEN OFF_YN = 'Y' THEN 'OFFLINE' ELSE 'ONLINE' END GUBUN 		
                , SUM(SALES_QTY) QTY 		
                , ROW_NUMBER() OVER(PARTITION BY OFF_YN ORDER BY SUM(SALES_QTY) DESC) RNK 		
          FROM  FSN.FS_SALE		
         WHERE  CAN_YN = 'N'		
         GROUP 		
            BY  PROD_CD 		
                , CASE WHEN OFF_YN = 'Y' THEN 'OFFLINE' ELSE 'ONLINE' END		
        ) A 		
 WHERE  RNK <= 10 	

-- UNION ALL 사용
SELECT  *       
  FROM  (       
        SELECT  'OFFLINE' GUBUN
                , PROD_CD         
                , SUM(SALES_QTY) QTY        
                , ROW_NUMBER() OVER(ORDER BY SUM(SALES_QTY) DESC) RNK       
          FROM  FSN.FS_SALE     
         WHERE  CAN_YN = 'N'        
                AND OFF_YN = 'Y'
         GROUP      
            BY  PROD_CD            
        ) A         
 WHERE  RNK <= 10   
 
UNION ALL 

SELECT  *       
  FROM  (       
        SELECT  'ONLINE' GUBUN
                , PROD_CD         
                , SUM(SALES_QTY) QTY        
                , ROW_NUMBER() OVER(ORDER BY SUM(SALES_QTY) DESC) RNK       
          FROM  FSN.FS_SALE     
         WHERE  CAN_YN = 'N'    
                AND OFF_YN = 'N'
         GROUP      
            BY  PROD_CD             
        ) A         
 WHERE  RNK <= 10   

``` 

16. VVIP 회원 각각의 2017년도 매출 내역을 확인하고자 합니다. 			
매출액(할인액 제외) 기준 TOP 100명의 리스트를 추출해주세요. 

``` sql		
SELECT  A.* 			
  FROM  ( 			
        SELECT  A.MEM_NO 			
                , SUM(PAYMT_AMT) PAYMT_AMT			
                , ROW_NUMBER() OVER(ORDER BY SUM(PAYMT_AMT) DESC) RNK		
          FROM  FS_SALE A 			
          JOIN  FS_VVIP B 			
            ON  A.MEM_NO = B.MEM_NO 			
         WHERE  YY = '2017'			
                AND CAN_YN = 'N' 			
         GROUP 			
            BY  A.MEM_NO			
        ) A 			
 WHERE  RNK <= 100			
``` 			

17. 2017년도 가입한 회원의 코호트 분석을 수행해주세요.
즉, 첫가입 후 리텐션을 구해주세요. 

``` sql			
SELECT  JOIN_YM                 
        , YM                    
        , COUNT(DISTINCT B.MEM_NO) CUST_CNT                 
  FROM  (                   
        SELECT  DISTINCT YM                  
                , MEM_NO                          
          FROM  FS_SALE                     
         WHERE  YM = '2017'    
                AND CAN_YN = 'N'
        ) A                                 
  JOIN  (                   
        SELECT  LEFT(REPLACE(JOIN_DT,'-',''),6) JOIN_YM
                , MEM_NO 
          FROM  FS_MEMBER B 
         WHERE  LEFT(REPLACE(JOIN_DT,'-',''),4) = '2017'
        ) B 
    ON  A.MEM_NO = B.MEM_NO                     
        AND YM >= JOIN_YM                       
 GROUP                  
    BY  JOIN_YM     
        , YM    		
```  		

18. 2017년도 RFM 모델에 따라서 
RECENCY : 최근성 
FREQUENCY : 빈도 
MONTARY : 금액(구매액)
을 구해주세요. 

``` sql
SELECT  MEM_NO 
        , DATEDIFF(SYSDATE(), MAX_DT) RECENCY
        , ORD_CNT 
        , AMT 
  FROM  (
        SELECT  MEM_NO 
                , MAX(ORDER_DT) MAX_DT 
                , COUNT(DISTINCT ORDER_NO) ORD_CNT 
                , SUM(SALES_AMT) AMT
          FROM  FS_SALE A 
         WHERE  1=1 
                AND YY = '2017'
                AND CAN_YN ='N' 
         GROUP
            BY  MEM_NO 
        ) A 
```

19. 18문제에서 구한 결과를 이용해서 각각 R, F, M 척도의 고객 순위를 나타내주세요. 

``` sql
WITH RFM AS ( 
SELECT  MEM_NO 
        , DATEDIFF(SYSDATE(), MAX_DT) RECENCY
        , ORD_CNT 
        , AMT 
  FROM  (
        SELECT  MEM_NO 
                , MAX(ORDER_DT) MAX_DT 
                , COUNT(DISTINCT ORDER_NO) ORD_CNT 
                , SUM(SALES_AMT) AMT
          FROM  FS_SALE A 
         WHERE  1=1 
                AND YY = '2017'
                AND CAN_YN ='N' 
         GROUP
            BY  MEM_NO 
        ) A 
)
SELECT  * 
        , ROW_NUMBER() OVER(ORDER BY RECENCY ASC) R 
        , ROW_NUMBER() OVER(ORDER BY ORD_CNT DESC) F
        , ROW_NUMBER() OVER(ORDER BY AMT DESC) M
  FROM  RFM 
``` 
