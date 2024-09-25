# Deciphering Data Architectures  
#### Choosing Between a Modern Data Warehouse, Data Fabric, Data Lakehouse and Data Mesh.

## Part I. Foundation

- 1. [Big Data](#big-data)
- 2. [Types of Data Architectures](#types-of-data-architectures)
- 3. [The Architecture Design Session](#the-architecture-design-session)

## Part II. Common Data ARchitecture Concepts

- 4. [The Relational Data Warehouse](#the-relational-data-warehouse)
- 5. [Data Lake](#data-lake)




### Introduction Data Architectures

"If you think good data architecture is expensive, try a cheap one"

It is common knowledge that the growth of data has continued and continues to grow over the years, companies have realised this too and are aware of the benefits it can bring. That is why they have thrown themselves into the creation of data architectures. 
The problem is that choosing the right architecture to suit every need is not as easy as it might seem at first glance.
The author talks about a $100M project that had to be scrapped and started from scratch due that the architecute used the wrong technology.

Its all about getting the right information to the right people ar the right time in the right format.

### Big Data

**Introduction to Big Data**

Big data encompasses all data, regardless of size, speed, or type. It can be defined using the "six Vs" of big data:

`Volume`: The sheer amount of data, ranging from terabytes to petabytes, from sources like social media and IoT devices.  
`Variety`: Different data formats including structured (relational databases), semi-structured (logs, CSV, XML, JSON), unstructured (emails, documents), and binary data (images, audio, video).  
`Velocity`: The speed at which data is generated and processed, ranging from batch processing to real-time streaming.  
`Veracity`: The accuracy and reliability of data, which can be affected by sources' reliability.  
`Variability`: The consistency of data in terms of format, quality, and meaning.  
`Value`: The usefulness and relevance of data for gaining business insights and making informed decisions.  

Big Data v´s have been increasing over the years, at first the where only 3. Accodring to the Gartner group you needed to have 2 of those v´s to be able to talk about big data.
In other words, if you only had a large volume of data, but it didnt not grow or did no have different formats, then it was not big data, no matter how mcuh data you had.


**Data Maturity**  
Data maturity describes the stages of development in an organization’s ability to manage and utilize data

`Reactive`: Data is scattered and managed informally, often leading to inefficiencies and inconsistencies.  
`Informative`: Data centralization improves analysis and reporting, but solutions are often not scalable and are limited in data handling capabilities.  
`Predictive`: Advanced analytics and machine learning are incorporated for real-time decision-making, often facilitated by cloud-based solutions.  
`Transformative`: Capable of handling any data type and volume, with a focus on self-service business intelligence (BI) where non-technical users can easily generate reports and dashboards.  
`Self-Service Business Intelligence (BI)`: Traditional BI, which relies on IT to generate reports, is being replaced by self-service BI, enabling end users to directly access and analyze data. This shift requires a data architecture built with end users' needs in mind, promoting efficiency and reducing reliance on IT.  


### Types of Data Architectures

`DWH` Relational data consumption, with difficulties to scale in terms of cost and volume.  

`Data Lake` Introduced in 2010, is like a giant repository of files that had the potential to be consumed. It comes right on the heels of the rise of Big Data. It solves the scalability problems of DWH
but encounteres several obstagles:
 - `managemet` Lack of ACID and lineage properties led to redundat data problems.   
 - `lack of skills` end users were far from being able to use this data properly (jupyter notebooks etc...).  
 
`MDW modern Data Warehouse`: combine the benefits of both relationa DWH and Data Lake integrating various technologies to support data staging, preparation, security and compliance.
 - While Lakes primarily store raw, unstructured data, MDL support both.  
 - Performance: Combining the processing power of DWH with the storage capacity of Data lakes, MDW offer improve performance.  
 - Real-Time Data Processing.

`Data Fabric`: Evolution of MDW with more technology adde to source more data, secure it, and make it available. 

`Data Lakehouse` Become popular in 2020 when Databricks started using the term. The concept of a data lake in your architecture. All types of data are ingested into the data lake, and all queries and reports are done from the data lake.  
 
`Data Mesh`
At a glance, it looks a bit like parallel processing. Instead of having many data sources that deploy in one place where teh team of data engineers process the data and clean it etc..  
You have a bunch of small teams of engineers that process teh data and clean it from the source.  

(.img/Data_Architectures_Picture.PNG)

### The Architecture Design Session

Architectue Design Session (ADS, its a hight-level design of a solution to clllect data for specific business opportunities.  
The first ADS is the start of the architecture process and will lead to many more discussions.

#### Why hold an ADS ?

Basically because they are crucial to build a successful data solution. 

`Arcuitecture design session` is a structure discussion with business and technical stakeholders that´s drive in a high-level design of soluto to collect data from specific business oportunities.  

The ADS produce two ouputs:

- An architecture that can server as starting point for the data solution.  
- A high-level plan of action, which may include follow-on demostrations, proofs of concept and product discussions.  


ADS is about to includ your organization customers and/or partner and discuss about current enviroment and pain point that were blocking progress.  


**Before the ADS**
- `Preparing:` You have to prepare all the needs, physical / virtual , enought room.
    Make sure find out the project budget and timeline and identify the decision maker.  
	
	- customer´s problems with their current architecture.  
	- How can you help the customer.    
	- Account Teams goals and the outcome they wan t from the meeting.  
	- How well the customer knows your products.  
	
![image](https://github.com/Enrique1987/data_solution_architect/blob/main/img/02_ADS_Meeting.PNG)
	

