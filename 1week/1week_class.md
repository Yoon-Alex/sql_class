# 1주차 쿼리

1. 서울시에 위치한 매장리스트를 받고싶어요. 	
``` sql 
SELECT  *	
  FROM  FS_STORE 	
 WHERE  ADDR LIKE '%서울%'	
```	
2. 온라인 구매내역 (테이블) 추출하고 싶어요. 	
``` sql 	
SELECT  *	
  FROM  FS_SALE	
 WHERE  OFF_YN = 'N'	
```	
	
3. ORDER_DT 주문일 기준으로 2016년 1월 28일의 구매 데이터만 추출하고 싶어요. 	
``` sql 	
SELECT  * 	
  FROM  FS_SALES	
 WHERE  DATE_FORMAT(ORDER_DT, '%Y%m%d' ) = '20160128'	
```	
4. ORDER_DT 주문일 기준으로 2016년 1월 28일 ~ 2016년 1월 30일까지 데이터를 보고 싶어요.	
``` sql 
SELECT  * 	
  FROM  FS_SALES	
 WHERE  DATE_FORMAT(ORDER_DT, '%Y%m%d' ) BETWEEN '20160128' AND '20160130'	
```	
5. ORDER_DT 주문일 기준으로 2016년 1월 28일의 구매 지표를 확인하고 싶어요.  	
구매자 수, 매출액, 주문건수. 	
``` sql 
SELECT  COUNT(DISTINCT MEM_NO) 구매자수	
        , SUM(PAYMT_AMT) 매출액	
        , COUNT(DISTINCT ORDER_NO) 주문건수	
  FROM  FS_SALES	
 WHERE  DATE_FORMAT(ORDER_DT, '%Y%m%d' ) = '20160128'	
	 AND CAN_YN = 'N' -- 환불건 제외 
```	
6. ORDER_DT 주문일 기준으로 2016년 1월 28일 ~ 2016년 1월 30일까지 구매지표를 확인하고 싶어요. 	
``` sql 
SELECT  COUNT(DISTINCT MEM_NO) 구매자수	
        , SUM(PAYMT_AMT) 매출액	
        , COUNT(DISTINCT ORDER_NO) 주문건수	
  FROM  FS_SALES	
 WHERE  DATE_FORMAT(ORDER_DT, '%Y%m%d' ) BETWEEN '20160128' AND '20160130'	
	 AND CAN_YN = 'N' 
```	

* 6-1. 각 일자마다의 구매지표를 확인하고 싶다면? 	
``` sql 
일자별.	
SELECT  DATE_FORMAT(ORDER_DT, '%Y%m%d') YMD  	
        , COUNT(DISTINCT MEM_NO) 구매자수 	
        , SUM(PAYMT_AMT) 매출액   	
        , COUNT(DISTINCT ORDER_NO) 주문건수    	
  FROM  FS_SALES    	
 WHERE  DATE_FORMAT(ORDER_DT, '%Y%m%d' ) BETWEEN '20160128' AND '20160130'  	
        AND CAN_YN = 'N' 	
 GROUP 	
    BY  DATE_FORMAT(ORDER_DT, '%Y%m%d') 	
```	

  
7. 2017년도 vs 2018년도의 각 연도별 거래액, 구매자수, 주문건수, 구매상품 수를 나타내주세요. 	
``` sql 
SELECT  LEFT(YM,4) YY   	
        , COUNT(DISTINCT MEM_NO) 구매자수   	
        , SUM(PAYMT_AMT) 거래액    	
        , COUNT(DISTINCT ORDER_NO) 주문건수 	
        , SUM(SALES_QTY) 판매상품수  	
  FROM  FS_SALE     	
 WHERE  1=1     	
        AND YM BETWEEN '201601' AND '201712'    	
        AND CAN_YN ='N' 	
 GROUP  	
    BY  LEFT(YM,4) 	
```	
8. 객단가, 주문단가, 주문건당 상품 수, 상품 평균단가 등을 구해주세요.  	
    * 엑셀 활용. 


9. 2018년 1월의 환불 고객 수? 	
``` sql 
SELECT  COUNT(DISTINCT MEM_NO)	
  FROM  FS_SALES	
 WHERE  LEFT(SALES_DY, 6) = '201801'	
 	    AND CAN_YN = 'Y'
```	
10. 2016년도 VS 2017년도 월별 환불고객 수 비교. 	
추이가 발생하는가? 	
``` sql 
SELECT  YM 	
	    , COUNT(DISTINCT MEM_NO) 환불건수
  FROM  FS_SALE	
 WHERE  1=1	
        AND YM BETWEEN '201601' AND '201712' 	
        AND CAN_YN = 'Y'	
 GROUP 	
    BY  YM 	
```	
11. 환불건 외 각 월의 반품율을 구하고 싶다면. 	
``` sql 
CASE WHEN 사용. 	
SELECT  YM 	
        , COUNT(DISTINCT CASE WHEN CAN_YN='Y' THEN ORDER_NO END) 환불건수
        , COUNT(DISTINCT CASE WHEN CAN_YN='N' THEN ORDER_NO END) 구매건수
  FROM  FS_SALE	
 WHERE  1=1	
 	AND YM BETWEEN '201601' AND '201712'
 GROUP 	
    BY  YM 	
```	
12. 유효회원 중 성, 연령별 인구통계학적 특성을 확인하고 싶습니다. 	
CASE WHEN 사용. 	
``` sql 	
SELECT  GD 
        , CASE WHEN (2021 - BIRTH_YY + 1) >= 60 THEN 'OVER60'   
               WHEN (2021 - BIRTH_YY + 1) >= 55 THEN '5559'
               WHEN (2021 - BIRTH_YY + 1) >= 50 THEN '5054'
               WHEN (2021 - BIRTH_YY + 1) >= 45 THEN '4549'
               WHEN (2021 - BIRTH_YY + 1) >= 40 THEN '4044'
               WHEN (2021 - BIRTH_YY + 1) >= 35 THEN '3539'
               WHEN (2021 - BIRTH_YY + 1) >= 30 THEN '3034'
               WHEN (2021 - BIRTH_YY + 1) >= 25 THEN '2529'
               WHEN (2021 - BIRTH_YY + 1) >= 20 THEN '2024'
               ELSE (2021 - BIRTH_YY + 1) < 20 THEN '~20'
               ELSE '알수없음' END AGEBAND
        , COUNT(DISTINCT MEM_NO) N
  FROM  FS_MEMBER   
 WHERE  WITHDR_GUBN = 'N'   
        AND REST_YN = 'N'   
        AND GD IS NOT NULL  
 GROUP 
    BY  GD 
        , CASE WHEN (2021 - BIRTH_YY + 1) >= 60 THEN 'OVER60'   
               WHEN (2021 - BIRTH_YY + 1) >= 55 THEN '5559'
               WHEN (2021 - BIRTH_YY + 1) >= 50 THEN '5054'
               WHEN (2021 - BIRTH_YY + 1) >= 45 THEN '4549'
               WHEN (2021 - BIRTH_YY + 1) >= 40 THEN '4044'
               WHEN (2021 - BIRTH_YY + 1) >= 35 THEN '3539'
               WHEN (2021 - BIRTH_YY + 1) >= 30 THEN '3034'
               WHEN (2021 - BIRTH_YY + 1) >= 25 THEN '2529'
               WHEN (2021 - BIRTH_YY + 1) >= 20 THEN '2024'
               ELSE (2021 - BIRTH_YY + 1) < 20 THEN '~20'
               ELSE '알수없음' END 

-- 더 간단하게. FLOOR 활용
SELECT  GD 	
        , CASE WHEN (2021 - BIRTH_YY + 1) >= 60 THEN 'OVER60'	
               ELSE CONCAT(FLOOR((2021 - BIRTH_YY + 1)/5)*5,FLOOR((2021 - BIRTH_YY + 1)/5)*5+5) END AGE	
        , COUNT(DISTINCT MEM_NO) N 	
  FROM  FS_MEMBER 	
 WHERE  WITHDR_GUBN = 'N' 	
        AND REST_YN = 'N' 	
        AND GD IS NOT NULL 	
 GROUP 	
    BY  GD 	
        , CASE WHEN (2021 - BIRTH_YY + 1) >= 60 THEN 'OVER60'	
               ELSE CONCAT(FLOOR((2021 - BIRTH_YY + 1)/5)*5,FLOOR((2021 - BIRTH_YY + 1)/5)*5+5) END 	
```	
13. 2017년도 매출 데이터, 고객데이터를 활용하여 ‘유효회원‘ 중	
각 회원의 구매건수, 나이 정보를 조합하여 아래 고객 군으로 나누어 고객 수를 세어주세요. 	

2534 연령대의 회원을 ‘MZ’ 	
35세 이상 회원을 구매력이 높은 회원이라는 ‘LOYAL’ 	
그외 회원 중 구매횟수가 1회 이하인 회원을 ‘FUTURE’라는 	
이름의 고객 군으로 나누어주세요.  	

``` sql 	
SELECT  CASE WHEN AGE BETWEEN 25 AND 34 THEN 'MZ'	
               WHEN AGE >= 34 THEN 'LOYAL'	
               WHEN IFNULL(ORD_CNT,0) <= 1 THEN 'FUTURE' END GUBUN 	
        , COUNT(DISTINCT A.MEM_NO)	
  FROM  ( 	
        SELECT  MEM_NO 	
          FROM  FS_MEMBER 	
         WHERE  1=1 	
                AND WITHDR_GUBN = 'N' 	
                AND REST_YN = 'N' 	
        ) A 	
  LEFT 	
  JOIN  ( 	
        SELECT  MEM_NO 	
                , MAX(AGE) AGE 	
                , COUNT(DISTINCT ORDER_NO) ORD_CNT	
          FROM  FS_SALE 	
         WHERE  YY = '2017'	
                AND CAN_YN = 'N' 	
         GROUP 	
            BY  MEM_NO 	
        ) B 	
    ON  A.MEM_NO = B.MEM_NO     	
 GROUP 	
    BY  CASE WHEN AGE BETWEEN 25 AND 34 THEN 'MZ'	
               WHEN AGE >= 34 THEN 'LOYAL'	
               WHEN IFNULL(ORD_CNT,0) <= 1 THEN 'FUTURE' END	
```