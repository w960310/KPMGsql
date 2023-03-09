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
---------------------------
drop table if exists cc;
drop table if exists a1;
drop table if exists b1;

create temp table cc
(value_date char(10));

insert into cc
values('111/09/30');

select AED_KED.value_date, policy_no, po_issue_date, plan_abbr_code, 
'TWD' as currency, modx, paid_to_date, days, agp, upr, face_amt,unit_value
from AED_KED INNER JOIN cc on AED_KED.value_date = cc.value_date
where policy_no in ('174100139409','178300113187',
'143600202062','143800122234','177500206100')
into temp b1;


select AHAD.value_date, policy_no, co_issue_date, dur, plan_abbr_code, 
face_amt, 'TWD' as currency, modx, paid_to_date, unit_value, day1, day2, 
t0, t1, agp_p_modx, upr
from AHAD INNER JOIN cc on AHAD.value_date = cc.value_date
where policy_no in ('147700801530','172200968520',
'165100016760','142301073951','148000214656')
into temp a1;

insert into a1
select KAD.value_date, policy_no, co_issue_date, dur, plan_abbr_code, 
face_amt, 'TWD' as currency, modx, paid_to_date, unit_value, day1, day2, 
t0, t1, agp_p_modx, upr
from KAD INNER JOIN cc on KAD.value_date = cc.value_date
where policy_no in ('165700023270','175100478335',
'147700847954','147700846377','160600319847');

insert into a1
select JPA.value_date, policy_no, co_issue_date, dur, plan_abbr_code, 
face_amt, 'TWD' as currency, modx, paid_to_date, unit_value, day1, day2, 
t0, t1, agp_p_modx, upr
from JPA INNER JOIN cc on JPA.value_date = cc.value_date
where policy_no in ('146800320619','175300508676',
'175300509777','162800168946','179200260931');

insert into a1
select PWPR_v1.value_date, policy_no, co_issue_date, dur, plan_abbr_code, 
face_amt, 'TWD' as currency, modx, paid_to_date, unit_value, day1, day2, 
t0, t1, agp_p_modx, upr
from PWPR_v1 INNER JOIN cc on PWPR_v1.value_date = cc.value_date
where policy_no in ('173200247147','156900050205',
'177500328134','142300801419','176700355552');

unload to 'upr_aed_1110930.txt'
select a.*, b.po_sts_code from b1 a, polf b
where a.policy_no = b.policy_no;

unload to 'upr_1110930.txt'
select a.*, b.po_sts_code from a1 a, polf b
where a.policy_no = b.policy_no
order by plan_abbr_code


--

unload to JAD_policy_no.txt
select  plan_abbr_code, policy_no, upr, value_date
from KAD
where plan_abbr_code = 'JAD'
and  value_date = '111/12/31'

unload to JPA_policy_no.txt
select  plan_abbr_code, policy_no, upr, value_date
from JPA
where plan_abbr_code = 'JPA'
and  value_date = '111/12/31'

unload to JYL_policy_no.txt
select  plan_abbr_code, policy_no, upr, value_date
from JYL
where plan_abbr_code = 'JYL'
and  value_date = '111/12/31'

unload to KAD_policy_no.txt
select  plan_abbr_code, policy_no, upr, value_date
from KAD
where plan_abbr_code = 'KAD'
and  value_date = '111/12/31'

unload to SFR_policy_no.txt
select  plan_abbr_code, policy_no, upr, value_date
from SFR
where plan_abbr_code = 'SFR'
and  value_date = '111/12/31'

-語法錯誤

drop temp table s1

select  plan_abbr_code, policy_no, upr, value_date
from KAD
where plan_abbr_code = 'JAD'
and  value_date = '111/12/31'
into temp s1
;
insert into temp s1
select  plan_abbr_code, policy_no, upr, value_date
from JPA
where plan_abbr_code = 'JPA'
and  value_date = '111/12/31'
;
insert into temp s1
select  plan_abbr_code, policy_no, upr, value_date
from JYL
where plan_abbr_code = 'JYL'
and  value_date = '111/12/31'
;
insert into temp s1
select  plan_abbr_code, policy_no, upr, value_date
from KAD
where plan_abbr_code = 'KAD'
and  value_date = '111/12/31'
;
insert into temp s1
select  plan_abbr_code, policy_no, upr, value_date
from SFR
where plan_abbr_code = 'SFR'
and  value_date = '111/12/31'
;

unload to policy_no_all.txt
select * from s1


--

drop table if exists a1;
drop table if exists cc;

create temp table cc
(value_date char(10));

insert into cc
values('111/06/30');

select insurance_type, modx, sum(upr)
from upr43 inner join cc on upr43.value_date = cc.value_date
group by 1,2
order by 1,2;

select c.currency,a.insurance_type, c.modx, sum(upr)
from arco_a a, cc b, arkt c
where a.value_date = b.value_date
and a.policy_no = c.policy_no
and a.coverage_no = c.coverage_no
and a.policy_no||a.coverage_no not in (
select policy_no||coverage_no from upr43 a, cc b
where a.value_date = b.value_date)
and a.plan_abbr_code <> 'GPA'
group by 1,2,3
order by 1,2,3;

select 'P' insurance_type, modx, sum(upr)
from upr_com inner join cc on upr_com.value_date = cc.value_date
group by 1,2
order by 1,2;

select currency, insurance_type, '1' modx, sum(upr)
from vupr_all inner join cc on vupr_all.value_date = cc.value_date
group by 1,2,3
order by 1,2,3;

select c.insurance_type, a.modx, sum(upr)
from gico_upr a, cc b, gipf c
where a.value_date = b.value_date
and a.plan_code = c.plan_code
and a.rate_scale = c.rate_scale
group by 1,2
order by 1,2;

select insurance_type, modx, sum(upr)
from ginw_upr_v1 inner join cc on ginw_upr_v1.value_date = cc.value_date
group by 1,2
order by 1,2;
--
create temp table cc
(value_date char(10));

insert into cc
values('111/05/31');

drop table if exists a1;

select plan_abbr_code, policy_no, upr 
from KAD INNER JOIN cc ON KAD.value_date = cc.value_date
where plan_abbr_code = 'KAD'
into temp a1;

insert into a1
select plan_abbr_code, policy_no, upr 
from JPA INNER JOIN cc ON JPA.value_date = cc.value_date
where plan_abbr_code = 'JPA';

insert into a1
select plan_abbr_code, policy_no, upr 
from JYL INNER JOIN cc ON JYL.value_date = cc.value_date
where plan_abbr_code = 'PWL';

insert into a1
select plan_abbr_code, policy_no, upr 
from SFDR INNER JOIN cc ON SFDR.value_date = cc.value_date
where plan_abbr_code = 'SFDR';

insert into a1
select plan_abbr_code, policy_no, upr 
from SPA INNER JOIN cc ON SPA.value_date = cc.value_date
where plan_abbr_code = 'SPA';

unload to 'upr_1110531.txt'
select * from a1













