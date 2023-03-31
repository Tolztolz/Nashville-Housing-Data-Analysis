# Nashville-Housing-Data-Analysis
Nashville Housing Data Analysis Using SQL

This project involved cleaning and querying a dataset containing Nashville housing data using SQL. The dataset was obtained from  https://www.kaggle.com/datasets/yohan313/nashville-housing-data, and contained 56477 rows of data with 19 columns of information on various housing units in Nashville.

The project began creating a new database and importing it using SQL. I started with data cleaning, where I identified and handled missing or incorrect data, removed duplicate entries, and standardized data formats to ensure consistency across the dataset.

Next, I explored the dataset by running SQL queries to extract useful information such as the most expensive and least expensive neighborhoods, the most common types of housing units, and the average price per square foot in different neighborhoods.

Throughout the project, I used various SQL commands such as SELECT, WHERE, GROUP BY, ORDER BY, and JOIN, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views and Converting Data Types to extract and manipulate the data as needed. The project helped me to improve my efficiency and skills with SQL.

Overall, this project demonstrates my ability to use SQL to clean and analyze large datasets, and provides valuable insights into the Nashville housing market. The code and visualizations are available on my Github repository, and can be used as a reference for future projects in data analysis and SQL.


```sql

-- cleaning data in sql

select *
from PortfolioProj..NashvilleHousing

-- standardize sale date

select SaleDateConverted
-- convert (Date, SaleDate)
from PortfolioProj..NashvilleHousing

update NashvilleHousing
set SaleDate = convert (Date, SaleDate)

Alter table NashvilleHousing
Add SaleDateConverted Date;

update NashvilleHousing
set SaleDateConverted = convert (Date, SaleDate)


---- Property Address data

select PropertyAddress
from PortfolioProj..NashvilleHousing
where PropertyAddress is null

select *
from PortfolioProj..NashvilleHousing
-- where PropertyAddress is null
order by ParcelID
 
-- Populate the property address
select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, isnull (a.PropertyAddress, b.PropertyAddress)
from PortfolioProj..NashvilleHousing a
Join PortfolioProj..NashvilleHousing b
	on a.ParcelID = b.ParcelID
	And a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null

update a
set PropertyAddress = isnull (a.PropertyAddress, b.PropertyAddress)
from PortfolioProj..NashvilleHousing a
Join PortfolioProj..NashvilleHousing b
	on a.ParcelID = b.ParcelID
	And a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null


---- breaking out address into individual columns (address, city, state)

select PropertyAddress
from PortfolioProj..NashvilleHousing
-- where PropertyAddress is null
--order by ParcelID

select
SUBSTRING (PropertyAddress, 1, CHARINDEX (',', PropertyAddress) -1) as Address,
SUBSTRING (PropertyAddress, CHARINDEX (',', PropertyAddress) + 1, LEN (PropertyAddress)) as Address

from PortfolioProj..NashvilleHousing

Alter table NashvilleHousing
Add PropertySplitAddress nvarchar(255);

update NashvilleHousing
set PropertySplitAddress = SUBSTRING (PropertyAddress, 1, CHARINDEX (',', PropertyAddress) -1)


Alter table NashvilleHousing
Add PropertySplitCity nvarchar(255);

update NashvilleHousing
set PropertySplitCity = SUBSTRING (PropertyAddress, CHARINDEX (',', PropertyAddress) + 1, LEN (PropertyAddress))

select *
from PortfolioProj..NashvilleHousing

----owner address

--We will use another method apart from substring

select OwnerAddress
from PortfolioProj..NashvilleHousing

select
PARSENAME (REPLACE (OwnerAddress, ',', '.'), 3),
PARSENAME (REPLACE (OwnerAddress, ',', '.'), 2),
PARSENAME (REPLACE (OwnerAddress, ',', '.'), 1)
from PortfolioProj..NashvilleHousing

Alter table NashvilleHousing
Add OwnerSplitAddress nvarchar(255);

update NashvilleHousing
set OwnerSplitAddress = PARSENAME (REPLACE (OwnerAddress, ',', '.'), 3)


Alter table NashvilleHousing
Add OwnerSplitCity nvarchar(255);

update NashvilleHousing
set OwnerSplitCity = PARSENAME (REPLACE (OwnerAddress, ',', '.'), 2)


Alter table NashvilleHousing
Add OwnerSplitState nvarchar(255);

update NashvilleHousing
set OwnerSplitState = PARSENAME (REPLACE (OwnerAddress, ',', '.'), 1)

select *
from PortfolioProj..NashvilleHousing



---- Change Y & N to Yes and No in Sold as Vacant field

select distinct (SoldAsVacant), Count(SoldAsVacant)
from PortfolioProj..NashvilleHousing
Group by SoldAsVacant
order by 2


select SoldAsVacant,
Case When SoldAsVacant = 'Y' then 'Yes'
	When SoldAsVacant = 'N' then 'No'
	else SoldAsVacant
end
from PortfolioProj..NashvilleHousing


update NashvilleHousing
Set SoldAsVacant = Case When SoldAsVacant = 'Y' then 'Yes'
	When SoldAsVacant = 'N' then 'No'
	else SoldAsVacant
end



---- Remove Duplicates

With RowNumCTE as (
Select*,
	ROW_NUMBER() Over (
	PARTITION BY ParcelID,
				PropertyAddress,
				SalePrice,
				SaleDate,
				LegalReference
				Order by
					UniqueID
					) row_num

from PortfolioProj..NashvilleHousing
--Order by ParcelID
)

Select *
--Delete
from RowNumCTE
where row_num > 1
--Order by PropertyAddress




Select *
from PortfolioProj..NashvilleHousing


---- Delete Unused Column

Select *
from PortfolioProj..NashvilleHousing



ALTER TABLE PortfolioProj.dbo.NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate





