## DYNAMIC DEPRECIATION SCHEDULE

Easily and Accurately Report Depreciation Expense

Compare Monthly & Annual Trends

**Project description:** This schedule represents a complete list of fixed assets with depreciation scheduled out over time.  The goal was to modify a previously existing Excel file to automate as much of the depreciation schedule as possible. This was accomplished by creating complex formulas that removed the manual calculations previously required to maintain the file.  From this assets can be easily compare over specified months and reporting depreciation expense is made simple.

**Background:** This schedule uses a fictitious company (Fruits & Nuts, Inc.) and its assets to demonstrate its capabilities. The company operates on a fiscal calendar utilzing the 4-4-5 method of accounting. The fiscal year is comprised of 4 quarters with each quarter starting with two 4-week months and ending with a 5-week month. This allows months to be more accurately compared based on the number of weeks in the month despite each calendar month having an unequal amount of days.

### Define Current Month

As a user, the first thing you will want to do is to use the highlighted cell to specify which fiscal month you are reporting for. By clicking on the cell you can choose from the drop-down list. Specifying the month will be used to determine the current depreciation on each asset for that point in time. 

### Asset Specifications

Then comes the qualitative aspects of each asset. This will provide information on the type of asset (such as Leasehold Improvement, Machinery and Equipment, etc.). It details the project number, description, location, asset tag number, as well as the main vendor the asset was purchased from.

### Recording Depreciation to General Ledger Journal

Next the ledger accounts are listed based on what Financial Statement accounts will be affected by the assets – namely Accumulated Depreciation and Depreciation Expense. These accounts will be referenced via formulas later on in order to automatically generate monthly Depreciation Expense Journals.

### Asset Life Details

This part of the schedule holds the quantitative data of the assets. As you can see, only a few fields need to be manually entered when setting up a new asset. This information will be used in the formulas of other fields for calculation.

⋅⋅*	Depreciation Status : uses formula to determine status
⋅⋅*	Fully Depreciated : IF End of Life < = Current Month End Date
⋅⋅*	Depreciating : IF End of Life > Current Month End Date
⋅⋅*	Ending Depreciation : IF End of Life is during the Current Month
⋅⋅*	Fiscal Month : **_manually enter_**
⋅⋅*	Beg of Life : uses VLOOKUP to determine start of Fiscal Month
⋅⋅*	End of Life = Beg of Life + (365 * Years)
⋅⋅*	Years : **_manually enter_**
⋅⋅*	Weeks = Years * 52
⋅⋅*	Weeks in service = (Current Fiscal Month – Beg of Life) / 7
⋅⋅*	Weekly Depreciation = Acquisition Cost / Weeks
⋅⋅*	Acquisition Cost : **_manually enter_**
⋅⋅*	Accumulated Depreciation Total: Sum of all YTD depreciation
⋅⋅*	NBV (Net Book Value) = Acquisition Cost – Accumulated Depreciation Total

### View Years of Depreciation at a Glance

Next is the YTD Depreciation portion of the schedule. Through the use of complex nested Excel functions we can visualize year-to-date depreciation over time. In this case, we can see over a decade’s worth of depreciation by year in order to make comparisons and spot trends. For this example the current fiscal month is set to April 2021. Thus FY21 YTD depreciation is through the current month and does not extend past (notice that FY22 is zero). We will visit how this formula works later on.

### View Individual Month's Depreciation

Finally, the last portion of the schedule contains the calendarized depreciation. This is the most detailed layout of depreciation expense as it shows you asset depreciation for every month from the date the asset is set up through being fully depreciated.

From this detailed view you can compare month-over-month, rolling trends as well as getting a solid idea of what future month’s depreciation will look like (less any future additions). You can also visualize when assets are added and others finish depreciating to help understand fluctuations in expense every month.

### Automating Monthly Depreciation Calculations

The cells that populate the monthly depreciation section of the schedule utilize the most complicated function of the sheet. The purpose of the function is to calculate a specific asset’s depreciation for a given month while eliminating the possibility for human error otherwise possible with manual, time-consuming calculations. It needed to be developed to handle multiple situations and also be uniform enough to be used throughout this section without the need for alteration for each month/year. 

First, we will discuss the calculations and what-if scenarios that need to be performed at a high level to understand the use of this function.

1.	Has this asset already finished depreciating?
2.	Is this the final month of depreciation?
3.	Has this asset even started depreciating in this month? 
4.	Is this asset currently depreciating this month? If so, is this a 4 or 5 week month?

### Understanding the Monthly Depreciation Formula

Let’s take this cell for example and see how the formula completes all of the objectives and returns the correct depreciation for the month.

Original Formula:
=IF($M23<=FP$6,0,IF(AND($M23> FP$6, $M23<=FQ$6),$R23-(SUM(OFFSET($AJ23,0,0,1,MATCH(FP$6,$AJ$6:$GI$6)))),IF($L23> FQ$6,0,(IF($M23> FQ$6,$Q23*FQ$4)))))

Despite being a complex formula with many nested functions, if we translate it into plain English and break it up into parts we can see how the logical order of operations accomplishes our end goal.

=IF(END OF LIFE DATE<=PRIOR MONTH END DATE,0,IF(AND(END OF LIFE DATE > PRIOR MONTH END DATE, END OF LIFE DATE <=CURRENT MONTH END DATE),ACQUISITION COST-(SUM(OFFSET(START OF SCHEDULE PERIOD,0,0,1,MATCH(PRIOR MONTH END DATE,ALL SCHEDULE PERIODS)))),IF(BEGINNING OF LIFE DATE> CURRENT MONTH END DATE,0,(IF(END OF LIFE DATE > CURRENT MONTH END DATE,WEEKLY DEPRECIATION AMOUNT*NUMBER OF WEEKS IN THE MONTH)))))


### Define Current Month



### 1. Suggest hypotheses about the causes of observed phenomena

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. 

```javascript
if (isAwesome){
  return true
}
```

### 2. Assess assumptions on which statistical inference will be based

```javascript
if (isAwesome){
  return true
}
```

### 3. Support the selection of appropriate statistical tools and techniques

<img src="images/dummy_thumbnail.jpg?raw=true"/>

### 4. Provide a basis for further data collection through surveys or experiments

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. 

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
