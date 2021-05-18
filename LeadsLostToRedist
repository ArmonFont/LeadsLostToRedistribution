declare @StartDate date = cast(getdate()-60 as date)
declare @EndDate date = cast(getdate()-1 as date)
 
--Leads Taken Details
select a.UserID,  a.leadrefid Taken,  c.leadrefid Unassigned, c.CreateDtm NewOfferedDt, a.EventDate TakenDate, LeadAsgnToId NewAssigned, e.LeadActionId
into #leads
from MainPivotDatabase..AEActivityTable (nolock) a
left join LeadDatabase..LeadActivityTable (nolock) c on a.LeadRefID =  c.LeadRefId and c.LeadStsDtCd = 'NEW     ' and StsChgDtm >= EventDate
and StsChgById != #LeadManagerIDNumber#
and DATEPART(weekday,StsChgDtm) not in ('7','1') -- does not count any unassigned leads on weekends
and DATEDIFF(day, EventDate,StsChgDtm) < 3 -- does not count any unassigned 3 days after assigned
join MainPivotDatabase..AEInfoTable (nolock) d on a.UserID = d.UserId
left join LeadDatabase..LeadActionTable (nolock) e on c.LeadRefId = e.LeadNumber and LeadActionId in (#LeadActionIds#) and e.ActionById = a.UserID 
 where  a.eventdate >= @startdate
and a.eventdate < @enddate
and a.EventName = 'web lead'
and e.LeadActionId is null
group by a.UserID,  a.leadrefid,  c.leadrefid, c.CreateDtm, a.EventDate, e.LeadActionId,LeadAsgnToId
 
--dedup
;with cte as
(
select rn = row_number() over(partition by Taken order by NewOfferedDt desc)
,*
from #leads
)
delete cte where rn > 1
 
 
--credit pulls
select a.*
, case when  e.EventName = 'Credit Pull' then 1 else 0 end CreditPull
into #leads2
from #leads a
left join LeadDatabase..LeadInfoTable (nolock) b on a.Taken = b.LeadNumber
left join MainPivotDatabase..AEActivityTable(nolock) e on b.LOSLoanNumber = e.LoanNumber  and  e.EventName = 'Credit Pull' and a.UserID = e.UserID and a.Unassigned is null
 
 
 
--calls per lead RAW
select a.UserID,  a.leadrefid Taken,  c.leadrefid dialed, CreateDtm timedialed, a.EventDate LeadTakenDate
into #calls
from MainPivotDatabase..AEActivityTable (nolock) a
left join LeadDatabase..LeadActivityTable (nolock) c on a.LeadRefID =  c.LeadRefId and a.UserID= c.LeadAsgnToId 
and( LeadStsDtCd like '%ATMPT%' or LeadStsDtCd like '%CallCO%') and CreateDtm >= EventDate and c.ChgTypCd = 'set'
join MainPivotDatabase..AEInfoTable (nolock) d on a.UserID = d.UserId
where Department like '%TargetDepartment%'
and eventdate >= @startdate
and eventdate < @enddate
and EventName = 'web lead'
group by a.UserID,  a.leadrefid,  c.leadrefid, CreateDtm, EventDate, LeadAsgnToId
order by a.userid, c.LeadRefId, timedialed
 
 
--get timesdialed
select userid,taken
,sum(case when dialed is not null then 1 else 0 end) TimesDialed
into #calls1
from #calls
group by UserID,taken
 
select UserID, Taken
,case when TimesDialed = 0 then '0'
       when TimesDialed = 1 then '1'
       when timesdialed = 2 then '2'
       when TimesDialed  = 3 then '3'
       when TimesDialed = 4 then '4'
       when TimesDialed = 5 then '5'
       when TimesDialed = 5 then '6'
       else 'over 6' end TimesDialed
into #calls2
from #calls1
 
--agg
select c.FstNm + ' ' + c.LstNm AccountExec
,c.Department
, a.userid
,sum(case when a.Taken is not null then 1 else 0  end) Taken
,sum(CreditPull) Creditpull
,(1.00*sum(CreditPull)/(1.00*sum(case when a.Taken is not null then 1 else 0  end))) [CreditPull%]
,(sum(case when Unassigned is not null then 1 else 0 end)-sum(case when TimesDialed ='over 6' then 1 else 0 end)) Unassigned
,      ((1.00*sum(case when Unassigned is not null then 1 else 0 end)) -sum(case when TimesDialed ='over 6' then 1 else 0 end))
       /
       (1.00*sum(case when a.Taken is not null then 1 else 0  end)) [Unassign%]
,sum(case when TimesDialed ='0' then 1 else 0 end) 'No Dials'
,sum(case when TimesDialed ='1' then 1 else 0 end) '1 Dial'
,sum(case when TimesDialed ='2' then 1 else 0 end) '2 Dials'
,sum(case when TimesDialed ='3' then 1 else 0 end) '3 Dials'
,sum(case when TimesDialed ='4' then 1 else 0 end) '4 Dials'
,sum(case when TimesDialed ='5' then 1 else 0 end) '5 Dials'
,sum(case when TimesDialed ='6' then 1 else 0 end) '6 Dials'
,sum(case when TimesDialed ='over 6' then 1 else 0 end) 'over 6 Dials'
into #final
from #leads2 a
left join #calls2 b on a.Unassigned = b.Taken and a.UserID = b.UserID
join MainPivotDatabase..AEActivityTable (nolock) c on a.userid = c.UserId
group by a.UserID, c.FstNm + ' ' + c.LstNm , c.WaterFallDivision
having sum(case when a.Taken is not null then 1 else 0  end) >= @min --used as parameter in BI software
order by Taken desc
 
select *
from #final
where Department = @division --used as parameter in BI software
order by [Unassign%] asc
 
drop table #calls
drop table #calls1
drop table #calls2
drop table #leads
drop table #leads2
drop table #final
