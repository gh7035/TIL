``` sql
select
name,
price * stock_quantity as `가격`
from products;

select
concat(name,'-', email)
from customers; -- 표준
-- select concat(name,'-',NULL, email) from customers; -- n

select
concat_ws('--',name,email) 
from customers;

select 
length(email) 
from customers;

select 
name, 
length(name) 
from customers; -- 한국어 기준 각 3바이트 씩

select 
name, 
char_length(name) 
from customers; -- 글자수

-- NULL 비어있는 값 X, 양수 없는 - 연산 불가
select concat('이동욱', NULL);
-- select concat(name,'-',NULL, email) from customers; -> 값이 전부 null로 표시

-- 상품 설명이 null 인것을 '상품 설명 없음'으로 표시(실 데이터 변경 x)
select 
name, 
IFNULL(description, "상품 설명 없음") description 
from products;

-- where name != "" and name is not null;
-- ->
-- where IFNULL(name, '') != ''


-- 설명이 없으면 name으로, name 마저 없다면 '설명 없음' 으로
select 
name, 
coalesce(description, name, '설명 없음') 
from products;

select 
rtrim(name) 
from products;

select 
name,
price as `원래가격`, 
round(price * 0.85) as sale_price 
from products;

select 
customer_name, 
category, 
IFNULL(product_name, '분류없음') 
from order_stat;

SELECT 
    customer_name, 
    COALESCE(category, product_name, '분류없음') AS category, 
    product_name 
FROM order_stat;

SELECt 
substring_index(email, char_length(email) 
from customers;

```

``` sql
-- 갯수 세기
select count(*) from order_stat;



select count(category)
from order_stat; -- null은 count 에서 제외

select count(ifnull(category, ""))
from order_stat; -- null을 count에 포함, count 안에 함수 포함 가능

select
count(*) tot_count,
count(category) cate_cnt,
count(category)/count(*) rate
from order_stat; -- count 활용

select 
sum(quantity) as '총 판매수량',
AVG(quantity) as '평균 판매 수량'
from order_stat; -- sum(), avg() 활용

select 
sum(quantity * price) as '총 매출액',
AVG(quantity * price) as '평균 매출액'
from order_stat; -- sum(), avg() 활용

select 
max(price) as '가장 비싼 금액', 
min(price) as '가장 싼 금액' 
from order_stat; -- max() / min() 활용

select  
count(order_id) as `총 주문 건수`,
count(distinct customer_name) as `총 고객` 
from order_stat; -- distinct(중복 제거) 활용

select
customer_name,
count(*) order_cnt
from order_stat
group by customer_name;

select
category,
count(*) cnt
from order_stat
group by category;

select
customer_name,
category,
count(*) cnt
from order_stat
group by customer_name, category; -- select 의 갯수와, group by 의 갯수는 같아야 한다.

select
ifnull(category, '알수 없음') as `분류`,
SUM(price*quantity) total_price,
count(*) cnt
from order_stat
group by customer_name, category; -- sum() 수식 활용 + null도 group에 포함 된다.

select
ifnull(category, '알수 없음') as `분류`,
SUM(price*quantity) total_price,
count(*) cnt
from order_stat
-- where total_price >= 500000
group by `분류`
-- where은 기본적인 조건절로서 우선적으로 모든 필드를 조건에 둘 수 있다.
-- 하지만 having은 group by 된 이후 특정한 필드로 그룹화 되어진 새로운 테이블에 조건을 줄 수 있다.
having sum(price*quantity) > 500000
and sum(price*quantity) < 100000000;

select
customer_name,
count(*)
from order_stat
group by customer_name
having count(*) >= 3;


```