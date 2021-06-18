# Cleaning Nashville Housing Data with SQL Queries

Not all data comes in a nice and neat format. Many times datasets include erroneous data from the source it was retrieved which must be addressed and cleansed.
This dataset includes information on Nashville housing data from January 2013 - October 2016. Here are some queries that will help cleanse the data to make it more usable for future projects.

---

## 1) Standardize Date Format

```sql
ALTER TABLE NashvilleHousing
ADD SaleDateConverted DATE;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(DATE, SaleDate)
```

---

## 2) Populate Property Address Data

```sql
SELECT *
FROM PortfolioProject.dbo.NashvilleHousing
WHERE PropertyAddress IS NULL
```

-- Populate a null PropertyAddress if there is a duplicate which contains the address
-- Must do a self-join that says if the ParcelID = ParcelID, then PropertyAddress(null) = PropertyAddress(not null)
-- Since the UniqueID is unique we can make sure that you are joining unique rows

-- Test ISNULL to see if it works correctly

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

-- Now UPDATE to correct the nulls

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

```sql
SELECT
	SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) AS Address,
	SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) AS Address
FROM PortfolioProject.dbo.NashvilleHousing
```

-- Alter the table to add these additional columns and then load in the data from the above formulas

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

```sql
SELECT
	PARSENAME(REPLACE(OwnerAddress,',','.'), 3),
	PARSENAME(REPLACE(OwnerAddress,',','.'), 2),
	PARSENAME(REPLACE(OwnerAddress,',','.'), 1)
FROM PortfolioProject.dbo.NashvilleHousing
```

-- Now add the values to the table by creating new columns and adding the data

-- For Owner Address

```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD OwnerSplitAddress NVARCHAR(255)

UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.'), 3)
```

-- For Owner City

```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD OwnerSplitCity NVARCHAR(255)

UPDATE NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.'), 2)
```

-- For Owner State

```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
ADD OwnerSplitState NVARCHAR(255)

UPDATE NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.'), 1)
```

---

## 5) Change Y and N to Yes and No in "Sold as Vacant" field

-- There is a mix of Y, N, Yes and No in this column so we want to make it consistent by changing it to only Yes or No

```sql
SELECT DISTINCT SoldAsVacant, COUNT(SoldAsVacant)
FROM PortfolioProject.dbo.NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2
```

-- Change all the Y's to Yes and N's to No

```sql
SELECT SoldAsVacant,
	CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		 WHEN SoldAsVacant = 'N' THEN 'No'
		 ELSE SoldAsVacant
		 END
FROM PortfolioProject.dbo.NashvilleHousing
```

-- Update the original data

```sql
UPDATE NashvilleHousing
SET SoldAsVacant = 
	CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		 WHEN SoldAsVacant = 'N' THEN 'No'
		 ELSE SoldAsVacant -- this says otherwise leave it alone
		 END
	FROM PortfolioProject.dbo.NashvilleHousing
  ```
