#Introduction<br/> 
#==========
This library of functions was developed to meet the needs of the RMG Data Team working primarily with Quality and Absence data generated by the RMG projects in Bangladesh.

The aim of the library is to assist in the following stages of creating data sets from data provided as part of the RMG projects:

+ Consolidate data on excel worksheets in a number of excel files in the same directory into a single workbook
+ Renaming sheets in a consistent manner. 
+ Perform various checks on that workbook, including:
	+ Making various checks with regard to dates on each sheet
	+ Identifying the 'headers' on each sheet
	+ Identifying the end point of the data on each sheet
	+ Checking that 'headers' have the same meaning on each sheet

+ Creating an object that describes the data structure of each worksheet that can then be passed to a pandas program that will use the information to create a DataFrame that includes all relevant data on each sheet.

In an ideal world I would create a set of proper documentation for this library, perhaps one day this will happen. In the meantime this readme will follow the above aims in order that they appear such that the use of the library can be explained to those for whom the library is intended. Explaining the module in the order of the tasks makes sense as the workflow will generally always follow the same pattern. Any additional useful tips will be provided along the way. 

---

#Using this Module</br>
#==============
In order to use this module download the WorkbookFunctions.py file. You will need to add this file to a folder in the python path in order to be able to import it when using the DataNitro iPython shell. To determine what folders are in the python path open up a DataNitro iPython shell and do the following:

```python
import sys
sys.path
```

This will return a list of folders in the python path (i.e. the folders which python searches when `import` calls are made). I believe you can add the WorkbookFunctions.py file to any folder that appears in that path, although it is typical with Windows to add the file in the Python27/Lib/site-packages folder. 

I always import the module in the following manner:

```python
import WorkbookFunctions as wf
```

This is the convention that the remainder of this document follows. 

---

##Consolidate Sheets</br>
###Purpose and Information</br>
Many times the data that we work with are in folders that contain a number of Excel workbooks each of which will contain a sheet of data that we wish to read into a pandas DataFrame. However, before doing that we need to make a number of checks on the data and also to work out what structure the data are in before we can pass the data to a pandas program for conversion to a DataFrame. It is far simpler to make these checks and to define the structure of each relevant worksheet to be passed to a pandas program if all the worksheets are in the same workbook. 

The WorkbookFunctions module provides the `sheet_compiler` class object for compiling sheets from a number of workbooks into a single (new) workbook. Here are some facts about the sheet_compiler class:

+ The compile operations the class undertakes can only compile sheets from Excel workbooks in the same directory. Multiple directories cannot be used. 
+ The compile operations the class undertakes can only select and copy one sheet per workbook to the new workbook created.
+ The compile operations the class undertakes will only copy a sheet if the sheet is uniquely identified in the workbook according to the arguments provided by the programmer. More on this below


###Workflow and Syntax</br>
#### Creating a `sheet_compiler` Object <br/>
The syntax to create a sheet_compiler object is :

`wf.sheet_compiler(filepath)`

|     |     |
| --- | --- |
|filepath: | string of path to file directory to be worked with

Thus to initialise sheet_compiler object, pass the filepath that identifies the folder which contains the Excel workbooks from which you wish to copy worksheets to a new workbook:

```python
compiler = wf.sheet_compiler(r'path\to\folder')
```

Please note the use of the raw string literal (`r'`) when providing the path name. This is needed when paths contain backslashes (which they nearly always do) as the backslash is a special character in python.

####Getting a File List <br/>
Before compiling any sheets we need to know what files are contained in the folder we are going to work with. A list of the files in the directory can be obtained by calling the `get_file_list()` method. The syntax for this method is as follows: <br/>

`wf.sheet_compiler.get_file_list()`

|     |     |
| --- | --- |
|**Returns**: List of files in the filepath provided to the `sheet_compiler` Class object creator |

So an example call might look like:

```python
file_list = compiler.get_file_list()
file_list
```

> This file list should be checked by the user. 

If there are non excel files, these should be manually removed as this list will eventually be passed to the compiler. The compiler does not care in what order the list is, but the user might care. In the workbook of compiled sheets for example the user wants the sheets to appear in ascending order with regard to date of the worksheet. Typically the files we are working with a named according to date. However, the `get_file_list()` method uses the `os.listdir()` function which does not guarantee order. Therefore the order of the list should be manually manipulated until the user is satisfied that the workbooks will be approached in the order in which he would like to the see the worksheets in the compiled workbook. See the Troubleshooting.md file for some examples of how to deal with tricky situations regarding the file list.  

####Compiling the Sheets<br/>
Once the file_list is as the user wishes it to be it can be passed to the `compile_sheets()` method. The compile sheet method, actually does the work of compiling the sheets and returns a string with a report about the success of the compile. If some sheets could not be moved the user is notified by the returned string. 

The syntax for the `compile_sheets()` method is: </br>

`wf.sheet_compiler.compile_sheets(file_list, new_wkbk_name, sub_string1 [, sub_string2])`

|     |     |
| --- | --- |
| file_list: | A list of file names from which a worksheet will be copied |
| new_wkbk_name: | The name of the new workbook to which the individual worksheets will be compiled including the file extension (e.g. "compiled_workbook.xls") |
| sub_string1: | A string that uniquely identifies a sheet in the workbooks from which a sheet will be copied |
| sub_string2: | (Optional) A string that uniquely identifies a sheet in the workbooks from which a sheet will be copied |
| **Returns**: |String that indicates to user the success of the compile operation. |


As all the workbooks from which sheets will be copied tend to be of the same type, they will tend to have the same sheet names. The arguments sub_string1 and sub_string2 are passed to the method which upon opening the relevant workbook will search for the sheet to be moved by creating a list of sheet names in that workbook. The sheet names are strings. Therefore, the user should look at the sheet names used in the workbooks from which sheets will be copied and identify up to two sub_strings that will uniquely identify the sheet that is to be copied to the new workbook. 

By way of example suppose that each workbook contained three sheets:

1. "Daily Efficiency Summary"
2. "Daily Sewing Summary"
3. "Daily Sewing Quality"

If we were interested to obtain the second of those sheets we could pass `sub_string1 = "Sewing"`, `sub_string2 = "Summary"` as arguments to the `compile_sheets()` method. The full syntax might look look like this:

```python
compile_result = compiler.compile_sheets(file_list, 'compiled_workbook.xls', 'Sewing', 'Summary')
compile_result
```

The variable `compile_result` is a string that is a report that tells the user how successful the operation was for each file. 

> The user should examine the output and manually move sheets as necessary from any files where the compile operation was not successful. 

---

##Renaming Sheets<br/>
###Purpose and Information<br/>
When testing the structure of worksheets in a workbook, and their contents, it is useful if the sheets are named consistently, so that they can be easily identified by the user as needing attention. 

The WorkbookFunctions module provides the `rename_sheets()` function for this purpose. This function will rename up to 99 sheets acording to prefix + 2 digit serial. Practically speaking it is not recommended to work with files with more than 99 sheets, as the memory used can exceed that available and tasks become difficult to execute.


###Workflow and Syntax<br/>
The syntax for the function is as follows:

`wf.rename_sheets(prefix)`

|     |     |
| --- | --- |
| prefix : |string|
| **Returns** : |`None`|

An example function call might look like this:

```python
wf.rename_sheets('P')
```

---

##Dates<br/>
###Purpose and Information<br/>
Dates are incredibly important. We need to be able to accurate identify which date all data come from, and this is a job that is not frequently made easy by the people supplying the data. For one thing, the date may be part of some other complex string, dates may be accidentally repeated etc. For that reason the WorkbookFunctions module provides the `Dates` class object. Generally the cell that contains the date will be the same on each sheet of the workbook. The `Dates` class object will do a number of things for the user, to be seen below:


###Workflow and Syntax<br/>
####Creating a `wf.Dates` object<br/>
The syntax to create a Dates object is :

`wf.Dates(date_cell_ref, [strp_format, [separator, index_pos]])`

|     |     |
| --- | --- |
|date_cell_ref: |  tuple of two integers that reference the cell on which the date representation is located |
|strp_format: | (Optional) string representation of date format to be parsed by `datetime.datetime.strptime()` function |
|separator: | (Optional) separator to be used to split date representation string into a list using python `string.split()` method |
|index_pos: | (Optional) index position of relevant date string in the list that is a result of splitting the date representation using the separator. |

Some explanation is needed here. All dates on every worksheet will be converted into `datetime.date` objects in order for the other `wf.Dates` methods to be executable. Sometimes Excel dates can be read directly by DataNitro as `datetime.date` objects. In other cases they will be read as strings that need to be manipulated before they can be converted to `datetime.date` objects. 

In order to determine which universe you are in do a preliminary test. Say the cell with the date representation is represented by the tuple (2, 19) (row 2, column 19), then simply in the DataNitro iPython shell enter:

```python
Cell(2, 19).value
```

If the output of doing this looks like this:

```python
datetime.datetime(2013, 2, 4, 0, 0)
```

then DataNitro is reading the date directly as a datetime object. This is good. In this situation the only argument necessary to create the Dates class object is the tuple referencing the date. The syntax for creating a Dates class object in this situation is as follows:

```python
dates = wf.Dates((2, 19))
```

The next simplest case is when the output is simply a basic string representation of the date such as '3.12.2013'. In this case, the user should supply the `strp_format` argument which will enable the date string to be read as a datetime object. In this situation (and given the date string as just written), the synatax for creating a Dates class object is as follows:

```python
dates = wf.Dates((2, 19), "%d.%m.%Y")
```

> If you are unclear about how to use datetime.datetime.strptime() formats please see: https://docs.python.org/2/library/datetime.html#strftime-strptime-behavior

The least simplest case is when the output looks anything like this: "Production Report for Date: 2/11/13"

To convert this type of string to something that can be formatted using `datetime.strptime()` we need to be able to access the final part of the string i.e. the part that actually contains the string representation of the date without all the additional text. If we were to do this outside the context of WorkbookFunctions we might do the following:

```python
complex_string = "Production Report for Date: 2/11/13"
string_list = complex_string.split(':')
date_string = string_list[-1]
date_string

"2/11/13"
```

And indeed the arguments when creating the Dates object mirror that exact process. The string is split according to a particular separator (in the example ':'), and then a particular index point of the resulting list is accessed (in the example [-1]). Therefore, the syntax for creating a Dates object when faced with such string representations would be as follows:

```python
dates = wf.Dates((2, 19), %d/%m/%y", ':', -1)
```


####Getting a Single Date Value<br/>
To get a single date value from the active worksheet use the `wf.Dates.cell_to_date()` method. The syntax for this is simply:

`wf.Dates.cell_to_date()`

|     |     |
| --- | --- |
|**Returns**: | `datetime.date` object or `None` if conversion not possible given the arguments to the `Dates` class object creator |

An example call might look like:

```python
single_date = dates.cell_to_date()
```


####Getting All Dates in Workbook<br/>
To get a dictionary of all the dates found in the workbook (including sheets where date conversion was not possible given the arguments to the `Dates` Class object creator) use the `check_all_dates()` method. The syntax for calling this method is as follows:

`wf.Dates.check_all_dates()`

|     |     |
| --- | --- |
|**Returns**: | A dictionary that has keys that are the names of every sheet in the workbook, and values that are either the `datetime.date` objects found on those sheets, or a string indicating that the conversion to `datetime.date` object was not possible given the arguments provided to the `Dates` Class object creator. 

An example call might look as follows:

```python
dates_dict = dates.check_all_dates()
```

> The user should examine this dictionary to check that every sheet has a date that is convertible given the arguments supplied. 

The point of the WorkbookFunctions module is not to deal with every possible alternative, but rather to show the user where the assumed standard format is not applicable. If on certain sheets the standard format is not applicable, changes must be made, either manually, or by using general DataNitro data manipulation techniques in order to ensure that the format is the same on every sheet, and therefore the methods can run as intended on every sheet in the Workbook. Therefore, once the dictionary is created and errors identified, changes should be made to the worksheets themselves and the method re-called until such time as a `datetime.date` is available for every worksheet. The techniques used to ensure the formatting is standard will differ according to the workbook being worked with. See the Troubleshooing file for some tips. Every time a data technician encounters an issue this should be logged along with the solution such that the group can learn from techniques developed. 


####Getting All Date Cell Types in Workbook<br/>
If the date dictionary has many date values that are no found using the above, it can be useful to do a quick check of the types of value found at the date cell on each sheet. To do this use the `get_types()` method. The syntax for calling this method is as follows:

`wf.Dates.get_types()`

An example call might look as follows:

```python
dates__type_dict = dates.get_types()
```


####Checking for Duplicate Dates<br/>
Sometimes the persons providing this data forget to change the date on the worksheets, and this can lead to duplicate dates. In order to identify duplicate dates in the Workbook use the `duplicates()` method. The syntax for this method is as follows:

`wf.Dates.duplicates([date_dict])`

|     |     |
| --- | --- |
| date_dict: | (Optional) dictionary of keys that represent every sheet in the workbook and associated `datetime.date` objects |
| **Returns**: | Dict where dates that are found more than once in the values of date_dict are the keys, and the values are the keys of date_dict at which the duplicate dates are found. |

If the user does not supply a date_dict argument then the method will create one by calling the `wf.Dates.check_all_dates()` method.

The method will raise an exception if it is not the case that every value is a `dateimte.date` object in either the date_dict passed as argument or the date_dict created when no argument is passed to the method. 

Some data sets are such a mess, that it might be that the user has to create a date_dict himself, using techniques not provided by WorkbookFunctions. In these circumstances this *homemade* date_dict can be passed to the `duplicates()` method. 

An example call might be as follows:

```python
dates_dict = dates.check_all_dates()
duplicates_dict = dates.duplicates(dates_dict)
```

The above call passes the date_dict created by the `check_all_dates()` method, and thus saves the `duplicates()` method some leg work in having to re-create the date_dict. However, the following would be an equally valid alternative:

```python
duplicates_dict = dates.duplicates()
```

Alternatively if a *homemade* date_dict has been created not using any `WorkbookFunctions` methods then the call might look like this:

```python
duplicates_dict = dates.duplicates(homemade_date_dict)
```

> The user should examine the output and may have to make manual changes. 

Typically if a date is repeated, the 'true' date can be found by looking at the filename from which that data sheet was extracted. 


####Checking the Relative Order<br/>
The extent to which the sheets in the workbook need to be in the same order that the dates on the worksheets imply is not totally clear. On the one hand when we read the data from each sheet into and pandas DataFrame the data can simply be sorted according to date. On the other hand if the order of the sheets in the Workbook does not match the order implied by the dates found on the sheets this can be indicative of other problems with the data. Therefore, at a minimum the user should check that the order of the sheets matches the order implied by the dates on the sheets using the `relative_order()` method. The syntax for this method is as follows:

`wf.Dates.relative_order([date_dict])`

|     |     |
| --- | --- |
| date_dict: | (Optional) dictionary of keys that represent every sheet in the workbook and associated `datetime.date` objects |
|**Returns**: | Returns a dictionary that shows order of sheets implied by dates in the date_dict and the actual order of the sheets, if different. |

Again, if the user does not supply a date_dict argument then the method will create one by calling the `wf.Dates.check_all_dates()` method.

The method will raise an exception if it is not the case that every value is a `dateimte.date` object in either the date_dict passed as argument or the date_dict created when no argument is passed to the method. 

As above, the syntax allows the user to pass a 'homemade' date_dict, the date dict created previously, or None. So calls might look as follows:

```python
dates_dict = dates.check_all_dates()
relative_dict = dates.relative_order(dates_dict)
```

or

```python
relative_dict = dates.relative_order()
```

or

```python
relative_dict = dates.relative_order(homemade_date_dict)
```

> The output dictionary should at least be looked at by the user. 

If simple changes can be made to ensure the order, then they should be made, but any changes should be verified by looking at the files from which the data sheets are drawn. 

If changes to the sheet order are made it is useful to then use the `wf.rename_sheets()` function to rename the sheets according to the new order. 

####Checking the Date Discontinuities<br/>
In theory we should have six workbooks per week from which sheets have been extracted and compiled. Therefore if there are any discontinuities in the dates greater than one day, then it is possible that some files were missing from the folder from which sheets were compiled, or data might otherwise be missing. In order to check whether there are such discontinuities use the `discontinuities()` method. The syntax for this method is as follows:

`wf.Dates.discontinuities(discontinuity_value[,date_dict])`

|     |     |
| --- | --- |
| date_dict: | (Optional) dictionary of keys that represent every sheet in the workbook and associated `datetime.date` objects |
| **Returns**: | Returns list of tuples where each tuple is a pair of contiguous sheets where the dates found on those sheets indicate a discontinuity of more than the number of days specified as the discontinuity_value. |

Again, if the user does not supply a date_dict argument then the method will create one by calling the `wf.Dates.check_all_dates()` method.

The method will raise an exception if it is not the case that every value is a `dateimte.date` object in either the date_dict passed as argument or the date_dict created when no argument is passed to the method. 

As above, the syntax allows the user to pass a 'homemade' date_dict, the date dict created previously, or None. So calls might look as follows for checking for discontinuities of two days or more:

```python
dates_dict = dates.check_all_dates()
dicontinuity_dict = dates.discontinuities(2, dates_dict)
```

or

```python
dicontinuity_dict = dates.discontinuities(2)
```

or

```python
dicontinuity_dict = dates.discontinuities(2, homemade_date_dict)
```

> The resulting list should be examined, and any causes of missing data sheets discussed with the project management. 

Users should log any discontinuities found. If extra data are 'found' upon further investigation, the data from these sheets should be added to the folder directory and the processes described above should be re-enacted. 

---

##FindPoints<br/>
###Purpose and Information<br/>
Before passing the workbook to a pandas program that creates a DataFrame of all the data on all worksheets we need to be able to describe the structure of the worksheets in the following manner:

+ For every sheet, a row value on which the column 'Headers' are found.
+ For every sheet, a row value that indicates the last row of data that we are interested in. 

In locating these points (or rather finding the places where they are not locatable) the user is also performing useful checks on the worksheets, which if dealt with before the data are passed to pandas will reduce error and confusion for the persons using those DataFrames.

To assist in this process `WorkbookFunctions` has a `FindPoints` Class object available to the user. 

###Workflow and Syntax<br/>
####Creating a `wf.FindPoints` object<br/>
The synatax for creating a `FindPoints` Class object is as follows:

`wf.FindPoints(col, start_row, end_value, [,adjustments])`

|     |     |
| --- | --- |
| col: | integer value of the column to be searched |
| start_row: | integer value of the first row of the column to be searched |
| end_value: | string value then when found indicates that the 'point' has been found |
| adjustments: | (Optional) negative or positive integer value by which amount the final row value will be adjusted |

Some explanation is needed here:<br/>
 Suppose that the user wishes to identify the point (meaning row) in which the 'Column Headers' are found. The user should look at the structure of a worksheet and identify a value that when found should indicate that the point has been located. In general the 'headers' found in the worksheets will be the same or very similar. So for example, if in identifying the 'header columns' the first column header is 'Line' and this is found in column 2, then the user would specify the end_value as 'Line', the column value as 2, and then should specify which row the `FindPoints` class should begin to search for the end_value. Unless there are special reasons not to, this will generally be row 1. So the syntax for creating a `FindPoints` Class object in such circumstances would be as follows:<br/>

```python
headers = wf.FindPoints(2, 1, 'Line')
```

####Finding Points on a Single Active Worksheet<br/>
To find a point (as headers) on a single worksheet use the `find_point()` method. The syntax for this method is as follows:

`wf.FindPoints.find_point()`

|     |     |
| --- | --- |
| **Returns:** | Integer value of row in which the point is located |

> If the point is not found within the first 300 rows after the start_row, then an exception is raised. 

An example call might look like this:

```python
header_row = headers.find_point()
```

####Finding All Points in a Workbook<br/>
To find all points on every sheet in a workbook use the `find_all_points()` method. The syntax for this method is as follows:

`wf.FindPoints.find_all_points()`

|     |     |
| --- | --- |
| **Returns:** | Dict with one key for each sheet in workbook with point row as value. If no point row is found, the key maps to a string value notifying the user of the absence of the point row. |

An example call might look as follows:

```python
headers_dict = headers.find_all_points()
headers_dict
```

 > The user should examine the dict, and pay attention to any sheets where the point row was not located, determine the reason for this, and make any necessary changes to the data. 

The exact same process can be used to find the 'end points' of the data. Typically there will be a 'Total' row at the end of the data (that we are not interested in), but that is a useful place holder for identifying the end of the data we *are* interested in. Perhaps the word 'Total' is located in the fourth column. This total row might occur 2 rows after the end of the data we are interested in. Therefore, we would want to adjust the end_row values by -2. The process for doing this might look as follows:

```python
end_points = wf.FindPoints(4, 1, 'Total', -2)
end_point_dict = end_points.find_all_points()
end_point_dict
```

> With regard to finding end points it is better to be safe than sorry. 

If the user is not totally convinced that the end row needs to be adjusted they can simply not pass any adjustment argument. It is better to capture some unnecessary data when passing the data to pandas, than to lose important data. Alternatively the user could verify the need to make an adjustment by using techniques outside of `WorkbookFunctions`

---

##Columns<br/>
###Purpose and Information<br/>
When the data from a workbook is passed to a pandas program to create a DataFrame the user can specifiy which columns of data he wishes to retain. Not every column will be preserved as we are only interested in some of the data that are kept by entities providing data. The argument that is passed to the pandas Workbook parser are integer values of the columns to be retained. Therefore in order to be sure that the same data are taken from every worksheet the user must be satisfied that the same data are in the same columns on every worksheet. Effectively this means checking that the columns 'headers' are the same on every worksheet. In practice that the columns headers are the same and in the same order might in fact be of little practical significance as the entire worksheet could be parsed by pandas and then when the concatenation of the individual worksheet DataFrames happens, the DataFrame columns with the same header values will be automatically aligned. However, there will be cases where certain worksheets have different 'header' values due to typos, or the insertion of a new column, or the column header name is changed. This will cause the pandas program to create a DataFrame that is not hugely useful to the user. Therefore it is useful to check that the colum 'header' values are the same for every worksheet in the workbook beign worked with. To this end `WorkbookFunctions` has provided the `Columns` Class object. 

###Workflow and Syntax</br>
####Creating a `Columns` Object <br/>
The syntax to create a `Columns` class object is as follows:

`wf.Columns(column_values)`

|     |     |
| --- | --- |
|column_values: | list of integers representing the locations of the columns of interest. |

So if the user is interested ultimately in preserving the data from columns 1, 2, 3, 4, 6, 9, 10 and 11, then the syntax for creating the `Columns` object would be as follows:

```python
col_vals = wf.Columns([1, 2, 3, 4, 6, 9, 10, 11])
```

> An exception will be raised if it is not the case that all elements of the list passed are integers. 

####Getting Values for a Specific Row
To get the column values at a specific row on the active worksheet use the `get_values()` method. The syntax of this method is as follows:<br/>
`wf.Columns.get_values(row)`

|     |     |
| --- | --- |
| row: | An integer representing the row from which values will be taken |
| **Returns** | Returns list of lowered stripped string values found in each cell referenced by row and each column value as passed to the `Columns` object creator. |

An example call might look like:

```python
values = col_vals.get_values(4)
```

The above will return a list of the values found in row 4 at the columns referenced by the list passed to the `Columns` object creator. 


####Comparing all Column Values<br/>
In order to check whether all of the sheets have the same values at specific points then use the `compare_all_columns()` method. The syntax for this method is as follows:

`wf.Columns.compare_all_columns(start_row_dict)`

|     |     |
| --- | --- |
| start_row_dict: | dict with one key per sheet in the workbook with values that are integers representing the rows in which the column values to be compared are found. |
| **Returns**: | dict that has a key for each sheet in the workbook and values that are lists that contain those column values which were not located on any sheet when compared with the master sheet. |

Some explanation is needed here: <br/>
 The general strategy of the method is to look at each sheet, take an integer row value for the sheet from the start_row dict, and compare the values found at the locations referenced by that row value at the column values as passed to the `Columns` object creator to those values found on the first worksheet (the 'master sheet'). Therefore this is a general method. In fact of course it will work with any integers provided in the start_row_dict.

> However, the method was specifically designed to identify differences between sheets with regards to the column 'headers'!

Bearing this is mind therefore, the start_row_dict is an object that should have been created by using the `FindPoints` object methods to locate each row on each sheet where the column 'headers' are found. The method will set the values found on the first sheet as the master values and compare all other values on all other sheets against those values. 

So, an example call might look like this:

```python
headers = wf.FindPoints(2, 1, 'Line')
headers_dict = headers.find_all_points()
cols = wf.Columns([1, 2, 3, 4, 6, 9, 10, 11])
columns_discrepancy_dict = cols.compare_all_columns(headers_dict)
columns_discrepancy_dict
```

> The user should examine the output and determine the reasons for any discrepancies. 

The data should be manipulated such that all columns do represent the same data. It is envisaged that this will happen initially outside of the WorkbookFunctions library, but in the near future such functionality may be added to this library to automate the process. 

---

##Creating an Object to Describe the Workbook<br/>
Once the above checks have been made a dictionary can be created that will contain all the information needed to pass to a pandas program to create a single DataFrame from the entire workbook. An example of how this might look is as follows:

```python
start_rows = wf.FindPoints(1, 1, 'Line')
start_rows_dict = start_rows.find_all_points()

end_rows = wf.FindPoints(4, 1, 'Total', -2)
end_rows_dict = end_rows.find_all_points()

date = wf.Dates((2, 19), "%d.%m.%Y")
date_dict = date.check_all_dates()

columns = [1, 2, 3, 4, 6, 9, 10, 11]

Workbook_Structure = {'start_rows' : start_rows_dict,
					  'end_rows'   : end_rows_dict,
					  'dates' 	   : date_dict
					  'cols' 	   : columns 
					  }
```

The Workbook_Structure object can be saved to .json and then used when creating the pandas DataFrame. 









