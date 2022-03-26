# 2주차 복습 

1) 2018년도 육아용품 매출 TOP 100 기록한 회원리스트를 회원정보와 함께 보여주세요. 
LEFT JOIN / ROW_NUMBER 활용 
``` sql 
SELECT  *
  FROM  ( 
        SELECT  MEM_NO 
                , COUNT(DISTINCT ORDER_NO) ORD_CNT
                , ROW_NUMBER() OVER(ORDER BY COUNT(DISTINCT ORDER_NO) DESC) RNK
          FROM  BABY_SALES A 
         WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
         GROUP 
            BY  MEM_NO 
        ) A             
  LEFT 
  JOIN  BABY_MEM B 
    ON  A.MEM_NO = B.MEM_NO 
 WHERE  RNK <= 100 
; 
``` 

2) 2019년 1월 가장 많이 팔린 제품은 무엇인가요!? 10개의 상품과, 그 제품의 특성을 알려주세요. (카테고리, 제품군)
* LEFT JOIN / ROW_NUMBER 활용 
``` sql 
SELECT  *
  FROM  ( 
        SELECT  PROD_CD 
                , COUNT(DISTINCT ORDER_NO) ORD_CNT
                , ROW_NUMBER() OVER(ORDER BY COUNT(DISTINCT ORDER_NO) DESC) RNK
          FROM  BABY_SALES A 
         WHERE  DATE_FORMAT(ORDER_DT, '%Y%m') = '201901'
         GROUP 
            BY  PROD_CD 
        ) A             
  LEFT 
  JOIN  BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD 
 WHERE  RNK <= 10
; 
``` 

3) 2018년도에 가장 많이 팔린 제품 3개를 1번이라도 구매한 회원들의 특성(성, 연령별) 을 확인해주세요. 
해당 회원특성이 저희의 주요 고객군이라고 여겨질 수 있습니다. 
- 연령 정보는 BABY_SALES의 AGE를 사용해주세요.
``` sql 
SELECT  GD
        , CASE WHEN AGE >= 60 THEN 'OVER60'
               WHEN AGE IS NULL THEN '알수없음'
               ELSE CONCAT(FLOOR(AGE/10)*10,FLOOR(AGE/10)*10+10) END AGE
        , COUNT(DISTINCT A.MEM_NO) N_MEM
  FROM  ( 
        SELECT  A.*
          FROM  BABY_SALES A 
          JOIN  ( 
                SELECT  PROD_CD 
                  FROM  ( 
                        SELECT  PROD_CD 
                                , COUNT(DISTINCT ORDER_NO) ORD_CNT
                                , ROW_NUMBER() OVER(ORDER BY COUNT(DISTINCT ORDER_NO) DESC) RNK
                          FROM  BABY_SALES A 
                         WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
                         GROUP 
                            BY  PROD_CD 
                        ) A 
                 WHERE  RNK <= 3 
                ) B      
            ON  A.PROD_CD = B.PROD_CD
         WHERE  DATE_FORMAT(ORDER_DT, '%Y') = '2018'
        ) A 
  JOIN  BABY_MEM B 
    ON  A.MEM_NO = B.MEM_NO
 GROUP 
    BY  GD
        , CASE WHEN AGE >= 60 THEN 'OVER60'
               WHEN AGE IS NULL THEN '알수없음'
               ELSE CONCAT(FLOOR(AGE/10)*10,FLOOR(AGE/10)*10+10) END 
``` 

4) 저희 브랜드의 단 1회 구매자의 구매 상품을 판매개수 순서대로 10개만 나타내주세요. 
- 어떤 카테고리의 상품일까요!? 
- BABY_SALES, BABY_PROD 테이블을 활용해서 구해주세요.  
- 해당 상품의 할인액 비율까지 구해주세요. 
``` sql
SELECT  A.* 
        , B.* 
  FROM  ( 
        SELECT  PROD_CD
                , SUM(QTY) QTY 
          FROM  ( 
                SELECT  MEM_NO 
                        , COUNT(DISTINCT ORDER_NO) ORD_CNT
                  FROM  BBY.BABY_SALES A 
                 GROUP 
                    BY  MEM_NO 
                HAVING  COUNT(DISTINCT ORDER_NO) = 1 
                ) A 
          JOIN  BBY.BABY_SALES B 
            ON  A.MEM_NO = B.MEM_NO 
         GROUP
            BY  PROD_CD 
        ) A 
  LEFT 
  JOIN  BBY.BABY_PROD B 
    ON  A.PROD_CD = B.PROD_CD
```     
