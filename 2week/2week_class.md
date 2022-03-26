# 2주차 

#### 1-1) 2016년도 서울 지역에서 일어난 거래액의 합은? 			
매출 테이블과 매장 정보 테이블 활용.

* LEFT JOIN 활용 			
``` sql			
SELECT  SUM(ORD_CNT)    			
        ,SUM(AMT)   			
  FROM  (   			
        SELECT  STORE_ID    			
                ,COUNT(DISTINCT ORDER_NO) ORD_CNT   			
                ,SUM(PAYMT_AMT) AMT 			
          FROM  FS_SALE 			
         WHERE  YM BETWEEN '201601' AND '201612'    			
                AND OFF_YN = 'Y'    			
                AND CAN_YN = 'N'    			
         GROUP  			
            BY  STORE_ID    			
        ) A     			
  LEFT  			
  JOIN  FS_STORE B  			
    ON  A.STORE_ID = B.STORE_ID     			
 WHERE  ADDR LIKE '%서울%' 			
```

* JOIN 활용			
``` sql			
SELECT  SUM(ORD_CNT)  			
        ,SUM(AMT)   			
  FROM  (   			
        SELECT  STORE_ID    			
                ,COUNT(DISTINCT ORDER_NO) ORD_CNT   			
                ,SUM(PAYMT_AMT) AMT 			
          FROM  FS_SALE 			
         WHERE  YM BETWEEN '201601' AND '201612'    			
                AND OFF_YN = 'Y'    			
                AND CAN_YN = 'N'    			
         GROUP  			
            BY  STORE_ID    			
        ) A     			
  JOIN  (   			
        SELECT  *			
          FROM  FS_STORE    			
         WHERE  ADDR LIKE '%서울%'            			
        ) B     			
    ON  A.STORE_ID = B.STORE_ID 			
```			
			
#### 1-2) 2016년도 오프라인 매장의 총결제 기준 지역별 거래액, 구매건 수는? 			

``` sql			
SELECT  SIDO    			
        ,SUM(ORD_CNT) ORD_CNT   			
        ,SUM(AMT) AMT   			
  FROM  (   			
        SELECT  STORE_ID    			
                ,COUNT(DISTINCT ORDER_NO) ORD_CNT   			
                ,SUM(PAYMT_AMT) AMT 			
          FROM  FS_SALE 			
         WHERE  YM BETWEEN '201601' AND '201612'    			
                AND OFF_YN = 'Y'    			
                AND CAN_YN = 'N'    			
         GROUP  			
            BY  STORE_ID    			
        ) A     			
  LEFT  			
  JOIN  FS_STORE B  			
    ON  A.STORE_ID = B.STORE_ID   			
 GROUP  			
    BY  SIDO    			
``` 

#### 1-3) 서울시 구별로 2016년도 판매 실적을 확인하고 싶습니다. 가장 매출이 높은 지역은 어디인가요? 			

``` sql			
SELECT  SIDO    			
        ,GUN    			
        ,SUM(ORD_CNT) ORD_CNT   			
        ,SUM(AMT) AMT   			
  FROM  (   			
        SELECT  STORE_ID    			
                ,COUNT(DISTINCT ORDER_NO) ORD_CNT   			
                ,SUM(PAYMT_AMT) AMT 			
          FROM  FS_SALE 			
         WHERE  YM BETWEEN '201601' AND '201612'    			
                AND OFF_YN = 'Y'    			
                AND CAN_YN = 'N'    			
         GROUP  			
            BY  STORE_ID    			
        ) A     			
  LEFT  			
  JOIN  FS_STORE B  			
    ON  A.STORE_ID = B.STORE_ID     			
 WHERE  ADDR LIKE '%서울%'    			
 GROUP  			
    BY  SIDO    			
        ,GUN    			
 ORDER  			
    BY  2 DESC  
    
    
-- 구매자 수를 COUNT 하는 경우에는 중복이 발생하는지, 기준 확인 후 추출 필요 

SELECT  SIDO               
        ,GUN                
        ,COUNT(DISTINCT ORDER_NO) ORD_CNT               
        ,SUM(SALES_AMT) AMT      
        ,COUNT(DISTINCT MEM_NO) 
  FROM  FS_SALE A 
  LEFT              
  JOIN  FS_STORE B              
    ON  A.STORE_ID = B.STORE_ID            
 WHERE  YM BETWEEN '201601' AND '201612'  
        AND ADDR LIKE '%서울%'     
        AND OFF_YN = 'Y'                
        AND CAN_YN = 'N'                
 GROUP              
    BY  SIDO               
        ,GUN      
	
``` 			
#### 2. 2017년도 가입하여 구매조차 하지 않고 탈퇴한 회원의 수를 확인해주세요. 			
``` sql			
-- 2017년도에 '가입'한 회원, 구매조차 하지 않고 탈퇴한 회원 리스트를 알고 싶습니다. 
SELECT  COUNT(DISTINCT A.MEM_NO) 
  FROM  ( 
        SELECT  * 
          FROM  FS_MEMBER 
         WHERE  LEFT(JOIN_DT, 4) = '2017'
        ) A 
  LEFT 
  JOIN  ( 
        SELECT  DISTINCT MEM_NO 
          FROM  FS_SALE 
         WHERE  YY >= '2017'
                AND CAN_YN = 'N' 
        ) B 
    ON  A.MEM_NO = B.MEM_NO 
 WHERE  WITHDR_GUBN = 'Y' -- 탈퇴
        AND B.MEM_NO IS NULL         			
``` 			

#### 3. 2017년 가입한 회원의 첫구매 데이터만 추출하고 싶습니다.			
ROW_NUMBER()			
``` sql			
SELECT  A.*                     
  FROM  (                   
        SELECT  A.*                 
                , ROW_NUMBER() OVER(PARTITION BY MEM_NO ORDER BY ORDER_DT) RNK      
          FROM  FS_SALE A                   
          JOIN  ( 
                SELECT  DISTINCT MEM_NO 
                  FROM  FS_MEMBER B 
                 WHERE  LEFT(JOIN_DT,4) = '2017'        
                ) B 
            ON  A.MEM_NO = B.MEM_NO
         WHERE  CAN_YN = 'N'         
                AND YY >= '2017'
        ) A 
 WHERE  RNK = 1     			
``` 	

#### 4. 회원들의 첫구매 이후 환불하는 경우가 너무 많다고 합니다. 순주문 (취반품 반영) 기준 첫구매 데이터를 추출해주세요. 			
-- 순주문 건만 추출하는 방법. 			
``` sql			
SELECT  ORDER_NO            
        , ORDER_DTL             
        , MEM_NO            
        , MIN(ORDER_DT) ORDER_DT            
        , SUM(CASE WHEN CAN_YN = 'N' THEN SALES_AMT         
                   WHEN CAN_YN = 'Y' THEN -1 * SALES_AMT END) SALE_AMT          
  FROM  FS_SALE A           
 GROUP          
    BY  ORDER_NO            
        , ORDER_DTL             
        , MEM_NO            
HAVING  SUM(CASE WHEN CAN_YN = 'N' THEN SALES_AMT           
                 WHEN CAN_YN = 'Y' THEN -1 * SALES_AMT END) > 0 	
``` 			
			
#### 5. JOIN절로 '순주문' 건만 추출하는 법. 				
``` sql			       
SELECT  A.*         
        , ROW_NUMBER() OVER(PARTITION BY MEM_NO ORDER BY ORDER_DT) RNK      
  FROM  FS_SALE A           
  JOIN  (           
        SELECT  ORDER_NO            
            , ORDER_DTL             
            , MEM_NO            
            , MIN(ORDER_DT) ORDER_DT            
            , SUM(CASE WHEN CAN_YN = 'N' THEN SALES_AMT         
                   WHEN CAN_YN = 'Y' THEN -1 * SALES_AMT END) SALE_AMT     
        FROM  FS_SALE A             
        GROUP           
            BY  ORDER_NO            
            , ORDER_DTL             
            , MEM_NO            
        HAVING  SUM(CASE WHEN CAN_YN = 'N' THEN SALES_AMT           
                 WHEN CAN_YN = 'Y' THEN -1 * SALES_AMT END) > 0      
        ) B         
    ON  A.ORDER_NO = B.ORDER_NO             
        AND A.ORDER_DTL = B.ORDER_DTL             		
``` 


6. 가입월에 따른 코호트 분석을 진행하고 싶습니다. 			
``` sql			
SELECT  JOIN_YM     			
        , YM        			
        , COUNT(DISTINCT B.MEM_NO) CUST_CNT     			
  FROM  (       			
        SELECT  YM      			
                , MEM_NO        			
                , SUM(SALES_AMT) NEW_SALES      			
          FROM  FS_SALE         			
         WHERE  YM BETWEEN '201701' AND '201712'        			
         GROUP      			
            BY  YM      			
                , MEM_NO        			
        ) A         			 			
  JOIN  (       			
        SELECT  MEM_NO      			
                , LEFT(REPLACE(JOIN_DT,'-',''), 6) JOIN_YM      			
          FROM  FS_MEMBER A         			
         WHERE  REPLACE(JOIN_DT, '-', '') BETWEEN '20170101' AND '20171231'     			
        ) B         			
    ON  A.MEM_NO = B.MEM_NO         			
        AND YM >= JOIN_YM   	    			
 GROUP      			
    BY  JOIN_YM     
	, YM 			
```  	

7. 구매전환률까지 구해주세요. 구매자/ 가입자			
``` sql			
SELECT  LEFT(JOIN_DT, 7) JOIN_YM			
        , COUNT(DISTINCT MEM_NO) JOIN_CUST			
  FROM  FS_MEMBER A 			
 WHERE  REPLACE(JOIN_DT, '-', '') BETWEEN '20170101' AND '20171231'  			
 GROUP 			
    BY  LEFT(JOIN_DT, 7) 			
``` 	

8. VVIP 회원 각각의 2017년도 매출 내역을 확인하고자 합니다. 			
TOP 100명의 리스트를 추출해주시고, 매출액(할인액 제외) 기준으로 순위를 매겨주세요.  	

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

9. 8번 문제에서 추출한 고객리스트에 고객 정보(FS_MEMBER)를 조합(JOIN)하여 			
연령대별 분포(매출액, 고객수, 주문건수) 확인 부탁드려요. 			

``` sql
SELECT  GD          
        , CASE WHEN (2021 - BIRTH_YY + 1) >= 60 THEN 'OVER60'           
               WHEN BIRTH_YY IS NULL THEN '알수없음'            
               ELSE CONCAT(FLOOR((2021 - BIRTH_YY + 1)/10)*10,FLOOR((2021 - BIRTH_YY + 1)/10)*10+10) END AGE            
        , COUNT(DISTINCT A.MEM_NO) N_MEM            
        , SUM(PAYMT_AMT) PAYMT_AMT
  FROM  (           
        SELECT  A.MEM_NO            
                , SUM(PAYMT_AMT) PAYMT_AMT     
                , COUNT(DISTINCT ORDER_NO) ORD_CNT
                , ROW_NUMBER() OVER(ORDER BY SUM(PAYMT_AMT) DESC) RNK           
          FROM  FS_SALE A           
          JOIN  FS_VVIP B           
            ON  A.MEM_NO = B.MEM_NO             
         WHERE  YY = '2017'         
                AND CAN_YN = 'N'            
         GROUP          
            BY  A.MEM_NO            
        ) A             
  LEFT          
  JOIN  FS_MEMBER B             
    ON  A.MEM_NO = B.MEM_NO         
 WHERE  RNK <= 100          
 GROUP          
    BY  GD          
        , CASE WHEN (2021 - BIRTH_YY + 1) >= 60 THEN 'OVER60'           
               WHEN BIRTH_YY IS NULL THEN '알수없음'            
               ELSE CONCAT(FLOOR((2021 - BIRTH_YY + 1)/10)*10,FLOOR((2021 - BIRTH_YY + 1)/10)*10+10) END        	
``` 

10. 인천에 살고있거나 인천 매장에서 구매행동을 보였던 적이 있는 회원을 
타겟으로 추출하고 싶습니다. 
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

11. 온라인에서 가장 많이 팔린 제품 10개와		
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
``` 
