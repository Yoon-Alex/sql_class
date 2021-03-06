1. 매출(BABY_SALES) 테이블에서 등급(GRADE) 별 구매자 수, 거래액, 실거래액(PAYMT_AMT)을 		
구하여 구매지표를 구해주세요. 		
구매지표 : 객단가, 주문단가, 주문건당 상품 수, 상품 평균단가	

``` sql 
SELECT  GRADE   
        , COUNT(DISTINCT MEM_NO) CUST   
        , SUM(SALES_AMT) SALES_AMT  
        , SUM(PAYMT_AMT) PAYMT_AMT  
        , SUM(QTY) QTY  
  FROM  BBY.BABY_SALES  
 WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'    
 GROUP  
    BY  GRADE   
``` 	

2. 매출(BABY_SALES) 테이블에서 채널(CHNL)별 매출, 구매자수, 주문건수, 상품수를 확인해주세요. 		
어떤 경로에서 매출이 가장 많이 일어나나요?  		
해당 채널에서 발생하는 구매자의 특징은 무엇인가요? 		
구매지표를 활용해서 분석해주세요. 		

``` sql 
SELECT  CHNL    
        , COUNT(DISTINCT MEM_NO) CUST   
        , COUNT(DISTINCT ORDER_NO) ORD_CNT  
        , SUM(SALES_AMT) SALES_AMT  
        , SUM(PAYMT_AMT) PAYMT_AMT  
  FROM  BBY.BABY_SALES  
 WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'    
 GROUP  
    BY  CHNL          
```		

		
3. 육아용품의 경우 주말 동안 다쓴 물티슈, 기저귀, 분유를 월요일에 주문하는 경향이 높다고 알려져 있습니다. https://ppss.kr/archives/185928		
 기저귀의 경우 부피가 크고 자주 사용되기 때문입니다. 이 주장이 사실인지 주장하는 분석자료 보고서를 제출해주세요. 		
즉, 앱 매출을 살펴봤을때 구매빈도가 '월'요일에 높은지를 확인하고 싶습니다. 
``` sql  
SELECT  DAYS    
       , AVG(ORD_CNT) AVG_CNT   
  FROM  (   
        SELECT  DATE_FORMAT(ORDER_DT, '%W') DAYS    
                , DATE_FORMAT(ORDER_DT, '%Y%m%d') YMD   
                , COUNT(DISTINCT ORDER_NO) ORD_CNT      
          FROM  BBY.BABY_SALES A    
         GROUP  
            BY  DATE_FORMAT(ORDER_DT, '%W')     
               ,DATE_FORMAT(ORDER_DT, '%Y%m%d') 
        ) A     
 GROUP  
    BY  DAYS
``` 

4. 온라인 주문의 경우 시간대에 대한 특징이 발생하나요? 어떤 시간에 광고를 해야 더욱 효과적인 결과를 이끌어낼지 분석 요청드려요.
``` sql  		
SELECT  HOUR        
        , AVG(ORD_CNT) AVG_CNT      
  FROM  (       
        SELECT  DATE_FORMAT(ORDER_DT, '%H') HOUR        
                ,DATE_FORMAT(ORDER_DT, '%Y%m%d') YMD        
                ,COUNT(DISTINCT ORDER_NO) ORD_CNT   
          FROM  BBY.BABY_SALES A        
         WHERE  A.CHNL IN ('App','Web')
         GROUP      
            BY  DATE_FORMAT(ORDER_DT, '%H')         
                ,DATE_FORMAT(ORDER_DT, '%Y%m%d')        
        ) A         
 GROUP      
    BY  HOUR
``` 
