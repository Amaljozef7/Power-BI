# SLC Maximum Offers Prediction

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/1682fb14-e8d5-4e78-8fbc-701317bfa3ca)


### Disclaimer
This file provides a brief overview of the dashboard that I have created. Since the data is confidential, the information used below is fictitious and not real. The values shown below are not real. Due to the confidentiality of the data, I am unable to provide further details. 

## Overview

This dashboard will assist the college's admission department in projecting the maximum number of offers they need to send out every semester. By accurately predicting the number of offers required, they can secure the necessary confirmations for their annual application targets. Currently, the data is scattered across three different locations, and staff must manually download and calculate it using Excel spreadsheets. This dashboard will automate the process, displaying all the necessary figures automatically.

### Steps followed 

- Step 1 :Connecting Data to Power BI from the sharepoint folder in which the file is stored
    
        1. Click on Get Data in powerquery
        2. Click more and navigate to SharePoint Folder
        3. Enter the SharePoint URL in the next dialog box.
        4. Sign into the organizational account.
        5. Then click on the Transform.
        6. This will create a query. We name this query as sharepoint.

#### The data file is stored in three different locations on SharePoint. The concept outlined below involves connecting to these folders, and when a new file is added, Power BI will automatically detect the data and update the dashboard. 



- Step 2 - Connecting to different folders in the sharepoint.
    
        1. Right click on the sharepoint query.
        2. Click reference. A new query will be created. Rename the query.
        3. Copy the sharepoint folder path.
        4. Copy and paste the sharepoint folder path in below link and put it in the folrmula bar of P.

        Table.SelectRows(Source, each ([Folder Path] = " = Table.SelectRows(Source, each ([Folder Path] = "https://my.sharepoint.com/folderpath/"))

        5. This will update query with the details of all the file present in the folder.
        6. Do the same steps for all folder.

 ![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/a6b51ffa-7773-4823-853e-eed05ca23745)


- Step 3 Transforming the Data
#### Tracking Reports
        1. Click on the 2 down arrows near to contents.
        3.	Click on the parameter and click 'OK'. This will create a sample file to transform data.
        4.	Go to 'Transform Sample File' from Source_Tracking.
        5.	Remove the last 2 auto-generated steps: 'Promoted Headers' and 'Change Type'.
        6.	Go to the Navigation steps. Update the formula like this (it refers to the first sheet in the spreadsheet): = Source{0}[Data]. This is because sheet name is different in different files.
        7. The campus name is not present in every row. So we will do a fill down.
        8. Removed all unncessary columns.
        9. Select the following columns and unpivot other columns: "Source.Name", "Column1", "Column3", "Column4", "Column5", "Column6", "Column7".
        10. Different file has different column names. So it is renamed using below formula.
        = Table.RenameColumns(#"Filtered Rows3",{{"Column1","Campus"},{"Column3", "Cluster"}, {"Column4", "Program Name"}, {"Column5", "Program Code"}, {"Column6", "Program Plan"}, {"Column7", "Program Choice"}}).
        11. The spreadsheet is saved in the SharePoint folder with the format 'Year-Semester'. This information is present in the first column. Extract this information from the first column and split the 'Source.Name' column using a hyphen as a delimiter.
        12.	Then split the second column by a dot as a delimiter to get the Semester name. Remove the column containing the 'xlsx' value.
        13. Disable Load for all the queries.



- Step 4 : Creating Fact and dimension Table
The reason we are building a dimension table is to create a star schema model. This will help in avoiding excessive data in the fact table. Since the data is substantial, loading times will be high. Additionally, since we are connected to a SharePoint folder, it will take more time. The star schema will also facilitate easier implementation of slicers.
Suggestions: We need to avoid including descriptive data in the fact table. However, due to the complexity of our data, we have included some descriptive data in the fact table. In the future, we should try to avoid incorporating descriptive data into the fact table.

Following dimension tables were created.

            1. Dim Program Code
            2. Dim Campus
            3. Dim Program Code
            4. Dim Due date.
            5. Dim Level - This table is created for sorting purposes, so that applications will be displayed first, followed by offers, and so on
            7. Dim Category

Step 5 - Calaculated Tables

#### Offer Projection.

This is used to create a parameter to increase or decrease offers. For example, if the college wants to see how it looks when they increase offers by 10%, they can do so by sliding the slicer.

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/35d6cd24-4779-4460-885c-49351de8143b)

    1.	Go to Modeling Ribbon.
    2.	Click on the New parameter.
    3.	Select numeric range.
    4.	Set minimum numeric range to -100 and maximum to 100.
    5.	Click ok. This will create a table and which contain 2 items parameter and a measure Parameter value. 

- Step 7 - Calculated measure

##### 1.Total Applications

        CALCULATE(SUM('Fact'[Value]), 'Fact'[Level] IN {“Applications" })
 
Use above formula for calculating for other totals like offers, Payment, Confirmation received, Registrations.

##### 2. Max offer Projections

Add the maximum offer to the percentage of the maximum offer value according to the user input in the parameter.
            
     Max Offers Projections = INT(ROUND([Max Offers] + [Max Offers]* SELECTEDVALUE('Offer Projection'[Parameter]) / 100,0))


##### 3. Total Audit Value

In the calculated table within the fact table, we have a column called "Next Year," which represents the current year plus one (i.e., the next year). This is done to display values from the previous year alongside the audit value of the current year. For example, when predicting the maximum offer value for 2023, the audit value will be for 2023, while other metrics like offers and confirmations will be from 2022.

In the formula, you can see "Next Year+1" used for the "Year" variable. This may appear a bit confusing. In the fact table, there are two columns: "Year" and "Next Year." If the current year is 2023, the "Next Year" column's value will be 2024. The "Selected Value" function searches for the value in the slicer, which is based on the "Next Year." As previously mentioned, since we are displaying the audit value of the current year, it is stored in the "Next Year". For example, the value for 2023 is stored in 2024. This is why we use "Next Year+1.


    -- Creating Variable
    -- Variable to filter summary
    VAR SelectedCategory = "Summary"
    -- Only domestic is filtered 
    VAR SelectedLevel = "Domestic"
    -- Selecting the Next Year+1
    VAR SelectedYear = SELECTEDVALUE('Fact'[NextYear])+1
    VAR prevYear = SelectedYear
    Var term = SELECTEDVALUE('Fact'[Term])
    RETURN
    CALCULATE(
    SUM('Fact'[Value]),
    FILTER(
        ALL('Fact'),
        'Fact'[Category] = SelectedCategory &&
        'Fact'[Level] = SelectedLevel &&
        'Fact'[Next Year] = prevYear &&
        'Fact'[Term] = term &&
        'Fact'[Program Code] IN VALUES('Fact'[Program Code]) &&
        'Fact'[Program Choice] IN VALUES('Fact'[Program Choice])
    ))
##### 4. Competition
Classfying Programs according to the Competition.

    Competition = 
    SWITCH(
    TRUE(),
    'Fact'[Comp_Perc]>0.60,"Highly Competitive",
    'Fact'[Comp_Perc]>0.40,"Competitive",
    'Fact'[Comp_Perc]>0.20,"Moderately Competitive",
    'Fact'[Comp_Perc]>0.10,"Less Competitive",
    'Fact'[Comp_Perc]>0.02,"Least Competitive",
    "Not Competitive"
)

##### 5. Conversion Rate

Divide previous Year Total confirmation by total year. 
    
    Conversion Rate = DIVIDE([Prev Total Confirmation],[Prev Total Offer],0)

##### 6.Calculated Max Offers

    Calculated Max_Offers = INT(DIVIDE([Total Audit#],[Conversion Rate],0))

- Step 6 - Dashboard Creation

    #### 1. User Guide

Since data is stored in various locations, we need to download it from different sources and store it in a folder. To simplify this task, our Power BI dashboard is designed to navigate directly to these folders. The first page of the dashboard contains various buttons for folder navigation.

        1.	Go insert a tab and create a button. 
        2.	In the format tab click on Action and update URL.


#### 2. Admission & Enrollment

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/96cf8088-1bb0-4bc4-91ae-941c685c4a56)

Matrix
Values:
Rows - Program names, Program Code, Program Choice

Columns - Year

Values – Total Applications , Under Review, Total Offers, Total Payments, CAP#, Max Offers, Withdrawn November Registrations.
Rows: -
Program Name, Program code
Columns: -
Year
2.	Slicer Added – Year, Program, Program Code, Program Choice.

#### 3. Capacity and Audit

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/dc2f48f9-d015-4aea-b814-d78204909824)

#### 4. Max Offer Projections

In this dashboard, the user enters a value in the parameter section. This value represents the percentage increase or decrease that the user requires in the number of offers to be sent in the next year.

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/8b7c7c3f-a92a-4c1d-acb8-e925edb4064a)

#### 5.	Conversion Rate
It calculates the maximum number of offers that should be sent based on the previous year's offers and confirmations. For example, in the year 2023, for the Business course, the Total Audit Value, which represents the target value, is set at 19. In the previous year, the college sent a total of 21 offers but received only 8 confirmations. As a result, the conversion rate was 0.38. To achieve a target of 19 for 2023, they would need to send 49 offers.

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/ace4083d-8062-4563-a0bb-6b6e2a506945)

#### 6.Estimate Projections
The idea here is that there is an estimated number of max offers. However, in the previous year, for some programs, the college might have sent more offers than the estimated value. In such cases, we calculate the increase in the percentage of offers sent. Then, we add this percentage increase to the current estimated offer. However, in our calculations, we are not taking this into account. Instead, we divide the percentage increase by 2 and calculate the projected max offers by adding this value to the estimated max offers in between other estimates.

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/7db376e8-f985-48e0-a264-e4709b98140a)

#### 7.	Date Level

These are certain Due dates for the college. This data visulize according to the due dates.

1 February – Due date for sending applications.

1 May – Due date for sending confirmation to offer issued.

15 June - Due date for making Payment.

1 September - Due date for Registration.

1 November - Due date for Audit.

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/d224d86a-f45b-4b4e-a4ff-75a789b275a2)

#### 8 Program Competitiveness
This dashboard will take the highest payment from the payments columns. Then it will calculate the 60 percent of it. All the program above 60 percent is considered as competitive program. All the program above 40 are competitive. Like wise it will classify programs according to the payment.

![Snap_1](https://github.com/Amaljozef7/Power-BI/assets/134343054/147233b5-bdba-4da0-8110-028ecce8aae6)

