# LIFE SPRING HOSPITAL ANALYSIS

## PREVIEW
- [PROJECT OVERVIEW](#project-overview)
- [DATA SOURCE](#data-source)
- [TOOLS](#tools)
- [DATA CLEANING](#data-cleaning)
- [EXPLORATORY DATA ANALYSIS](#exploratory-data-analysis)
- [DATA ANALYSIS](#data-analysis)
- [INSIGHTS](#insights)
- [RECOMMENDATION](#recommendation)

## PROJECT OVERVIEW
This project contains 10,000 Life Spring’s hospital record. The goal is to understand the factors that affect patient care quality. Identify areas where care can be improved using data insights. Propose simple, actionable solutions to enhance patient care.

## DATA SOURCE
The dataset used in this project was provided by Life Spring hospital.

## TOOLS
1.	EXCEL: The dataset was gotten as a CSV, so I used Excel to open it. Go through the dataset to know the information it contains as well as cleaning.
2.	SQL: The dataset was loaded to SQL for cleaning and querying.
3.	Power BI: This tool was used for visualization.

## DATA CLEANING
1.	Confirming the total normal of dataset and columns.
2.	Making sure they all represent their various data type.
3.	Checking for duplicates and null values.

## EXPLORATORY DATA ANALYSIS
Understand key variables and their relationships. Analyze Patient Care: Look for patterns, such as satisfaction scores by diagnosis, or follow-up care. 

# DATA ANALYSIS
~~~
SELECT * FROM hospital_info;

-- Data cleaning
-- total rows
select count(*)
from hospital_info;

-- column count
select count(column_name)
from information_schema.columns
where table_name = 'hospital_info';

-- data type
select column_name, data_type
from information_schema.columns
where table_name = 'hospital_info';

-- change column name
alter table hospital_info
rename column `Follow-up_Appointment` to FollowUp_Appointment;

-- duplicates and null values
with duplicate_cte as
(
select 
row_number() over (partition by Patient_ID, Age, Gender, Primary_Diagnosis, Comorbidities, Length_of_Stay, 
					Discharge_Destination, FollowUp_Appointment, Medication_Adherence, Care_Satisfaction) as rn
from hospital_info
)

select *
from duplicate_cte
where rn > 1
and rn is null;

-- EDA
select *
from hospital_info;

select 
max(age) as max_age,
min(age) as min_age,
round(avg(age), 0) as avg_age
from hospital_info;

-- distribution of gender
select Gender, count(*) as Patient_count
from hospital_info
group by 1
order by 2 desc;

-- distribution of diagnosis
select Primary_Diagnosis, count(*) as patient_count
from hospital_info
group by 1
order by 2 desc;

-- distribution of diagnosis by gender 
select Primary_Diagnosis, gender, count(*) as patient_count
from hospital_info
group by 1, 2
order by 1 desc;

-- avg length of stay
select round(avg(Length_of_Stay), 0)
from hospital_info;

-- distribution of dischagre destination
select Discharge_Destination, count(*) as Patient_count
from hospital_info
group by 1
order by 2 desc;

-- distribution of discharge distination and follow up
select Discharge_Destination, FollowUp_Appointment, count(*) as Patient_count
from hospital_info
group by 1, 2
order by 1;

-- distribution of medication adherence 
select Medication_Adherence, count(*) as Patient_count
from hospital_info
group by 1
order by 2 desc;

-- distribution of discharged with medical adherence
select Discharge_Destination, Medication_Adherence, count(*) as Patient_count
from hospital_info
group by 1, 2
order by 1;

-- distribution of patients by follow up
select FollowUp_Appointment, count(*) as patient_count
from hospital_info
group by 1
order by 2 desc;

-- follow up by disease
select Primary_Diagnosis, FollowUp_Appointment, count(*) as patient_count
from hospital_info
group by 1, 2
order by 1;

-- medication adherence by primary disease
select Primary_Diagnosis, Medication_Adherence, count(*)
from hospital_info
group by 1, 2
order by 1;

-- distribution of patients by care satisfaction
select Care_Satisfaction, count(*) as Total_number
from hospital_info
group by 1
order by 2;

with Patient_review as
(
select Primary_Diagnosis, 
	case 
		when Care_Satisfaction >=4 then 'Excellent'
        when Care_Satisfaction = 3 then 'Satisfactory'
        else 'Poor'
	end as Care_satisfaction_grade
from hospital_info
)

select Primary_Diagnosis, care_satisfaction_grade, count(*) as Patient_count
from patient_review
group by 1, 2
order by 1, 3 desc;

-- what age group is most affected by each diagnosis
with rank_order as 
(
    select Primary_Diagnosis,
		case 
			when Age between 18 and 24 then 'Young Adult'
			when Age between 25 and 44 then 'Adult'
			when Age between 45 and 64 then 'Middle aged'
			when Age between 65 and 90 then 'Elderly'
		end as Age_group,
count(*) as Patient_count,
rank() over (partition by Primary_Diagnosis order by count(*) desc) as rank_order
from hospital_info
group by Primary_Diagnosis, Age_group
)

select Primary_Diagnosis, Age_group, Patient_count, rank_order
from rank_order;

-- age group distributions
select 
	case 
		when Age between 18 and 24 then 'Young Adult'
		when Age between 25 and 44 then 'Adult'
		when Age between 45 and 64 then 'Middle aged'
		when Age between 65 and 90 then 'Elderly'
	end as Age_group,
count(*) as Patient_count
from hospital_info
group by age_group
order by 2 desc;

~~~
## INSIGHTS
1.	Average length of stay is 16 days. 
2.	There is a predominance of male patients over female patients, with 5019 males and 4981 females.
3.	The patient distribution for Primary Diagnosis is as follows: Injuries (2038), Respiratory problems (2017), Cardiac issues (2001), Diabetes (1993), and Infections (1951).
4.	The patient distribution for Age group is as follows: Elderly (3510), Middle Aged (2777), Adult (2725), Youth (988).
5.	Patients were discharged to: Rehabilitation center (3387 patients), Nursing facility (3330 patients), and Home (3283 patients).
6.	Patients WITHOUT follow-up appointment (5095), and this may impact recovery and readmission rate. Patients WITH follow-up appointment (4905).
7.	Elderly patients are the most affected across all Primary Diagnosis.
8.	Patients that adhered to medication are 4995, while non-adherers are 5005 (nearly half of the patients).
9.	Patient Review summary: Excellent (4044), Poor (4001), Satisfactory (1955).
10.	Patient Satisfaction Grades by Primary Diagnosis:
    -	Cardiac Issues:
        - Poor: 819
        - Excellent: 775
        - Satisfactory: 407 
    -	Diabetes:
        - Poor: 747
        - Excellent: 833
        - Satisfactory: 413 
    -	Infections:
        - Poor: 780
        - Excellent: 808
        - Satisfactory: 363 
    - Injuries:
        - Poor: 824
        - Excellent: 835
        - Satisfactory: 379
    - Respiratory Problems: 
        - Poor: 831
        - Excellent: 793
        - Satisfactory: 393 
Patients with diabetes reported the highest excellent satisfaction rate (833), while those with respiratory problems reported the highest poor satisfaction rate (831).

## RECOMMENDATION

1.	Reduce Length of Stay: Optimize care pathways for earlier discharge. 
2.	Improve Follow-up Rates: Implement a better scheduling system and patient reminders to ensure follow-up care. 
3.	Enhance Respiratory Care: Since respiratory patients have the highest poor satisfaction rates, assess treatment plans and improve patient education on condition management.
4.	Personalized Care for Elderly Patients: Given their high numbers and likely complex needs, invest in specialized geriatric care programs. 
5.	Optimize Discharge Planning: Ensure patients discharged to rehab and nursing homes receive structured transition plans to prevent readmissions. 
6.	Monitor and Improve Patient Experience: Address key areas of dissatisfaction through staff training, better communication, and real-time patient feedback mechanisms.
7.	Patient Education & Counseling: Educate patients on the importance of medication adherence, potential side effects, and long-term benefits. 
8.	Provide easy-to-understand medication guides, especially for elderly patients.
9.	Address Financial Barriers: Assess if cost is a barrier and offer information on generic alternatives, insurance coverage, etc.
10.	Collect feedback from non-medication adherence patients to understand specific reasons for non-compliance and tailor interventions accordingly.
11.	Collection of patients reviews for sentimental analysis.





