
# PyCitySchools Analysis

- Observed Trend 1: Students tend to have better reading scores than math scores in general. Also, the budget spending per student does not seem to have a big impact in the average scores, but instead the schools that have a greater budget spending per student have lower overall scores, as observed in the "Scores by School Spending' part.

- Observed Trend 2: The school size seems to be a factor that affects the average scores. The smaller schools tend to have a better performance than the large ones, as observed in the "School by Size Summary"  

- Observed Trend 3: The "Charter" schools have better average score rates than the "District" schools. The top 5 schools are "Charter" schools mean while the bottom 5 schools are "District" types.


```python
# Importing dependencies
import pandas as pd
```


```python
# Importing files
file_schools = "raw_data/schools_complete.csv"
file_students = "raw_data/students_complete.csv"
```


```python
# Reading the files
df_schools = pd.read_csv(file_schools)
#df_schools.head()
df_students = pd.read_csv(file_students)
#df_students.head()
```

# District Summary


```python
# Calculating the number of students
tot_students = df_students["Student ID"].count()

# Calculating the number of schools
tot_schools = df_schools["School ID"].count()

# Calculating the total budget
tot_budget = df_schools["budget"].sum()

# Calculating te average math score
avrg_math = round(df_students["math_score"].mean(),2)

# Calculating te average reading score
avrg_read = round(df_students["reading_score"].mean(),2)

# Calculating the % Passing Math
# Calculating the number of students that pass math (70 or above)
pass_math = df_students.loc[df_students["math_score"]>=70,:]
# Getting the Passing rate for math (students that passed math/total students) in %
tot_pass_math = round((pass_math["math_score"].count()/tot_students)*100,2)

# Calculating the % Passing Reading
# Calculating the number of students that pass reading (70 or above)
pass_read = df_students.loc[df_students["reading_score"]>=70,:]
# Getting the Passing rate for math (students that passed math/total students) in %
tot_pass_read = round((pass_read["reading_score"].count()/tot_students)*100,2)

# Calculating the % of passing overall (average of the above two) 
perc_pass_overall = round(((tot_pass_math + tot_pass_read)/2),2)

#Creating the Data Frame
district = [{
    "Total Schools": tot_schools, "Total Students": tot_students, "Total Budget": tot_budget, 
    "Average Math Score": avrg_math, "Average Reading Score": avrg_read, "% Passing Math": tot_pass_math, 
    "% Passing Reading": tot_pass_read, "% Overall Passing Rate": perc_pass_overall
}]
district_df = pd.DataFrame(district)
#district_df

# Changing the order of the columns in the DataFrame
columns = ["Total Schools", "Total Students", "Total Budget", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "% Overall Passing Rate"]
distric_order = district_df.loc[:,columns]

#Giving Format to the DataFrame
distric_order["Total Students"] = distric_order["Total Students"].map("{:,}".format)
#distric_order["Total Budget"] = distric_order["Total Budget"].map("${:.2f}".format)
distric_order["Total Budget"] = distric_order["Total Budget"].map("${:,.2f}".format)
distric_order
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428.00</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>74.98</td>
      <td>85.81</td>
      <td>80.4</td>
    </tr>
  </tbody>
</table>
</div>



# School Summary


```python
# renaming "school" column name, so we can merge it with the "students df"
rename_schools = df_schools.rename(columns={"name":"school"})

# Merging "schools df" and "students df"
merge_school = pd.merge(rename_schools,df_students, on="school", how="outer")

# Group the chart by school
grouped_school = merge_school.groupby(['school'])

# Getting the type of school by school
type_school = grouped_school["type"].max()
# Renaming the series
type_school = type_school.rename("School Type")

# Getting the number of students
num_students = grouped_school["Student ID"].count()
#If we wanted a data frame we would have used: num_students = grouped_school[["Student ID"]].count()
# Naming the calculated column
num_students = num_students.rename("Total Students")

# Calculating the budget per student for each school
per_student_bud = grouped_school["budget"].min()/num_students
# Naming the calculated column
per_student_bud = per_student_bud.rename("Per Student Budget")

# Calculating the average math score by school
avrg_math_school = round(grouped_school["math_score"].mean(),2)
# Renaming the column
avrg_math_school = avrg_math_school.rename("Average Math Score")

# Calculating the average reading score by school
# Double brackets make it a data frame: avrg_read_school = round(grouped_school[["reading_score"]].mean(),2)
avrg_read_school = round(grouped_school["reading_score"].mean(),2)
# Renaming the column
avrg_read_school = avrg_read_school.rename("Average Reading Score")

# Calculating the passing rate by school
# Counting the amount of students that got 70 or more in reading
pass_read = df_students.loc[df_students["reading_score"]>=70,:]
pass_read_group = pass_read.groupby(['school'])
# Dividing the amount of students that pass into the total amount of students per school
pass_read_by_school = round((pass_read_group["Student ID"].count()/num_students)*100,2)
# Renaming the series
pass_read_by_school = pass_read_by_school.rename("% Passing Readding")
#pass_read_by_school

# Calculating the passing rate by school
# Counting the amount of students that got 70 or more in Math
pass_math = df_students.loc[df_students["math_score"]>=70,:]
pass_math_group = pass_math.groupby(['school'])
# Dividing the amount of students that pass into the total amount of students per school
pass_math_by_school = round((pass_math_group["Student ID"].count()/num_students)*100,2)
# Renaming the series
pass_math_by_school = pass_math_by_school.rename("% Passing Math")
#pass_math_by_school

# Calculating the overall passing rate by school
overall_rate_sch = round((pass_read_by_school+pass_math_by_school)/2,2)
# Renaming the series
overall_rate_sch = overall_rate_sch.rename("% Overall Passing Rate")
overall_rate_sch

#Concatenating the previous indidual series:
merge_school_final_x = pd.concat([type_school, num_students, per_student_bud, avrg_math_school, avrg_read_school, pass_read_by_school, pass_math_by_school, overall_rate_sch], axis=1)
merge_school_final_x
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Readding</th>
      <th>% Passing Math</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school</th>
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
      <td>4976</td>
      <td>628.0</td>
      <td>77.05</td>
      <td>81.03</td>
      <td>81.93</td>
      <td>66.68</td>
      <td>74.31</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>582.0</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>97.04</td>
      <td>94.13</td>
      <td>95.58</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>639.0</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>80.74</td>
      <td>65.99</td>
      <td>73.36</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>644.0</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>79.30</td>
      <td>68.31</td>
      <td>73.81</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>625.0</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>97.14</td>
      <td>93.39</td>
      <td>95.26</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>652.0</td>
      <td>77.29</td>
      <td>80.93</td>
      <td>80.86</td>
      <td>66.75</td>
      <td>73.81</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>581.0</td>
      <td>83.80</td>
      <td>83.81</td>
      <td>96.25</td>
      <td>92.51</td>
      <td>94.38</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>655.0</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>81.32</td>
      <td>65.68</td>
      <td>73.50</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>650.0</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>81.22</td>
      <td>66.06</td>
      <td>73.64</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>609.0</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>95.95</td>
      <td>94.59</td>
      <td>95.27</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>637.0</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>80.22</td>
      <td>66.37</td>
      <td>73.30</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>600.0</td>
      <td>83.36</td>
      <td>83.73</td>
      <td>95.85</td>
      <td>93.87</td>
      <td>94.86</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>638.0</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>97.31</td>
      <td>93.27</td>
      <td>95.29</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>578.0</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>96.54</td>
      <td>93.87</td>
      <td>95.21</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>583.0</td>
      <td>83.68</td>
      <td>83.96</td>
      <td>96.61</td>
      <td>93.33</td>
      <td>94.97</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing Schools (By Passing Rate)


```python
#Sorting by top 5 performing schools based on Overall Passing Rate
#sorted values: sorted(overall_rate_sch, reverse=True)
top_schools = merge_school_final_x.sort_values(['% Overall Passing Rate'], ascending=False, inplace=False)
top_5 = top_schools.head(5)
top_5
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Readding</th>
      <th>% Passing Math</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school</th>
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
      <td>1858</td>
      <td>582.0</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>97.04</td>
      <td>94.13</td>
      <td>95.58</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>638.0</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>97.31</td>
      <td>93.27</td>
      <td>95.29</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>609.0</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>95.95</td>
      <td>94.59</td>
      <td>95.27</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>625.0</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>97.14</td>
      <td>93.39</td>
      <td>95.26</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>578.0</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>96.54</td>
      <td>93.87</td>
      <td>95.21</td>
    </tr>
  </tbody>
</table>
</div>



# Bottom Performing Schools (By Passing Rate)


```python
#Sorting by bottom 5 performing schools based on Overall Passing Rate
bot_schools = merge_school_final_x.sort_values(['% Overall Passing Rate'], ascending=True, inplace=False)
bottom_5 = bot_schools.head(5)
bottom_5
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Readding</th>
      <th>% Passing Math</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school</th>
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
      <td>3999</td>
      <td>637.0</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>80.22</td>
      <td>66.37</td>
      <td>73.30</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>639.0</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>80.74</td>
      <td>65.99</td>
      <td>73.36</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>655.0</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>81.32</td>
      <td>65.68</td>
      <td>73.50</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>650.0</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>81.22</td>
      <td>66.06</td>
      <td>73.64</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>644.0</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>79.30</td>
      <td>68.31</td>
      <td>73.81</td>
    </tr>
  </tbody>
</table>
</div>



# Math Scores by Grade


```python
# Grouping the merged table into schools and grades
grouped_school_grade = merge_school.groupby(['school', 'grade'])
# Calculating the mean of the math scores by school and grade
math_grade_1 = round(grouped_school_grade["math_score"].mean(),2)
#math_grade_1
# Creating a DataFrame with the above values
df2 = math_grade_1.to_frame()
#reseting the indexes
df2 = df2.reset_index(level=['school','grade'])
#df2
#Converting the table into a pivot table
df2.pivot(index='school', columns='grade', values='math_score')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.00</td>
      <td>77.52</td>
      <td>76.49</td>
      <td>77.08</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.15</td>
      <td>82.77</td>
      <td>83.28</td>
      <td>83.09</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.54</td>
      <td>76.88</td>
      <td>77.15</td>
      <td>76.40</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.67</td>
      <td>76.92</td>
      <td>76.18</td>
      <td>77.36</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>84.23</td>
      <td>83.84</td>
      <td>83.36</td>
      <td>82.04</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.34</td>
      <td>77.14</td>
      <td>77.19</td>
      <td>77.44</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.43</td>
      <td>85.00</td>
      <td>82.86</td>
      <td>83.79</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>75.91</td>
      <td>76.45</td>
      <td>77.23</td>
      <td>77.03</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>76.69</td>
      <td>77.49</td>
      <td>76.86</td>
      <td>77.19</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.37</td>
      <td>84.33</td>
      <td>84.12</td>
      <td>83.63</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.61</td>
      <td>76.40</td>
      <td>77.69</td>
      <td>76.86</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>82.92</td>
      <td>83.38</td>
      <td>83.78</td>
      <td>83.42</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.09</td>
      <td>83.50</td>
      <td>83.50</td>
      <td>83.59</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.72</td>
      <td>83.20</td>
      <td>83.04</td>
      <td>83.09</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>84.01</td>
      <td>83.84</td>
      <td>83.64</td>
      <td>83.26</td>
    </tr>
  </tbody>
</table>
</div>



# Reading Scores by Grade


```python
# Grouping the merged table into schools and grades
grouped_school_grade = merge_school.groupby(['school', 'grade'])
# Calculating the mean of the reading scores by school and grade
read_grade = round(grouped_school_grade["reading_score"].mean(),2)
read_grade
# Converting the GroupByDataFrame object to a DataFrame
df3 = read_grade.to_frame()
df3 = df3.reset_index(level=['school','grade'])
#df3
# Converting the DataFrame into a pivot table
df3.pivot(index='school', columns='grade', values='reading_score')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>80.91</td>
      <td>80.95</td>
      <td>80.91</td>
      <td>81.30</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>84.25</td>
      <td>83.79</td>
      <td>84.29</td>
      <td>83.68</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.41</td>
      <td>80.64</td>
      <td>81.38</td>
      <td>81.20</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>81.26</td>
      <td>80.40</td>
      <td>80.66</td>
      <td>80.63</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.71</td>
      <td>84.29</td>
      <td>84.01</td>
      <td>83.37</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.66</td>
      <td>81.40</td>
      <td>80.86</td>
      <td>80.87</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.32</td>
      <td>83.82</td>
      <td>84.70</td>
      <td>83.68</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.51</td>
      <td>81.42</td>
      <td>80.31</td>
      <td>81.29</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>80.77</td>
      <td>80.62</td>
      <td>81.23</td>
      <td>81.26</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.61</td>
      <td>84.34</td>
      <td>84.59</td>
      <td>83.81</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.63</td>
      <td>80.86</td>
      <td>80.38</td>
      <td>80.99</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.44</td>
      <td>84.37</td>
      <td>82.78</td>
      <td>84.12</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>84.25</td>
      <td>83.59</td>
      <td>83.83</td>
      <td>83.73</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>84.02</td>
      <td>83.76</td>
      <td>84.32</td>
      <td>83.94</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.81</td>
      <td>84.16</td>
      <td>84.07</td>
      <td>83.83</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending
- Create a table that breaks down school performances based on average Spending Ranges (Per Student). Use 4 reasonable bins to group school spending. Include in the table each of the following:


```python
# Calculating the greatest budget per student
a = top_schools['Per Student Budget'].max()
# Calculating the lowest budget per student
b = top_schools['Per Student Budget'].min()
# Creating the bins
spending_bins = [0, round((b+(a-b)/4),1), round((b+(a-b)/2),1), round((a-(a-b)/4),1),round(a+50,1)]
# Creating the group names
spending_groups = [
    ("< $"+str(round((b+(a-b)/4),1))), ("$"+str(round((b+(a-b)/4),1))+"-"+"$"+str(round((b+(a-b)/2),1))),
    ("$"+str(round((b+(a-b)/2),1))+"-$"+str(round((a-(a-b)/4),1))),("$"+str(round((a-(a-b)/4),1))+"-$"+str(round(a+50,1)))
]
spending_groups
```




    ['< $597.2', '$597.2-$616.5', '$616.5-$635.8', '$635.8-$705.0']




```python
# # Categories of perstudent budget per school
# test_1 = pd.cut(merge_school_final_x["Per Student Budget"], bins=spending_bins, labels=spending_groups)
# test_1
```


```python
#Place the per student budgets into bins
scores_by_spending = merge_school_final_x
scores_by_spending["Budget Summary"] = pd.cut(scores_by_spending["Per Student Budget"], bins=spending_bins, labels=spending_groups)
#scores_by_spending
# Reseting the index
df4 = scores_by_spending
df4 = df4.reset_index(level=['school'])
# Creating a group based off of the bins
df4_groups = df4.groupby("Budget Summary")
df4_groups["Average Math Score", "Average Reading Score", "% Passing Readding", "% Passing Math", "% Overall Passing Rate"].mean()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Readding</th>
      <th>% Passing Math</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Budget Summary</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt; $597.2</th>
      <td>83.452500</td>
      <td>83.935000</td>
      <td>96.610000</td>
      <td>93.460000</td>
      <td>95.035000</td>
    </tr>
    <tr>
      <th>$597.2-$616.5</th>
      <td>83.600000</td>
      <td>83.885000</td>
      <td>95.900000</td>
      <td>94.230000</td>
      <td>95.065000</td>
    </tr>
    <tr>
      <th>$616.5-$635.8</th>
      <td>80.200000</td>
      <td>82.425000</td>
      <td>89.535000</td>
      <td>80.035000</td>
      <td>84.785000</td>
    </tr>
    <tr>
      <th>$635.8-$705.0</th>
      <td>77.865714</td>
      <td>81.368571</td>
      <td>82.995714</td>
      <td>70.347143</td>
      <td>76.672857</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Size


```python
# Calculating the greatest quantity of students
a2 = top_schools['Total Students'].max()

# Calculating the lowest quantity of students
b2 = top_schools['Total Students'].min()

# Creating the bins
size_bins = [0, round((a2-b2)/3), round(2*(a2-b2)/3),round(a2+50)]

# Creating the group names
size_groups = ["small", "medium", "large"]
size_bins
```




    [0, 1516.0, 3033.0, 5026]




```python
#Place the size into bins
scores_by_size = merge_school_final_x
scores_by_size["School Size"] = pd.cut(
    scores_by_size["Total Students"], bins=size_bins, labels=size_groups)
#scores_by_size

# Reseting the index
df5 = scores_by_size
df5 = df5.reset_index(level=['school'])

# Creating a group based off of the bins
df5_groups = df5.groupby("School Size")
df5_groups[
    "Average Math Score", "Average Reading Score", "% Passing Readding", "% Passing Math", "% Overall Passing Rate"
].mean()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Readding</th>
      <th>% Passing Math</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>small</th>
      <td>83.663333</td>
      <td>83.8900</td>
      <td>96.446667</td>
      <td>93.496667</td>
      <td>94.9700</td>
    </tr>
    <tr>
      <th>medium</th>
      <td>80.903750</td>
      <td>82.8250</td>
      <td>90.588750</td>
      <td>83.556250</td>
      <td>87.0725</td>
    </tr>
    <tr>
      <th>large</th>
      <td>77.062500</td>
      <td>80.9175</td>
      <td>81.057500</td>
      <td>66.465000</td>
      <td>73.7650</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Type


```python
scores_by_type = merge_school_final_x
# Reseting the index
df6 = scores_by_type
df6 = df6.reset_index(level=['school'])
# Creating a group based off of the school type
df6_groups = df6.groupby("School Type")
df6_groups[
    "Average Math Score", "Average Reading Score", "% Passing Readding", "% Passing Math", "% Overall Passing Rate"
].mean()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Readding</th>
      <th>% Passing Math</th>
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
      <td>83.472500</td>
      <td>83.897500</td>
      <td>96.586250</td>
      <td>93.620000</td>
      <td>95.102500</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.955714</td>
      <td>80.965714</td>
      <td>80.798571</td>
      <td>66.548571</td>
      <td>73.675714</td>
    </tr>
  </tbody>
</table>
</div>


