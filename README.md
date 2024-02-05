# IPL-Data-Analysis
-- 1. We will create database schema to load the tables

CREATE DATABASE ipl;

-- 2. Create table to store data

CREATE TABLE ipl.matches( id integer,
					  season integer,
                      city varchar(100),
                      date_ date ,
                      team1 varchar(100),
                      team2 varchar(100),
                      toss_winner varchar(100),
                      toss_decision varchar(100), 
                      result varchar(100),
                      dl_applied integer,
                      winner varchar(100),
                      win_by_runs integer,
                      win_by_wickets integer, 
                      player_of_the_match varchar(100),
                      venue varchar(100),
                      umpire1 varchar(100), 
                      umpire2 varchar(100), 
                      umpire3 varchar(100));
                      
                      
CREATE TABLE ipl.deliveries (matchid integer,
                             inning integer, 
                             batting_team varchar(100),
							 bowling_team varchar(100), 
                             over_ integer, 
                             ball integer,
                             batsman varchar(100),
                             non_striker varchar(100), 
                             bowler varchar(100), 
                             is_super_over integer,
                             wide_runs integer,
                             bye_runs integer,
                             legbye_runs integer, 
                             noball_runs integer, 
                             penalty_runs integer, 
                             batsman_runs integer, 
                             extra_runs integer, 
                             total_runs integer, 
						     player_dismissed varchar(100),
                             dismissal_kind varchar(100), 
                             fielder varchar(100));
                           
                             
-- 3. Check the tables structure

-- 4. Now, Load the data in tables

LOAD DATA INFILE 'IPL MATCHES.csv' 
INTO TABLE matches
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES;


LOAD DATA INFILE 'IPL DELIVERIES.csv' 
INTO TABLE deliveries
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES;


SHOW GLOBAL VARIABLES LIKE 'local_infile';
# -- SET GLOBAL local_infile = 'ON';
SET GLOBAL local_infile=1;




# Now, Begin with the queries:
-----------------------------------------------------------------------------------------------------------------------------------

# Shape of Data
-----------------
-- Shapes are combination of rows and columns
-- To know about your table structure (I mean rows and columns) use below query.

select count(*)
as No_of_row 
from ipl.matches;

select count(*) as No_of_columns
from information_schema.columns
where TABLE_NAME='matches';


# Viewing data
----------------
select * 
from ipl.matches;

Select * 
from ipl.deliveries;
 
 
# View selected columns
------------------------
select m.season,m.city,m.date_,m.team1,m.team2,m.winner,m.win_by_runs
from ipl.matches m
where m.season='2017'
limit 5;


# Distinct values
-------------------
select distinct year(m.date_) as 'Year of Match'
from ipl.matches m
order by 1;

select count(distinct player_of_the_match) as 'No of Matches'
from ipl.matches m
order by 1;


# Find season winner for each season (season winner is the winner of the last match of each season)
--------------------------------------
SELECT season, winner
FROM (
    SELECT season, winner, ROW_NUMBER() OVER (PARTITION BY season ORDER BY date_ DESC) as row_num
    FROM ipl.matches
) AS subquery
WHERE row_num = 1
ORDER BY season DESC;

 
 
# Find venue of 10 most recently played matches
------------------------------------------------
select DISTINCT m.venue,m.date_ 
from ipl.matches m
order by m.date_ desc
limit 10;


# Case (4,6, single,0)
-------------------------
select distinct batsman,bowler,ball,
CASE
when total_runs=1 then 'Single'
when total_runs=4 then 'Boundry'
when total_runs=6 then 'Six'
 else 'Duck'
end as 'Run in words' from ipl.deliveries ;


# Data Aggregation
--------------------
select winner,max(win_by_wickets),max(win_by_runs)
from ipl.matches
group by winner
order by 3 desc;


# How many extra runs have been conceded in ipl
-------------------------------------------------
select distinct bowler,sum(extra_runs)
from ipl.deliveries
group by bowler
having sum(extra_runs)>0;
 
 
# On an average, teams won by how many runs in ipl
---------------------------------------------------
select winner,avg(win_by_runs) 
from ipl.matches
group by winner
having avg(win_by_runs)>0
order by 2 desc;


# How many extra runs were conceded in ipl by SK Warne
-------------------------------------------------------
select bowler,sum(extra_runs)
from ipl.deliveries
where bowler='SK Warne'
group by bowler;


# How many boundaries (4s or 6s) have been hit in ipl
-------------------------------------------------------
select m.winner, d.total_runs,count(d.total_runs) AS TOTAL_RUNS 
from ipl.deliveries d
inner join ipl.matches m on m.id=d.matchid
where d.total_runs in (4,6)
group by m.winner, d.total_runs;


/*Cricket trading has become increasingly popular in recent years.
With the evolving cricket landscape, there are new opportunities and 
it can be a great way to connect with other cricket enthusiasts and potentially make some profit.
Here you have to buy or sell shares of a team and the result will be decided by the winning team.
There are different markets too where you can Invest on YES or NO for a particular over's runs or 
team runs or a specific batsman's runs. There are plenty of markets to explore.*/

/*This project can help us predicting the outcome of a event by analyzing the past data. For a particular over
we can get the past record of the bowler and batsmans who will be playing in that over. Its not like every time
our prediction from past data will come true. But we dont need to 100% accuracy rate to be in profit. Even if we
are getting results with 51% accuracy rate, it will do the job.*/

# Batsman's stats agains a Bowler
------------------------------------------
select *
from ipl.deliveries
where batsman='Yuvraj Singh' and bowler='Harbhajan Singh';

# Batsman's stats agains a team
------------------------------------------
select *
from ipl.deliveries
where batsman='Yuvraj Singh' and bowling_team='Mumbai Indians';

# Bowler's stats agains a Team
------------------------------------------
select *
from ipl.deliveries
where bowler='Harbhajan Singh' and batting_team='Delhi Daredevils';

# Bowler's stats agains Batsmans
------------------------------------------
select *
from ipl.deliveries
where bowler='Harbhajan Singh' and batsman IN ('Yuvraj Singh','S Dhawan');


# How many balls did SK Warne bowl to batsman SR Tendulkar
-----------------------------------------------------------
select batsman,bowler, count(ball) 
from ipl.deliveries
#where bowler='BCJ Cutting' and batsman='DR Smith'
group by batsman,bowler;



# How many matches were played in the month of April
------------------------------------------------------
select count(*) 
from ipl.matches
where month(date_)='4';

select count(*) from ipl.matches
where extract(month from date_)=4;


# How many matches were played in the March and June
------------------------------------------------------
select count(*) 
from ipl.matches
where month(date_) in ('3','6');


# Total number of wickets taken in ipl (count not null values)
---------------------------------------------------------------
select count(player_dismissed) as 'Wicket' 
from ipl.deliveries
where player_dismissed <>"";

select dismissal_kind,player_dismissed, count(*) 
from ipl.deliveries
where player_dismissed <>""
group by dismissal_kind,player_dismissed
order by 3 desc;


# Pattern Match ( Like operators % _ )
----------------------------------------
select Distinct player_of_the_match
from ipl.matches 
where player_of_the_match Like '%M%';

select Distinct player_of_the_match 
from ipl.matches 
where player_of_the_match 
like 'JJ %';

select distinct player_of_the_match 
from ipl.matches 
where player_of_the_match 
like 'K_ P%';


# How many teams have word royal in it (could be anywhere in the team name, any case)
--------------------------------------------------------------------------------------
SELECT DISTINCT team1
FROM ipl.matches
WHERE team1 LIKE '%Royal%';


# Group by - Maximum runs by which any team won a match per season
-------------------------------------------------------------------
select season,max(win_by_runs) 
from ipl.matches
group by season
order by 1;


# Create score card for each match Id
-----------------------------------------
select batting_team,batsman,sum(batsman_runs) 
from ipl.deliveries
group by batting_team,batsman
order by 3 desc;


# Top 10 players with max boundaries (4 or 6)
-----------------------------------------------
select DISTINCT batsman,count(total_runs) 
from ipl.deliveries
where total_runs in (4,6)
group by batsman 
order by 2 desc
limit 10;


# Top 20 bowlers who conceded highest extra runs
--------------------------------------------------
select bowler,sum(extra_runs) as 'highest extra runs' 
from ipl.deliveries
group by bowler
order by 2 desc
limit 20;


# Top 10 wicket takers
------------------------
select bowler,count(player_dismissed) as NoWicket_Taken
from ipl.deliveries
group by bowler
order by NoWicket_Taken desc
limit 10;


# Name and number of wickets by bowlers who have taken more than or equal to 100 wickets in ipl
------------------------------------------------------------------------------------------------
SELECT bowler,
       COUNT(player_dismissed) AS NoWicket_Taken
FROM ipl.deliveries
GROUP BY bowler
HAVING COUNT(player_dismissed) >= 100
ORDER BY NoWicket_Taken DESC
LIMIT 10;


# Top 2 player_of_the_match for each season
---------------------------------------------
select season, player_of_the_match, CNT 
from
(
select row_number() over (partition by season) as
rn,season,player_of_the_match,Cnt
from (
select season,player_of_the_match, count(player_of_the_match) 
as Cnt
from ipl.matches
group by season,player_of_the_match
order by 1 asc,3 desc
 ) rw
) Temp
where Temp.rn<3;


# Window Functions - (CTE) -- Combine column date from matches with table
-------------------------------------------------------------------------- 

WITH t1 as (select id,season,date_,city,team1,team2,winner from ipl.matches),
     t2 as (select matchid,batting_team,bowling_team from ipl.deliveries)
select distinct 
t1.season,t1.date_,t1.city,t1.team1,t1.team2,t2.batting_team,t2.bowling_team,
t1.winner
 from t1 inner join t2 on t1.id=t2.matchid
