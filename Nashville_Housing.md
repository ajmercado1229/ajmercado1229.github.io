# Cleaning Nashville Housing Data with SQL Queries

Not all data comes in a nice and neat format. Many times datasets include erroneous data from the source it was retrieved which must be addressed and cleansed.
This dataset includes information on Nashville housing data from January 2013 - October 2016. Here are some queries that will help cleanse the data to make it more usable for future projects.

---

## 1) Standardize Date Format

Currently the SaleDate field is set in DATETIME format, but the timestamps all record zeros. We can add a new date field into the data and make it only a DATE.

```sql
ALTER TABLE NashvilleHousing
ADD SaleDateConverted DATE;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(DATE, SaleDate)
```

---

## 2) Populate Property Address Data

Some records contain a null where there should be a PropertyAddress. Luckily, ParcelID should be the same for the same property even if the addres is missing. We can populate the erroneous nulls by finding records with the same ParcelID and inserting the PropertyAddress from the non-null records. This will require a self-join.

First take an initial look at these null records to see what we are working with.

```sql
SELECT *
FROM PortfolioProject.dbo.NashvilleHousing
WHERE PropertyAddress IS NULL
```

Now write a join query using the ISNULL function and see if it populates the null records the way we want it to.

```sql
SELECT 
	a.ParcelID,
	a.PropertyAddress,
	b.ParcelID,
	b.PropertyAddress,
	ISNULL(a.PropertyAddress,b.PropertyAddress)
FROM PortfolioProject.dbo.NashvilleHousing AS a
JOIN PortfolioProject.dbo.NashvilleHousing AS b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL
```

Since we know that this fomrula works, write an UPDATE query to fix the null records by populating the PropertyAddress.

```sql
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
FROM PortfolioProject.dbo.NashvilleHousing AS a
JOIN PortfolioProject.dbo.NashvilleHousing AS b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL
```

---

## 3) Breaking out Address into Individual Columns (Address, City, State) from PropertyAddress

-- Break out the address into individual columns
PropertyAddress in this dataset is combined into one field. A query using substring can be written to split up the address into multiple fields. This will make it more usable and functional in the future.

```sql
SELECT
	SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) AS Address,
	SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) AS Address
FROM PortfolioProject.dbo.NashvilleHousing
```

Alter the table to add these additional columns and then load in the data from the above formulas.

```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255)

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1)

ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD PropertySplitCity NVARCHAR(255)

UPDATE NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress))
```

---

## 4) Breaking out Address into Individual Columns (Address, City, State) from OwnerAddress

Like PropertyAddress, the OwnerAddress has the same problem of being combined in one column. Alternative to the substring function, parsename can be used to solve this issue.

```sql
SELECT
	PARSENAME(REPLACE(OwnerAddress,',','.'), 3),
	PARSENAME(REPLACE(OwnerAddress,',','.'), 2),
	PARSENAME(REPLACE(OwnerAddress,',','.'), 1)
FROM PortfolioProject.dbo.NashvilleHousing
```

Now add the values to the table by creating new columns and adding the data

### For Owner Address

```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD OwnerSplitAddress NVARCHAR(255)

UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.'), 3)
```

### For Owner City

```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD OwnerSplitCity NVARCHAR(255)

UPDATE NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.'), 2)
```

### For Owner State

```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD OwnerSplitState NVARCHAR(255)

UPDATE NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.'), 1)
```

---

## 5) Change Y and N to Yes and No in "Sold as Vacant" field

There is a mix of Y, N, Yes and No in this column so we want to make it consistent by changing it to only Yes or No

```sql
SELECT DISTINCT SoldAsVacant, COUNT(SoldAsVacant)
FROM PortfolioProject.dbo.NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2
```

Change all the Y's to Yes and N's to No

```sql
SELECT SoldAsVacant,
	CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		 WHEN SoldAsVacant = 'N' THEN 'No'
		 ELSE SoldAsVacant
		 END
FROM PortfolioProject.dbo.NashvilleHousing
```

Update the original data

```sql
UPDATE NashvilleHousing
SET SoldAsVacant = 
	CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		 WHEN SoldAsVacant = 'N' THEN 'No'
		 ELSE SoldAsVacant -- this says otherwise leave it alone
		 END
	FROM PortfolioProject.dbo.NashvilleHousing
  ```
