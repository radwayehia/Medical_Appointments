# Project: Medical_appointments-may-2016

#  Table of Contents
<ul>
<li><a href="#intro">Introduction</a></li>
<li><a href="#wrangling">Data Wrangling</a></li>
<li><a href="#eda">Exploratory Data Analysis</a></li>
<li><a href="#conclusions">Conclusions</a></li>
</ul>
<a id='intro'></a>
## Introduction

### Dataset Description 

We'll be analyzing a collected inforamtion of 100k medical appointments in Brazil and focusing on the question of whether or not patients show up for their appointment and what is the relation between them showing up and other data provided.                                                       
The columns included in this data table are:
> 1- PatientId: set a special Id for each patient to have the history of each patient related if later needed.
> 2- AppointmentID: unique number for each appointment booked.                                
> 3- Gender:males(m), females(F).                         
> 4- Scheduled Day: the date assigned for the appoinment.               
> 5- Appointment Day: the date the patient showed up in.                     
> 6- Age: the age of the patient.                                                              
> 7- Neighbourhood: indicates the location of the hospital.                     
> 8- Scholarship: whether or not the patient is enrolled in  Brasilian welfare program.                    
> 9- Hipertension: whether the patient has hypertention(1) or not(0).                       
> 10- Diabetes: whether the patient has Diabetes(1) or not(0).                     
> 11- Alcoholism: whether the patient is alcoholic(1) or not(0).                      
> 12- Handcap: whether the patient is handicapped(1) or not(0).                       
> 13- SMS_received: the patient recieving a message regarding the appoinment(1) or not(0).                   
> 14- No-show: whether the patient showed(No) or didn't show(Yes).

### Question(s) for Analysis
Q1: Does having a scholarship affects showing in appointment?                          
Q2: How age is associated with showing in appointment?                            
Q3: Does recieving SMS decrease showing in appointment?                       
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
# Upgrade pandas to use dataframe.explode() function. 
%pip install --upgrade pandas==0.25.0
<a id='wrangling'></a>
## Data Wrangling

noshow = pd.read_csv("noshowappointments-kagglev2-may-2016.csv")
noshow.head()
noshow.info()
noshow.shape
noshow.describe()
noshow.duplicated().sum() #checking duplicates
noshow.info() #checking dtypes, NAN values
### Data Cleaning
### Correcting data types
> first: changing type of patient id from a float to an integer
noshow["PatientId"] = noshow["PatientId"].astype(int)
noshow.info() #check the changes

> changing ScheduledDay , AppointmentDay from an object to a datetime
r = ['ScheduledDay', 'AppointmentDay']
for i in r :
    noshow[i] = pd.to_datetime(noshow[i])        
noshow.info() #check the changes
# Changing Columns names
# replace spaces with underscores and lowercase labels for the dataset
noshow.rename(columns = lambda x: x.strip().lower().replace('-','_'), inplace = True)
noshow.head()  #check the columns names changes
#changing names to be compatible with other columns 
old_names = ["patientid", "appointmentid", "appointmentday", "scheduledday"]
new_names = ["patient_id", "appointment_id", "appointment_day", "scheduled_day"]
for i in range(4) :
    noshow.rename(columns = {old_names[i]:new_names[i]},inplace = True)
noshow.head() #checking result
# Dropping illogical data
noshow.query("age < 0") #Selecting ages less than 0
noshow.drop(99832, axis = 0, inplace = True) #Dropping these data
r = noshow.query("handcap > 1").index.tolist() #Selecting values more than 1
noshow.drop(r, axis = 0, inplace = True) #Dropping these values
# turning no_show column to 0 for no , 1 for yes
noshow['no_show'] = noshow['no_show'].apply(lambda x: 0 if x=='No' else 1)
noshow.head()
<a id='eda'></a>
## Exploratory Data Analysis

> scatter plot showing relations between columns of the dataset
#set plot dimension
plt.figure(figsize = [16,4])

sns.heatmap(noshow.corr(), annot = True, fmt = '.2f',
cmap = 'RdYlGn',
vmin = -1, center = 0, vmax= 1);

# How scholarships affects showing in appointments?
COMPARING SAMPLES OF SCHOLARSHIPS WITH SHOWING IN APPOINTMENTS
noshow.groupby('scholarship')['no_show'].value_counts().unstack('no_show')
It shows that counts of people who received scholarships are almost 1/10 of the one who didn't so the sample isn't compatible
Visualization of scholarships and Appointments showing percentages
# defining plot percentage function
def mypercentplot(df,xvar, normalize = True, color = ['red', 'blue']):
    ''' function to plot bar visualization of different values against no_show column
    Inputs: df dataset
    xvar column name
    normalize percentage transformation defaulted true
    colors red, blue defaults
    output: barplotted with percentage of chosen values against no_show  '''
    
    #If count plot multiply by 1, otherwise multiply be 100
    mul = 1
    if normalize:
        mul = 100
    #plot
    df.groupby([xvar])['no_show'].value_counts(normalize = normalize).unstack('no_show').mul(mul).plot.bar(edgecolor = "black", figsize = [14,6], rot = 0, width = .8, color = color);
    
    #Clean_up after plotting
    xvar = xvar.replace('_', " ") #replace _ with space
    
    #add_title and format it
    plt.title(f'percentage show/no by {xvar}'.title(), fontsize = 14, weight = 'bold')
    #add_xlabel and format it
    plt.xlabel(xvar.title(), fontsize = 10, weight = 'bold')
    #add_ylabel and format it
    plt.ylabel('percentage'.title(), fontsize = 10, weight = 'bold')
#call plotting function
mypercentplot(noshow, 'scholarship')
   people that missed their appointments and didn't have scholarship are the least among all
# Q2:  How age is associated with showing in appointment?
> Fig (1): A histogram showing relation between age and no_show columns
noshow.groupby('age')['no_show'].value_counts(normalize= True).unstack('no_show').mul(100).plot(figsize = [14,6])
 ages between 45 and 115 are showing the least numbers in missing their appointments which needs more investigations
> specifying age ranges in dataset
noshow["age"].describe()
> dividing ages into different stages
#Intervals for age and lables
Age_Intervals = [0,18,45, noshow["age"].max()]
Age_labels = ['childs', 'youngs', 'elders']
#Create a new Age_groups variables 
noshow['age_groups'] = pd.cut(noshow['age'], Age_Intervals, labels = Age_labels, include_lowest = True)

noshow['age_groups'].value_counts() #COMPARING THE SAMPLES
> statistics in different stages
noshow.groupby('age_groups')['no_show'].value_counts(normalize= True).unstack('no_show').reindex(Age_labels).mul(100)
>visualization of numbers of both attendants and absents of the same sample_numbers versus ages
#call plotting function
mypercentplot(noshow, 'age_groups')
percentages of elders group are most committed to their appointments
# Q3: Does recieving SMS decrease showing in appointment?

>Inspecting SMS relation with showing

mypercentplot(noshow,'sms_received')
people that received sms are missing appointments more frequently
COMPARING SAMPLES OF SMS_RECEIVED WITH SHOWING IN APPOINTMENTS
noshow.groupby('sms_received')['no_show'].value_counts().unstack('no_show')
It shows that counts of people who received sms are almost half of the one who didn't so the sample isn't compatible
<a id='conclusions'></a>
## Conclusions


I stated the analysis with showing some statistics about the data set and determined the data needs to be fixed.
In the cleaning stage I first changed the data types of columns then changed names of columns to be compatible with each other for easy remembering.
Then I dropped illogical data like negative ages and also turned the yes and no in no_show column into zeros and ones.      

> 1- In the first question I compared those with scholarships attending and missing their appointments to people without scholarships to see if having scholarship affected showing in appointments.                                  
  
It showed that people with no_scholarships showed in their appointments more than those with scholarships and also got a less numbers in missing their appointments.             
                                                      
limitation : The analysis could give better results if information about the payment timing process and the cost of appointments that were missed are provided and analyzed the aspect of prepaid appointments.
Also the samples weren't compatible so more data should be provided for more accurate results.

> 2- The second question was exploring the ages that are more willing to commit to the schedule. It showed that people that are classified as elders are the least people missing their appointments.                                                       
              
limitation: The groups were different in numbers of people, people with higher ages sample was the minimum, though the analysis was done with a ratio it will reduce the margin error to unit the samples of each stage and also analyze the medical status in these groups as it may affect the relation expected between age and showing in appointments.


> 3- The third question was seeing if sending an SMS could reduce missing the appointments. It showed that people who didn't receive SMS were less absent than those who didn't. 


Limitations: Sending messages more often could give different results as the number of sms sent were almost one third of the total sample, so more data should be provided for more accurate results.
from subprocess import call
call(['python', '-m', 'nbconvert', 'Investigate_a_Dataset.ipynb'])

