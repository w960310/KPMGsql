# KPMGsql

1.COI

unload to 'acti.txt'
select value_date, currency, sum(amt_3)
from acti
where acct_no in ('45060208','45060263')
and insurance_type in ('V','U')
and value_date >= '109/12/31'
group by 1,2
order by 1,2;

unload to 'upr_all.txt'
select value_date, currency, sum(upr)
from upr_all
where value_date >= '109/12/31'
and plan_abbr_code in (select distinct plan_abbr_code from pldf
where insurance_type in ('V','U'))
and dur <> '1'
group by 1,2
order by 1,2;

unload to 'vexrt.txt'
select left(exrt_date,6), currency, noon_selling_rate
from vexrt
where right(exrt_date,2) = '00'
and exrt_source_ind ='0'
and left(exrt_date,3) in ('109','110','111')

2.KAD,JAD,ADDR

select sum(cnt), sum(upr) from upr_all
where plan_abbr_code = 'GOFCR'
and dur = '1'
and value_date = '109/12/31'

select sum(cnt), sum(upr) from upr_all
where plan_abbr_code = 'GOFCR'
and dur <> '1'
and value_date = '109/12/31'

select sum(cnt), sum(upr) from upr_all
where plan_abbr_code = 'GOFCR'
and dur = '1'
and value_date = '110/12/31'

select sum(cnt), sum(upr) from upr_all
where plan_abbr_code = 'GOFCR'
and dur <> '1'
and value_date = '110/12/31'

select sum(cnt), sum(upr) from upr_all
where plan_abbr_code = 'GOFCR'
and dur = '1'
and value_date = '111/12/31'

select sum(cnt), sum(upr) from upr_all
where plan_abbr_code = 'GOFCR'
and dur <> '1'
and value_date = '111/12/31'


--
select sum(cnt), sum(agp_upr), sum(upr)
from upr_all
where plan_abbr_code = 'ADDR'
and dur = '1'
and value_date = '109/12/31'

select sum(cnt), sum(agp_upr), sum(upr)
from upr_all
where plan_abbr_code = 'ADDR'
and dur <> '1'
and value_date = '109/12/31'

保費從保費收入(亭毓)檔案看(檢核項要以此檔為主)
agp、upr可從UPR檔案看，豁免單記得加上upr43('DB')
有效件數該用sum(cnt)

drop table if exists a1;
drop table if exists a2;

select value_date, plan_abbr_code,
case when dur = '1'
     then 'FY'
     else 'RY' end as FY_RY,
sum(cnt) as cnt,
sum(agp_upr) as agp,
sum(upr) as upr
from upr_all
where left(value_date,3) in ('109','110','111')
and right(value_date,5) in ('12/31')
and plan_abbr_code in ('BHSR','ZHSR')
group by 1,2,3
order by 1,2,3
into temp a1;

select a.*, 
case when FY_RY = 'FY'
     then sum(amt_1+amt_2)
     else sum(amt_3) end as prem
from a1 a, acti b
where left(a.value_date,3) = left(b.value_date,3)
and a.plan_abbr_code = b.plan_abbr_code
group by 1,2,3,4,5,6
order by 1,2,3,4,5,6
into temp a2;

unload to 'BHSR_ZHSR.txt'
select * from a2

3.美元健康險

drop table if exists a1;
select value_date, plan_abbr_code, 
case when dur = '1'
     then 'FY'
     else 'RY' end as FY_RY,
sum(upr) as upr
from upr_all
where left(value_date,3) in ('109','110','111')
and plan_abbr_code in ('MJWPR','MSWPR')
group by 1,2,3
order by 1,2,3
into temp a1;

unload to '美元健康險.txt'
select * from a1


