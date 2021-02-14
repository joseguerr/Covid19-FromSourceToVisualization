# Covid19-FromSourceToVisualization


### GOAL: 
	set up the data structure to help Data Scientists and Data Analysts important insights related to Covid-19. Data sources comes from european 
	institutions (ECDC and Eurostat).
	Information includes confirmed cases (daily basis), Mortality (daily basis), Hospitalization/ ICU Cases (per week and at the end of the day),
	Testing numbers, Number for every country's population by age group.
	Predict the raise in cases and mortality rate
	
### Analysed Data: 
	Covid-19 new cases and deaths by country (ECDC)
	Covid-19 Hospital admissions & ICU cases (ECDC)
	Covid-19 Testing numbers (ECDC)
	Country Response to Covid-19 (lockdowns or other type of measures) (ECDC)
	Country population by age (EuroStat)
	  

### END-TO-END DATAFLOW:
	ingestion of data is made through the http and azure blob connector to ADF and then stored in azure data lake.
	Transformation is done in ADF through dataflow, hdinshight and databricks and storage in SQL database (for publishing) or data lake for ML models
	
### CONSIDERATIONS:
	Structure of the data
	Operational needs - how frequently is the data accessed, how quickly we need to serve it, simple or complex queries, access from multiple regions
	Database Requirements
	Storage Requirements
	  
-------------------------------------//-------------------------------------------------------------------	  
	  
### DATA INGESTION 
  The population by age data set is ingested from Azure blob storage (copy activity)
  The remain are done through HTTP connection
	
### TRANSFORMATION 1 IN DATAFLOW:
	Cases and Deaths data: Raw file columns -> country, country_code, continent, population, indicator, daily_count, date, rate_14_day, source
						   Transformed columns -> country, country_code_2_digit (Lookup), country_code_3_digit, population, cases_count (indicator
												  + daily_count), deaths_count (indicator+ daily_count), reported date, source
	The first transformation is basicaly a Lookup table (dim) for the countries and respective atributes.
	FILTER by 'Europe' and country code not null
	
	<img src=“github.com\joseguerr\Covid19-FromSourceToVisualization\blob\main\snapshots\dataflow%20death%20cases.png" width=“964” height="auto">
	
	
### TRANSFORMATION 2 IN DATAFLOW:
	Create a dim date with date_key, date, year, day_name, day_of_year, week_of_month, week_of_year, month_name, year_month, year_week
	We have records of week and daily counts, which are going to be seperated in different streams.
	We use the condition Split since two of the indicators relate to daily and the other two relate to weekly counts
	we use the derive column transformation to create the year_week column (year_week in dim date is in yyyymm format so we cannot join from there)
	we use lpad
	hospital admissions have the year_week column as 2020-W01!
	when then place the expression from derive transformation in the aggregate schema, so we won't need the derived column anymore.
	For the join transformation we can use inner join because for every week coming from weekly transformation we are going to find 
	when you create the sink in data flow, the data is run in a spark cluster which means data our result will be distrubited through different files.
		Because of this, it is always better to create a separate folder to accomodate all the files.

### TRANSFORMATION 3 IN DATAFLOW:
	drop the columns until 2019 from population_by_age file
	also, in other files we have country, country_code, etc. There for we separate the first column into two, the first called age_group and the other
	called country_code. we can then perform a look up on dim_country.
	the third step is to pivot the table in order to have all the data from one country in one row with columns for the different age groups
	
	
### MAKING PIPELINES PRODUCTION READY
	Pipelines executions are full automated  (run at regular times or on an event occurring)
	Activities only run once the upstream dependency has been satisfied
	Easier to monitor for execution progress and issues
	
### MONITORING
	
	Created a diagnostic setting under the monitor blade to log activity, pipiline and trigger runs
  
  
### End User Visuals
