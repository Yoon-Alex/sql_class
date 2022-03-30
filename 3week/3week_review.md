# 3주차 복습 

1) 2018년도 가장 많이 팔린 제품 TOP 10 리스트를 보여주되, 카테고리와 용품 구분을 지어서 보여주세요. BBY_PROD 테이블과 JOIN  
``` sql 
SELECT  A.*
        , B.*
  FROM  ( 
        SELECT  PROD_CD 
                , SUM(QTY) QTY 
                , ROW_NUMBER() OVER(ORDER BY SUM(QTY) DESC) RNK 
          FROM  BABY_SALES A 
         WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
         GROUP 
            BY  PROD_CD 
        ) A 
  LEFT 
  JOIN  BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 WHERE  RNK <= 10 
``` 

2) 2018년도 BBY_SALES 데이터와 BBY_PROD 데이터를 활용하여 가장 많이 팔린 브랜드(BRAND1), 가장 많이 팔린 제품의 카테고리(CTGR,CTG_1 )를 조사해주세요.   
    - 브랜드 1등은?
``` sql     
SELECT  BRAND1
        , SUM(QTY) QTY 
  FROM  ( 
        SELECT  PROD_CD 
                , SUM(QTY) QTY 
                , ROW_NUMBER() OVER(ORDER BY SUM(QTY) DESC) RNK 
          FROM  BABY_SALES A 
         WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
         GROUP 
            BY  PROD_CD 
        ) A 
  LEFT 
  JOIN  BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 GROUP 
    BY  BRAND1
 ORDER 
    BY  2 DESC 
```     
    - 카테고리 1등은?
``` sql 
SELECT  CTGR
        , CTG_1 
        , SUM(QTY) QTY 
  FROM  ( 
        SELECT  PROD_CD 
                , SUM(QTY) QTY 
                , ROW_NUMBER() OVER(ORDER BY SUM(QTY) DESC) RNK 
          FROM  BABY_SALES A 
         WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
         GROUP 
            BY  PROD_CD 
        ) A 
  LEFT 
  JOIN  BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 GROUP 
    BY  CTGR
        , CTG_1 
 ORDER 
    BY  3 DESC 
```     
3) BBY 서비스는 아기용품(기저귀, 물티슈, 공갈젖꼭지 등)을 주로 취급하는 브랜드입니다.    
허나, 여성용품도 다른 구매처에 비해서는 싸게 구입할 수 있다는 장점이 있어 아기 용품 외에도 매출을 차지하는 부분입니다.    
* 1. 2018년도 여성용품의 매출액이 어느정도 전체 대비(%) 차지하는지 구해주세요.     
* 2. 또한 여성용품을 1개 이상이라도 구매한 회원의 성 연령별 특징을 구해주세요.   
    
1. 2018년도 여성용품의 매출액이 어느정도 전체 대비(%) 차지하는지 구해주세요. 
``` sql     
SELECT  CTG_1
        , SUM(SALES_AMT) AMT 
  FROM  BABY_SALES A 
  LEFT 
  JOIN  BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
 GROUP
    BY  CTG_1
```     

2. 또한 여성용품을 1개 이상이라도 구매한 회원의 성 연령별 특징을 구해주세요. 
``` sql 
SELECT  GD 
        , CASE WHEN AGE < 20 THEN '~20'
               WHEN AGE < 30 THEN '2029'
               WHEN AGE < 40 THEN '3039'
               WHEN AGE < 50 THEN '4049'
               WHEN AGE < 60 THEN '5059'
               WHEN AGE >= 60 THEN '60~'
               ELSE '알수없음' END AGEBAND 
        , COUNT(DISTINCT A.MEM_NO) MEM_NO 
  FROM  BABY_MEM A 
  JOIN  ( 
        SELECT  DISTINCT MEM_NO 
                , AGE
          FROM  BABY_SALES A 
          LEFT 
          JOIN  BABY_PROD B 
            ON  A.PROD_CD = B.PROD_CD 
         WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
                AND CTG_1 = '여성용품'
        ) B 
    ON  A.MEM_NO = B.MEM_NO 
 GROUP 
    BY  GD 
        , CASE WHEN AGE < 20 THEN '~20'
               WHEN AGE < 30 THEN '2029'
               WHEN AGE < 40 THEN '3039'
               WHEN AGE < 50 THEN '4049'
               WHEN AGE < 60 THEN '5059'
               WHEN AGE >= 60 THEN '60~'
               ELSE '알수없음' END 
``` 
