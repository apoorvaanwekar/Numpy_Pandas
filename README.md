**PyCity Schools Analysis**

* Trend 1: Students are doing better in Reading as compared to Math as the passing percentage in Reading is better as compared to the math passing percentage
* Trend 2: Budget per student is not a matter of concern in either type of schools, Per student budget in either Charter or District school is almost similar.
* Trend 3: All the schools have a consistent level of performance among each grade. 

```python
# Import Dependencies
import pandas as pd
import numpy as np

# Create a reference the CSV file desired
schools_csv_path = "raw_data/schools_complete.csv"
students_csv_path = "raw_data/students_complete.csv"

# Read the first CSV into a Pandas DataFrame
schools_df = pd.read_csv(schools_csv_path)

#Renaming the columns as per the desired solutions
schools_df = schools_df.rename(columns={"name":"School Name", "type":"School Type", "budget": "Total Budget"})

# Read the second CSV into a Pandas DataFrame
students_df = pd.read_csv(students_csv_path)

#Renaming the columns as per the desired solutions
students_df = students_df.rename(columns={"name":"Student Name", "school":"School Name"})

#Merging the two table on "School Name" :Righ Join to get all the rows from right table and only corresponding roles from left table
schools_students_df = pd.merge(schools_df,students_df,on="School Name",how="right")
```

**District Summary**
```python
#Extracting only District Schools from the data frame
district_schools_df=schools_students_df.loc[schools_students_df['School Type'] == "District"]

#Compute the fileds
Total_Schools = district_schools_df['School ID'].nunique()
Total_Students=district_schools_df["Student ID"].nunique()
Total_budget=schools_df["Total Budget"].sum()
Per_Student_Budget=district_schools_df["Total Budget"].sum()/district_schools_df["Student ID"].nunique()
Avg_math_score=district_schools_df["math_score"].mean()
Avg_reading_score=district_schools_df["reading_score"].mean()
Math_expert = district_schools_df['math_score'] > 70
true_math=Math_expert.sum()
Percent_Passing_Math=(true_math/Total_Students)*100

reading_expert = district_schools_df['reading_score'] > 70
true_reader=reading_expert.sum()
Percent_Passing_reading=(true_reader/Total_Students)*100
Percent_Passing_reading
Overall_passing=(Percent_Passing_Math/Percent_Passing_reading)*100
Overall_passing


#Create dataframe from the fields
summary_df = pd.DataFrame({ "Average Math Score": [Avg_math_score],
"Average Reading Score":[Avg_reading_score],
"% Passing Math": [Percent_Passing_Math],
"% Passing Reading": [Percent_Passing_reading],
"% Overall Passing Rate":[Overall_passing],
"Total Budget": [Total_budget],
"Total Students": [Total_Students],
"Total Schools":[Total_Schools] 
})

#Reorder columns
summary_df = summary_df.reindex(columns=['Total Schools','Total Students', 'Total Budget', 'Average Math Score', 'Average Reading Score','% Passing Math','% Passing reading','% Overall Passing Rate'])

summary_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7</td>
      <td>26976</td>
      <td>24649428</td>
      <td>76.987026</td>
      <td>80.962485</td>
      <td>64.305308</td>
      <td>NaN</td>
      <td>82.053829</td>
    </tr>
  </tbody>
</table>
</div>



**School Summary**
```python
#applying a condition where math score is greater then 70 (passing score=70) 
#creating a new column to track students who passed in math by assinging values (1 to pass :0 to fail)
schools_students_df['passing_math'] = np.where(schools_students_df['math_score'] >= 70, 1, 0)

#applying a condition where reading score is greater then 70 (passing score=70) 
#creating a new column to track students who passed in reading by assinging values (1 to pass :0 to fail)
schools_students_df['passing_reading'] = np.where(schools_students_df['reading_score'] >= 70, 1, 0)

#using group by method, grouping the data frame on School name and aggregating the data using "agg" function
school_summary_df = schools_students_df.groupby(["School Name"]).agg({'School Type':pd.Series.max,'School ID': pd.Series.max, 'Total Budget': pd.Series.max,'Student ID': pd.Series.nunique, 'reading_score': pd.Series.mean, 'math_score': pd.Series.mean, 'passing_math': pd.Series.sum, 'passing_reading': pd.Series.sum})
school_summary_df = school_summary_df.rename(columns={"Student ID":"Total Students", "math_score":"Average_math_score","reading_score":"Average_reading_score"})
school_summary_df['% Passing Math'] = (school_summary_df['passing_math']/school_summary_df['Total Students'])*100
school_summary_df['% Passing Reading'] = (school_summary_df['passing_reading']/school_summary_df['Total Students'])*100
school_summary_df['Overall Passing Rate'] = (school_summary_df['% Passing Math'] + school_summary_df['% Passing Reading'])/2
school_summary_df['Per Student Budget'] = school_summary_df["Total Budget"]/school_summary_df['Total Students']
```


```python
school_summary_result = school_summary_df.drop(['passing_math', 'passing_reading', 'School ID'], axis=1)
school_summary_result['Total Budget'] = school_summary_result['Total Budget'].map("${:,.0f}".format)
school_summary_result['Per Student Budget'] = school_summary_result['Per Student Budget'].map("${:,.0f}".format)
school_summary_result
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Budget</th>
      <th>Total Students</th>
      <th>Average_reading_score</th>
      <th>Average_math_score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
      <th>Per Student Budget</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>$3,124,928</td>
      <td>4976</td>
      <td>81.033963</td>
      <td>77.048432</td>
      <td>66.680064</td>
      <td>81.933280</td>
      <td>74.306672</td>
      <td>$628</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>$1,081,356</td>
      <td>1858</td>
      <td>83.975780</td>
      <td>83.061895</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>95.586652</td>
      <td>$582</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>$1,884,411</td>
      <td>2949</td>
      <td>81.158020</td>
      <td>76.711767</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>73.363852</td>
      <td>$639</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>$1,763,916</td>
      <td>2739</td>
      <td>80.746258</td>
      <td>77.102592</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>73.804308</td>
      <td>$644</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>$917,500</td>
      <td>1468</td>
      <td>83.816757</td>
      <td>83.351499</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>95.265668</td>
      <td>$625</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>$3,022,020</td>
      <td>4635</td>
      <td>80.934412</td>
      <td>77.289752</td>
      <td>66.752967</td>
      <td>80.862999</td>
      <td>73.807983</td>
      <td>$652</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>$248,087</td>
      <td>427</td>
      <td>83.814988</td>
      <td>83.803279</td>
      <td>92.505855</td>
      <td>96.252927</td>
      <td>94.379391</td>
      <td>$581</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>$1,910,635</td>
      <td>2917</td>
      <td>81.182722</td>
      <td>76.629414</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>73.500171</td>
      <td>$655</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>$3,094,650</td>
      <td>4761</td>
      <td>80.966394</td>
      <td>77.072464</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>73.639992</td>
      <td>$650</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>$585,858</td>
      <td>962</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>95.270270</td>
      <td>$609</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>$2,547,363</td>
      <td>3999</td>
      <td>80.744686</td>
      <td>76.842711</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>73.293323</td>
      <td>$637</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>$1,056,600</td>
      <td>1761</td>
      <td>83.725724</td>
      <td>83.359455</td>
      <td>93.867121</td>
      <td>95.854628</td>
      <td>94.860875</td>
      <td>$600</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>$1,043,130</td>
      <td>1635</td>
      <td>83.848930</td>
      <td>83.418349</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>95.290520</td>
      <td>$638</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>$1,319,574</td>
      <td>2283</td>
      <td>83.989488</td>
      <td>83.274201</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>95.203679</td>
      <td>$578</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>$1,049,400</td>
      <td>1800</td>
      <td>83.955000</td>
      <td>83.682222</td>
      <td>93.333333</td>
      <td>96.611111</td>
      <td>94.972222</td>
      <td>$583</td>
    </tr>
  </tbody>
</table>
</div>


**Top Performing Schools (By Passing Rate)**

```python
#### Top Performing Schools (By Passing Rate)###
top=school_summary_result.nlargest(5,'Overall Passing Rate')
top
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Budget</th>
      <th>Total Students</th>
      <th>Average_reading_score</th>
      <th>Average_math_score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
      <th>Per Student Budget</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>$1,081,356</td>
      <td>1858</td>
      <td>83.975780</td>
      <td>83.061895</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>95.586652</td>
      <td>$582</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>$1,043,130</td>
      <td>1635</td>
      <td>83.848930</td>
      <td>83.418349</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>95.290520</td>
      <td>$638</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>$585,858</td>
      <td>962</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>95.270270</td>
      <td>$609</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>$917,500</td>
      <td>1468</td>
      <td>83.816757</td>
      <td>83.351499</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>95.265668</td>
      <td>$625</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>$1,319,574</td>
      <td>2283</td>
      <td>83.989488</td>
      <td>83.274201</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>95.203679</td>
      <td>$578</td>
    </tr>
  </tbody>
</table>
</div>


**Bottom Performing Schools (By Passing Rate)**

```python
#### Bottom Performing Schools (By Passing Rate)###
bottom=school_summary_result.nsmallest(5,'Overall Passing Rate')
bottom
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Budget</th>
      <th>Total Students</th>
      <th>Average_reading_score</th>
      <th>Average_math_score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
      <th>Per Student Budget</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>$2,547,363</td>
      <td>3999</td>
      <td>80.744686</td>
      <td>76.842711</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>73.293323</td>
      <td>$637</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>$1,884,411</td>
      <td>2949</td>
      <td>81.158020</td>
      <td>76.711767</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>73.363852</td>
      <td>$639</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>$1,910,635</td>
      <td>2917</td>
      <td>81.182722</td>
      <td>76.629414</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>73.500171</td>
      <td>$655</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>$3,094,650</td>
      <td>4761</td>
      <td>80.966394</td>
      <td>77.072464</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>73.639992</td>
      <td>$650</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>$1,763,916</td>
      <td>2739</td>
      <td>80.746258</td>
      <td>77.102592</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>73.804308</td>
      <td>$644</td>
    </tr>
  </tbody>
</table>
</div>


**Math Scores by Grade**

```python
###### Math Scores by Grade #######
#grouping the data on School Name and grade and using agg function to compute the average math score
name_df=students_df.groupby(["School Name","grade"]).agg({'math_score': pd.Series.mean})
#Using unstack function to pivot the table
name_df.unstack()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="4" halign="left">math_score</th>
    </tr>
    <tr>
      <th>grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
      <td>77.083676</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
      <td>83.094697</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
      <td>76.403037</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
      <td>77.361345</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
      <td>82.044010</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
      <td>77.438495</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
      <td>83.787402</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
      <td>77.027251</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
      <td>77.187857</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
      <td>83.625455</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
      <td>76.859966</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
      <td>83.420755</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
      <td>83.590022</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
      <td>83.085578</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
      <td>83.264706</td>
    </tr>
  </tbody>
</table>
</div>


**Reading Score by Grade**

```python
#####Reading Score by Grade#####

#grouping the data on School Name and grade and using agg function to compute the average reading score
name_reading=students_df.groupby(["School Name","grade"]).agg({'reading_score': pd.Series.mean})

#Using unstack function to pivot the table
name_reading.unstack()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="4" halign="left">reading_score</th>
    </tr>
    <tr>
      <th>grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
      <td>81.303155</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
      <td>83.676136</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
      <td>81.198598</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
      <td>80.632653</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
      <td>83.369193</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
      <td>80.866860</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
      <td>83.677165</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
      <td>81.290284</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
      <td>81.260714</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
      <td>83.807273</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
      <td>80.993127</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
      <td>84.122642</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
      <td>83.728850</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
      <td>83.939778</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
      <td>83.833333</td>
    </tr>
  </tbody>
</table>
</div>


**Scores by School Spending**

```python
#######################  Scores by School Spending   #########################
#Creating  the bins in which Data will be held
bins = [0, 585, 615, 645,675]

# Create the names for the four bins
group_names = ['<$585', '$585-615', '$615-645','$645-675']
# Join School_Students dataframe with school_summary dataframe to get per student budget
school_spending_summary_df = pd.merge(schools_students_df,school_summary_df, on="School ID",how="right")
school_spending_summary_df.drop(['Total Students','Average_reading_score','Average_math_score','% Passing Math','% Passing Reading','Overall Passing Rate','passing_math_y','passing_reading_y','Total Budget_y'], axis=1, inplace=True)


# Cut per student Budget and place the scores into bins
school_spending_summary_df["Per Student Summary"] = pd.cut(school_spending_summary_df["Per Student Budget"], bins, labels=group_names)
school_spending_summary_df = school_spending_summary_df.rename(columns={'passing_math_x':'passing_math', 'passing_reading_x':'passing_reading', 'Total Budget_x':'Total Budget'})



studentbudget_summary_df = school_spending_summary_df.groupby(['Per Student Summary']).agg({ 'Student ID': pd.Series.nunique, 'reading_score': pd.Series.mean, 'math_score': pd.Series.mean, 'passing_math': pd.Series.sum, 'passing_reading': pd.Series.sum})
studentbudget_summary_df = studentbudget_summary_df.rename(columns={ "Student ID": "Total Students","math_score":"Average_math_score","reading_score":"Average_reading_score"})
studentbudget_summary_df['% Passing Math'] = (studentbudget_summary_df['passing_math']/studentbudget_summary_df['Total Students'])*100
studentbudget_summary_df['% Passing Reading'] = (studentbudget_summary_df['passing_reading']/studentbudget_summary_df['Total Students'])*100
studentbudget_summary_df['Overall Passing Rate'] = (studentbudget_summary_df['% Passing Math'] + studentbudget_summary_df['% Passing Reading'])/2
studentbudget_summary_df.drop(['passing_math', 'passing_reading', 'Total Students'], axis=1, inplace=True)
studentbudget_summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average_reading_score</th>
      <th>Average_math_score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Per Student Summary</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.964039</td>
      <td>83.363065</td>
      <td>93.702889</td>
      <td>96.686558</td>
      <td>95.194724</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.838414</td>
      <td>83.529196</td>
      <td>94.124128</td>
      <td>95.886889</td>
      <td>95.005509</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>81.434088</td>
      <td>78.061635</td>
      <td>71.400428</td>
      <td>83.614770</td>
      <td>77.507599</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>81.005604</td>
      <td>77.049297</td>
      <td>66.230813</td>
      <td>81.109397</td>
      <td>73.670105</td>
    </tr>
  </tbody>
</table>
</div>


**Scores by School Size**

```python
#######################  Scores by School Size   #########################
# Creating  the bins in which Data will be held
bins = [0, 1000, 2000, 5000]

# Create the names for the four bins
group_names = ['Small (<1000)', 'Medium (1000-2000)', 'Large (2000-5000)']

# Cut School size and place the scores into bins
schools_students_df["School Size Summary"] = pd.cut(schools_students_df["size"], bins, labels=group_names)

## Creating a group based off of the bins
school_size_summary_df = schools_students_df.groupby(['School Size Summary']).agg({ 'Student ID': pd.Series.nunique, 'reading_score': pd.Series.mean, 'math_score': pd.Series.mean, 'passing_math': pd.Series.sum, 'passing_reading': pd.Series.sum})
school_size_summary_df = school_size_summary_df.rename(columns={ "Student ID": "Total Students","math_score":"Average_math_score","reading_score":"Average_reading_score"})
school_size_summary_df['% Passing Math'] = (school_size_summary_df['passing_math']/school_size_summary_df['Total Students'])*100
school_size_summary_df['% Passing Reading'] = (school_size_summary_df['passing_reading']/school_size_summary_df['Total Students'])*100
school_size_summary_df['Overall Passing Rate'] = (school_size_summary_df['% Passing Math'] + school_size_summary_df['% Passing Reading'])/2
school_size_summary_df.drop(['passing_math', 'passing_reading'], axis=1, inplace=True)
#school_size_summary_df['Overall Passing Rate'] = (school_size_summary_df['% Passing Math'] + school_size_summary_df['% Passing Reading'])/2
school_size_summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Students</th>
      <th>Average_reading_score</th>
      <th>Average_math_score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size Summary</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1000)</th>
      <td>1389</td>
      <td>83.974082</td>
      <td>83.828654</td>
      <td>93.952484</td>
      <td>96.040317</td>
      <td>94.996400</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>8522</td>
      <td>83.867989</td>
      <td>83.372682</td>
      <td>93.616522</td>
      <td>96.773058</td>
      <td>95.194790</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>29259</td>
      <td>81.198674</td>
      <td>77.477597</td>
      <td>68.652380</td>
      <td>82.125158</td>
      <td>75.388769</td>
    </tr>
  </tbody>
</table>
</div>


**Scores by School Type**

```python
#####  Scores by School Type ######


school_type_group= schools_students_df.groupby(["School Type"]).agg({'School ID': pd.Series.max, 'Total Budget': pd.Series.max,'Student ID': pd.Series.nunique, 'reading_score': pd.Series.mean, 'math_score': pd.Series.mean, 'passing_math': pd.Series.sum, 'passing_reading': pd.Series.sum})

```


```python
school_type_group_df = school_type_group.rename(columns={ "Student ID": "Total Students","math_score":"Average_math_score","reading_score":"Average_reading_score"})
school_type_group_df['% Passing Math'] = (school_type_group_df['passing_math']/school_type_group_df['Total Students'])*100
school_type_group_df['% Passing Reading'] = (school_type_group_df['passing_reading']/school_type_group_df['Total Students'])*100
school_type_group_df['% Overall Passing Rate'] = (school_type_group_df['% Passing Math'] + school_type_group_df['% Passing Reading'])/2
school_type_group_df.drop(['passing_math','Total Budget','School ID', 'passing_reading', 'Total Students'], axis=1, inplace=True)
school_type_group_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average_reading_score</th>
      <th>Average_math_score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.902821</td>
      <td>83.406183</td>
      <td>93.701821</td>
      <td>96.645891</td>
      <td>95.173856</td>
    </tr>
    <tr>
      <th>District</th>
      <td>80.962485</td>
      <td>76.987026</td>
      <td>66.518387</td>
      <td>80.905249</td>
      <td>73.711818</td>
    </tr>
  </tbody>
</table>
</div>


