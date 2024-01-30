# DH-day8
--INNER JOIN 내부조인(동등조인)
SELECT *
FROM 학생;

SELECT *
FROM 수강내역;

SELECT 학생.이름 
    ,학생.평점  
    , 학생.학번
    ,수강내역.수강내역번호
    ,수강내역.과목번호
    ,과목.과목이름
FROM 학생,수강내역,과목
WHERE 학생.학번 =수강내역.학번
AND 수강내역.과목번호 = 과목.과목번호
AND 학생.이름 = '양지운';

--학생이 수강내역 건수를 조회하시오
SELECT 학생.이름
    ,ROUND(학생.평점) 평점
    ,학생.학번
    ,COUNT(수강내역.수강내역번호) 수강건수
FROM 학생,수강내역
WHERE 학생.학번 = 수강내역.학번
GROUP BY 학생.학번,학생.평점 , 학생.이름;
-- outer join외부조인 null값을 포함시키고 싶을때
SELECT 학생.이름
    ,ROUND(학생.평점)평점
    ,학생.학번
    ,COUNT(수강내역.수강내역번호)수강건수
FROM 학생,수강내역
WHERE 학생.학번 = 수강내역.학번(+) -- null값을 포함시킬 쪽에(+)
GROUP BY 학생.학번,학생.평점 , 학생.이름;

SELECT COUNT(*)
FROM 학생 , 수강내역; --cross join (주의해야함)
                    -- 9*17 =153
SELECT COUNT(*)
FROM 학생 , 수강내역
WHERE 학생.학번 = 수강내역.학번;
--학생의 수강내역수와 총 수강학점을 출력하시오
SELECT 학생.이름
    ,ROUND(학생.평점,2)평점
    ,학생.학번
    ,COUNT(수강내역.수강내역번호)수강건수
    ,SUM(NVL(과목.학점,0)) 총수강학점
FROM 학생,수강내역, 과목
WHERE 학생.학번 = 수강내역.학번(+) 
AND 수강내역.과목번호 = 과목.과목번호(+)
GROUP BY 학생.학번,ROUND(학생.평점,2) , 학생.이름
ORDER BY 1;

SELECT *
FROM member;

SELECT *
FROM cart;

SELECT *
FROM prod;
--카드사용횟수 품목수 구매상품수
SELECT a.mem_id
    , a.mem_name
    , count(DISTINCT(b.cart_no)) as  카드사용횟수
    ,count (DISTINCT(b.cart_prod)) as 품목수
    ,SUM (b.cart_qty) as  구매상품수
FROM member a
     , cart b
WHERE a.mem_id= b.cart_member
AND a.mem_name='김은대'
GROUP BY a.mem_id, a.mem_name ;
--아우터 조인
-- 카드사용횟수 품목수 구매상품수
--총구매금액도 출력
SELECT a.mem_id
    , a.mem_name
    , count(DISTINCT(b.cart_no)) as  카드사용횟수
    ,count (DISTINCT(b.cart_prod)) as 품목수
    ,SUM (NVL(b.cart_qty,0)) as  구매상품수
    ,SUM(prod_sale*NVL(b.cart_qty,0)) as 총가격
FROM member a
     , cart b
     , prod c
WHERE a.mem_id= b.cart_member
AND b.cart_prod = c.prod_id
AND a.mem_name='김은대'
GROUP BY a.mem_id, a.mem_name ;

--employees, jobs 테이블을 활용하여
--salary가 15000 이상인 직원의 사번, 이름, salary , 직업명을 출력하시오
SELECT *
FROM employees;

SELECT *
FROM jobs;

SELECT a.employee_id
     ,a.emp_name
     ,a.salary
     ,b.job_title
FROM employees a, jobs b
WHERE a.job_id= b.job_id
AND a.salary>=15000;

/*subquery (쿼리안에 쿼리)
    1.스칼리 서브쿼리(select절)
    2.인라인 뷰 (from절)
    3.중첩쿼리(where절)
    */
--스칼라 서브쿼리는 단일행 반환
--주의할 점은 메인 쿼리테이블의 행 건수 만큼 조회하기 떄문에(무거운 테이블을 사용하면 오래걸림)
--위와같은 상황에는 조인을 이용하는게 더 좋음
SELECT a.emp_name
    ,a.department_id --부서 아이디 대신 부서명이 필요할때
                    --부서 아이디는 부서테이블의 pk(유니크함 단일행 반환)
    ,(select department_name
    From departments
    WHERE department_id = a.department_id) as dep_nm 
    ,(select job_title --스칼라서브쿼리로 jobtitle을 출력하시오
    From jobs
    WHERE job_id = a.job_id) as jobnm
FROM employees a;


SELECT AVG(salary)
FROM employees;
--중첩서브쿼리(where절)
--직원중 salary 전체평균 보다 높은 직원을 출력하시오
SELECT emp_name,salary
FROM employees
WHERE salary >=(SELECT AVG(salary)
                FROM employees)
ORDER BY 2;

SELECT emp_name,salary
FROM employees
WHERE salary  >= 6461.232352352362
ORDER BY 2;

--학생중 '전체평균 평점' 이상인 학생정보만 출력하시오
SELECT 이름,학번,평점
FROM 학생
WHERE 평점 >=(SELECT AVG(평점)
                FROM 학생);
SELECT *
FROM 수강내역;

--평점이 가장 높은 학생의 정보를 출력
SELECT 이름,학번,평점
FROM 학생
WHERE 평점 = (SELECT max(평점)
                FROM 학생);
--수강 이력이 없는 학생의 이름을 출력하시오
SELECT 이름
FROM 학생 a
WHERE 학번 NOT IN(SELECT 학번 
                FROM 수강내역);

--동시에 2개이상의 컬럼값이 같은 껀 조회
SELECT employee_id
    , emp_name
    ,job_id
FROM employees
WHERE (employee_id, job_id) IN (SELECT employee_id, job_id
                                 FROM  job_history);
--지역과 각 년도별 대출총잔액을 구하는 쿼리를 작성해보자.(kor_ loan_status)
SELECT region
    ,(select loan_jan_amt --스칼라서브쿼리로 jobtitle을 출력하시오
    From kor_loan_status
    WHERE loan_jan_amt LIKE '2011%') as atm_2011
    ,(select loan_jan_amt 
    From kor_loan_status
    WHERE loan_jan_amt LIKE '2012%') as atm_2012
    ,(select loan_jan_amt 
    From kor_loan_status
    WHERE period LIKE '2013%') as atm_2013
FROM kor_loan_status
GROUP BY region;

--SELECT region
--      ,SUM(CASE WHEN period LIKE '2011%' THEN loan_jan_amt 
--          ELSE 0 
--       END) as amt_2011
--    ,SUM(CASE WHEN period LIKE '2012%' THEN loan_jan_amt 
--          ELSE 0 
--       END) as amt_2012
--    ,SUM(CASE WHEN period LIKE '2013%' THEN loan_jan_amt 
--          ELSE 0 
--       END) as amt_2013
--FROM kor_loan_status
--GROUP BY ROLLUP(region);



