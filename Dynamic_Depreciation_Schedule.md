## Dynamic Depreciation Schedule

Easily and Accurately Report Depreciation Expense

Compare Monthly & Annual Trends

---

<img src="images/excel01.PNG?raw=true"/>

**Project description:** This schedule represents a complete list of fixed assets with depreciation scheduled out over time.  The goal was to modify a previously existing Excel file to automate as much of the depreciation schedule as possible. This was accomplished by creating complex formulas that removed the manual calculations previously required to maintain the file.  From this assets can be easily compare over specified months and reporting depreciation expense is made simple.

**Background:** This schedule uses a fictitious company (Fruits & Nuts, Inc.) and its assets to demonstrate its capabilities. The company operates on a fiscal calendar utilzing the 4-4-5 method of accounting. The fiscal year is comprised of 4 quarters with each quarter starting with two 4-week months and ending with a 5-week month. This allows months to be more accurately compared based on the number of weeks in the month despite each calendar month having an unequal amount of days.

---

### 1. Define Current Month

<img src="images/excel02.PNG?raw=true"/>

As a user, the first thing you will want to do is to use the highlighted cell to specify which fiscal month you are reporting for. By clicking on the cell you can choose from the drop-down list. Specifying the month will be used to determine the current depreciation on each asset for that point in time. 

---

### 2. Asset Specifications

<img src="images/excel03.PNG?raw=true"/>

Then comes the qualitative aspects of each asset. This will provide information on the type of asset (such as Leasehold Improvement, Machinery and Equipment, etc.). It details the project number, description, location, asset tag number, as well as the main vendor the asset was purchased from.

---

### 3. Recording Depreciation to General Ledger Journal

<img src="images/excel04.PNG?raw=true"/>

Next the ledger accounts are listed based on what Financial Statement accounts will be affected by the assets – namely Accumulated Depreciation and Depreciation Expense. These accounts will be referenced via formulas later on in order to automatically generate monthly Depreciation Expense Journals.

---

### 4. Asset Life Details

<img src="images/excel05.PNG?raw=true"/>

This part of the schedule holds the quantitative data of the assets. As you can see, only a few fields need to be manually entered when setting up a new asset. This information will be used in the formulas of other fields for calculation.

*	*Depreciation Status* : uses formula to determine status
*	*Fully Depreciated* : IF End of Life < = Current Month End Date
*	*Depreciating* : IF End of Life > Current Month End Date
*	*Ending Depreciation* : IF End of Life is during the Current Month
*	*Fiscal Month* : **_manually enter_**
*	*Beg of Life* : uses VLOOKUP to determine start of Fiscal Month
*	*End of Life* = Beg of Life + (365 * Years)
*	*Years* : **_manually enter_**
*	*Weeks* = Years * 52
*	*Weeks in service* = (Current Fiscal Month – Beg of Life) / 7
*	*Weekly Depreciation* = Acquisition Cost / Weeks
*	*Acquisition Cost* : **_manually enter_**
*	*Accumulated Depreciation Total* : Sum of all YTD depreciation
*	*NBV (Net Book Value)* = Acquisition Cost – Accumulated Depreciation Total

---

### 5. View Years of Depreciation at a Glance

<img src="images/excel07.PNG?raw=true"/>

Next is the YTD Depreciation portion of the schedule. Through the use of complex nested Excel functions we can visualize year-to-date depreciation over time. In this case, we can see over a decade’s worth of depreciation by year in order to make comparisons and spot trends. For this example the current fiscal month is set to April 2021. Thus FY21 YTD depreciation is through the current month and does not extend past (notice that FY22 is zero). We will visit how this formula works later on.

---

### 6. View Individual Month's Depreciation

<img src="images/excel08.PNG?raw=true"/>

Finally, the last portion of the schedule contains the calendarized depreciation. This is the most detailed layout of depreciation expense as it shows you asset depreciation for every month from the date the asset is set up through being fully depreciated.

<img src="images/excel09.PNG?raw=true"/>

From this detailed view you can compare month-over-month, rolling trends as well as getting a solid idea of what future month’s depreciation will look like (less any future additions). You can also visualize when assets are added and others finish depreciating to help understand fluctuations in expense every month.

---

### 7. Automating Monthly Depreciation Calculations

The cells that populate the monthly depreciation section of the schedule utilize the most complicated function of the sheet. The purpose of the function is to calculate a specific asset’s depreciation for a given month while eliminating the possibility for human error otherwise possible with manual, time-consuming calculations. It needed to be developed to handle multiple situations and also be uniform enough to be used throughout this section without the need for alteration for each month/year. 

First, we will discuss the calculations and what-if scenarios that need to be performed at a high level to understand the use of this function.

A)	Has this asset already finished depreciating?
B)	Is this the final month of depreciation?
C)	Has this asset even started depreciating in this month? 
D)	Is this asset currently depreciating this month? If so, is this a 4 or 5 week month?

---

### 8. Understanding the Monthly Depreciation Formula

<img src="images/excel10.PNG?raw=true"/>

Let’s take this cell for example and see how the formula completes all of the objectives and returns the correct depreciation for the month.

**Original Formula:**

<img src="images/form01.PNG?raw=true" width="100"> 

Despite being a complex formula with many nested functions, if we translate it into plain English and break it up into parts we can see how the logical order of operations accomplishes our end goal.

<img src="images/form02.PNG?raw=true"/>

It is easier to understand if we reintroduce our original questions and dissect the formula into parts:

**A) Has this asset already finished depreciating?**
First of all, let’s test whether the asset in question has finished depreciating already. If the end of life date is less than or equal to the last day of the prior fiscal month, then that means the asset has finished depreciatin and should return a zero for the current month.

<img src="images/form04.PNG?raw=true"/>

**B) Is this the final month of depreciation?**
If the asset fails the first test it will move on to the next test.This part of the formula will test whether the asset is going to finish depreciating in the current period. We can test that by seeing if the end of life date is BOTH greater than the prior month end date AND lesser than or equal to the current month end date (meaning it must fall in between). If this is true, then we are going to finalize the depreciation of this asset by expensing the remaining balance of depreciation. The remaining value of the asset is the original acquisition cost and less all depreciation up until this point (using sum, offset and match).

<img src="images/form05.PNG?raw=true"/>

**C) Has this asset even started depreciating in this month?**
If the asset has failed both prior tests, then we will now test if the asset is even depreciating at all during this month. For example, if you set the current month to April 2021 then nothing from May 2021 should be depreciating. This portion is included so that the formula can remain constant across all months (past, present, future). If the beginning of life date is greater than the current month end date, then return zero because it has not begun depreciating the current month.

<img src="images/form06.PNG?raw=true"/>

**D) Is this asset currently depreciating this month? If so, is this a 4 or 5 week month?**
Lastly, if the asset has failed all other tests then that indicates that it is depreciating through the current month. To reiterate, this means that it is not already fully depreciated, nor a future asset, nor does it end somewhere in the middle of the month. More often than not the assets should fall into this category. The prior tests were merely to test and remove outliers. If an asset is going to depreciate as usual through the month then we need to determine how much depreciation will be booked based on whether the current fiscal period is a 4 or 5 week month. So we will take the weekly expense allocation and multiply it by the number of weeks in the month to get the depreciation expense.

<img src="images/form07.PNG?raw=true"/>

By using a multitude of nested IF functions we can create a rather complex all-in-one formula which accounts for the past, present and future to return the correct depreciation amount for the month for a given asset. The beauty of this is that a user of this depreciation schedule does not have to individually test all of these scenarios and then manually calculate depreciation for each asset. Doing so would inevitably result in human error inacuracies. In addition, this formula allows you to easily view how depreciation changes month to month with regards to the length of the month and the status of the asset.

---

### 9. Automating Year-to-Date Depreciation Calculations

<img src="images/excel17.PNG?raw=true"/>

Now that we are familiar with the calendarized depreciation schedule, we can return to the YTD part of the sheet and examine the calculation used to determine that total. 

Once again, let’s break out this formula and see how it works.

**Original Formula:**

<img src="images/form08.PNG?raw=true"/>

**Dissected:**

<img src="images/form09.PNG?raw=true"/>

If the current month end date is greater than the last day of the fiscal year in question, that means that it has been depreciating over the whole year thus output the sum of the whole year. For example, if the current fiscal year is 2021, then for YTD 2020 give me the sum of all depreciation in 2020. 

<img src="images/form10.PNG?raw=true"/>

Otherwise, give me the sum of depreciation from the start of the year up until the current month. For example, let’s say the current month is April 2021 and the fiscal year runs from November 2020 – October 2021. In that case, the YTD depreciation is from November ’20 – April ’21.

---

### 10. Viewing Fixed Asset Totals

<img src="images/excel21.PNG?raw=true"/>

At the very bottom of the sheet is a summary view of all categories. This allows for easy comparison to the Depreciation Journals and the General Ledger to ensure that everything ties out completely.

---

### 11. Automatically Generated Monthly Depreciation Journal

The second tab of the Fixed Asset Schedule contains the Depreciation Journal to be booked monthly. It utilizes SUMIF, MATCHing and INDEXing to pull the sum of depreciation expense for each ledger account. The total journal can easily be compared to the totals from the schedule to ensure accurate reporting. 

<img src="images/excel22.PNG?raw=true"/>
