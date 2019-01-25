# Hypothesis-Testing-In-Python
'''Hypothesis: University towns have their mean housing prices less effected by recessions. Run a t-test to compare the ratio of the mean price of houses in university towns the quarter before the recession starts compared to the recession bottom. (price_ratio=quarter_before_recession/recession_bottom)

The following data files are available for this assignment:

From the Zillow research data site there is housing data for the United States. In particular the datafile for all homes at a city level, City_Zhvi_AllHomes.csv, has median home sale prices at a fine grained level.
From the Wikipedia page on college towns is a list of university towns in the United States which has been copy and pasted into the file university_towns.txt.
From Bureau of Economic Analysis, US Department of Commerce, the GDP over time of the United States in current dollars (use the chained value in 2009 dollars), in quarterly intervals, in the file gdplev.xls. For this assignment, only look at GDP data from the first quarter of 2000 onward.'''

##################################################################################################################################

###############STEP1


import pandas as pd
import numpy as np
import csv
import scipy
from scipy.stats import ttest_ind

def get_list_of_university_towns():
    '''Returns a DataFrame of towns and the states they are in from the 
    university_towns.txt list.'''
    
    #read/import txt file
    df=pd.read_fwf('university_towns.txt')
    
    
    #read text file into a list of lists. The inner list will be separated by '['
    list_of_lists=[]
    with open('university_towns.txt') as f:
        for line in f:
            inner_list=[line.strip()for line in line.split('[')]
            list_of_lists.append(inner_list)
    #turn list of lists into a data frame
    df=pd.DataFrame(list_of_lists)
    
    #create State column using conditionals
    #if df[1] doesn't have 'edit]', it won't write the state name, else write the name of the state in df[0]
    df['State']=np.where(df[1]=='edit]',df[0],'notastate')
    
    #create a for loop to replace 'notastate' to the right state.
    count=0
    for i in df['State']:
        if i=='notastate':
            df['State'][count]=df['State'][count-1]
        count=count+1

    #create a RegionName using conditionals
    #if df[2] doesn't have 'edit]', it will write the region name, else write notaregion
    df['RegionName']=np.where(df[1]!='edit]',df[0],np.nan)
    
    #For "RegionName", removing every character from " (" to the end.
    df['RegionName']=df['RegionName'].str.split('(').str[0] #remove from the parenthesis
    df['RegionName']=df['RegionName'].str.rstrip() #remove the space after the region Name

    
    #delete first four columns and get rid of rows with NaN
    df=df[['State','RegionName']].dropna()
    df=df.reset_index() #relabel the index
    df=df.drop('index',axis=1)

    
    return df

###############STEP2

def get_recession_start():
        
    '''Returns the year and quarter of the recession start time as a 
    string value in a format such as 2005q3'''
    
    #import excel spreadsheet
    df=pd.read_excel('gdplev.xls')
    
    #replace header value with 5th row of the dataset
    df.columns=['Current-Dollar and "Real" Gross Domestic Product','GDP in billions of current dollars','GDP in billions of chained 2009 dollars1','NAN','yearquarter','GDP in billions of current dollars2', 'GDP in billions of chained 2009 dollars','NAN2']
    df=df[218:]
    
    #drop unnecessary columns and reset index
    df=df.drop(['Current-Dollar and "Real" Gross Domestic Product','GDP in billions of current dollars','NAN','GDP in billions of chained 2009 dollars1','GDP in billions of current dollars2','NAN2'],axis=1)
    df=df.reset_index()
    
    #create a column that indicates when GDP is rising or declining
    count=1
    lst=['NA'] #the first row is NA
    while count<len(df['GDP in billions of chained 2009 dollars']):
        if df['GDP in billions of chained 2009 dollars'][count]<df['GDP in billions of chained 2009 dollars'][count-1]:
            lst.append('decline')
        elif df['GDP in billions of chained 2009 dollars'][count]==df['GDP in billions of chained 2009 dollars'][count-1]:
            lst.append('same')
        else:
            lst.append('rise')
            
        count=count+1
      
    #append lst to dataframe
    df['d/r']=pd.Series(lst).values
    
    #find all the recession starts
    recessionstartlst=[]
    count=1
    targetindex=0
    while count<len(df['d/r']):
        if df['d/r'][count]=='decline' and df['d/r'][count+1]=='decline' and df['d/r'][count+2]=='rise' and df['d/r'][count+3]=='rise':
            recessionstartlst.append(df['yearquarter'][count])
            targetindex=count
        count=count+1
   
    #check and see if rows right above recessionstartlst is also 'decline'. We want to go upwards and check each row until we reach the decline right before the first 'rise' to acknowledge the start of GDP decline
    while df['d/r'][targetindex]=='decline':
        if df['d/r'][targetindex]=='decline':
            targetindex=targetindex-1
            
    ansindex=targetindex+1
    
    
    return df['yearquarter'][ansindex]

###############STEP3

def get_recession_end():
    '''Returns the year and quarter of the recession end time as a 
    string value in a format such as 2005q3'''
    
    #import excel spreadsheet
    df=pd.read_excel('gdplev.xls')
    
    #replace header value with 5th row of the dataset
    df.columns=['Current-Dollar and "Real" Gross Domestic Product','GDP in billions of current dollars','GDP in billions of chained 2009 dollars1','NAN','yearquarter','GDP in billions of current dollars2', 'GDP in billions of chained 2009 dollars','NAN2']
    df=df[218:]
    
    #drop unnecessary columns
    df=df.drop(['Current-Dollar and "Real" Gross Domestic Product','GDP in billions of current dollars','NAN','GDP in billions of chained 2009 dollars1','GDP in billions of current dollars2','NAN2'],axis=1)
    
    df=df.reset_index()
    
    #create a column that indicates when GDP is rising or declining
    count=1
    lst=['NA']
    while count<len(df['GDP in billions of chained 2009 dollars']):
        if df['GDP in billions of chained 2009 dollars'][count]<df['GDP in billions of chained 2009 dollars'][count-1]:
            lst.append('decline')
        elif df['GDP in billions of chained 2009 dollars'][count]==df['GDP in billions of chained 2009 dollars'][count-1]:
            lst.append('same')
        else:
            lst.append('rise')
            
        count=count+1
      
    #append lst to dataframe
    df['d/r']=pd.Series(lst).values
    
    #find all the recession starts
    recessionendlst=[]
    count=1
    while count<len(df['d/r']):
        if df['d/r'][count]=='decline' and df['d/r'][count+1]=='decline' and df['d/r'][count+2]=='rise' and df['d/r'][count+3]=='rise':
            recessionendlst.append(df['yearquarter'][count+3])
        count=count+1

       
    return recessionendlst[0]

###############STEP4

def get_recession_bottom():
    '''Returns the year and quarter of the recession bottom time as a 
    string value in a format such as 2005q3'''
    
    #import excel spreadsheet
    df=pd.read_excel('gdplev.xls')
    
    #replace header value with 5th row of the dataset
    df.columns=['Current-Dollar and "Real" Gross Domestic Product','GDP in billions of current dollars','GDP in billions of chained 2009 dollars1','NAN','yearquarter','GDP in billions of current dollars2', 'GDP in billions of chained 2009 dollars','NAN2']
    df=df[218:]
    
    #drop unnecessary columns
    df=df.drop(['Current-Dollar and "Real" Gross Domestic Product','GDP in billions of current dollars','NAN','GDP in billions of chained 2009 dollars1','GDP in billions of current dollars2','NAN2'],axis=1)
    
    df=df.reset_index()
    
    #create a column that indicates when GDP is rising or declining
    count=1
    lst=['NA']
    while count<len(df['GDP in billions of chained 2009 dollars']):
        if df['GDP in billions of chained 2009 dollars'][count]<df['GDP in billions of chained 2009 dollars'][count-1]:
            lst.append('decline')
        elif df['GDP in billions of chained 2009 dollars'][count]==df['GDP in billions of chained 2009 dollars'][count-1]:
            lst.append('same')
        else:
            lst.append('rise')
            
        count=count+1
      
    #append lst to dataframe
    df['d/r']=pd.Series(lst).values
    
    #find all the recession starts
    recessionbottomlst=[]
    count=1
    while count<len(df['d/r']):
        if df['d/r'][count]=='decline' and df['d/r'][count+1]=='decline' and df['d/r'][count+2]=='rise' and df['d/r'][count+3]=='rise':
            recessionbottomlst.append(df['yearquarter'][count+1])
        count=count+1
       
    return recessionbottomlst[0]

###############STEP4

def convert_housing_data_to_quarters():
    '''Converts the housing data to quarters and returns it as mean 
    values in a dataframe. This dataframe should be a dataframe with
    columns for 2000q1 through 2016q3, and should have a multi-index
    in the shape of ["State","RegionName"].
    
    Note: Quarters are defined in the assignment description, they are
    not arbitrary three month periods.
    
    The resulting dataframe should have 67 columns, and 10,730 rows.
    '''
    #read csv file
    df=pd.read_csv('City_Zhvi_AllHomes.csv')
    #delete unwanted columns
    df=df.drop(df.columns[6:51],axis=1)
    
    #focus on just quarters
    dfq=df.drop(df.columns[:6], axis=1)
    
    #calculate three columns at a time into one column
    dfq_mean=pd.concat([dfq.ix[:,i:i+3].mean(axis=1) for i in range(0,len(dfq.columns),3)], axis=1)
    
    #rename columns
    yrs=list(np.arange(2000,2017))
    qtrs=['q1','q2','q3','q4']
    yrqt=[]
    for i in yrs:
        for k in qtrs:
            yrqt.append(str(i)+k)
    newtitles=yrqt[0:len(dfq_mean.columns)]
    dfq_mean.columns=newtitles
    
    #retrieve State and RegionName
    df_stateregion=df[['State','RegionName']]
    states = {'OH': 'Ohio', 'KY': 'Kentucky', 'AS': 'American Samoa', 'NV': 'Nevada', 'WY': 'Wyoming', 'NA': 'National', 'AL': 'Alabama', 'MD': 'Maryland', 'AK': 'Alaska', 'UT': 'Utah', 'OR': 'Oregon', 'MT': 'Montana', 'IL': 'Illinois', 'TN': 'Tennessee', 'DC': 'District of Columbia', 'VT': 'Vermont', 'ID': 'Idaho', 'AR': 'Arkansas', 'ME': 'Maine', 'WA': 'Washington', 'HI': 'Hawaii', 'WI': 'Wisconsin', 'MI': 'Michigan', 'IN': 'Indiana', 'NJ': 'New Jersey', 'AZ': 'Arizona', 'GU': 'Guam', 'MS': 'Mississippi', 'PR': 'Puerto Rico', 'NC': 'North Carolina', 'TX': 'Texas', 'SD': 'South Dakota', 'MP': 'Northern Mariana Islands', 'IA': 'Iowa', 'MO': 'Missouri', 'CT': 'Connecticut', 'WV': 'West Virginia', 'SC': 'South Carolina', 'LA': 'Louisiana', 'KS': 'Kansas', 'NY': 'New York', 'NE': 'Nebraska', 'OK': 'Oklahoma', 'FL': 'Florida', 'CA': 'California', 'CO': 'Colorado', 'PA': 'Pennsylvania', 'DE': 'Delaware', 'NM': 'New Mexico', 'RI': 'Rhode Island', 'MN': 'Minnesota', 'VI': 'Virgin Islands', 'NH': 'New Hampshire', 'MA': 'Massachusetts', 'GA': 'Georgia', 'ND': 'North Dakota', 'VA': 'Virginia'}
    df_stateregion['State1']=df_stateregion['State'].map(states) #match dictionary of states to their abbreviation
    df_stateregion=df_stateregion[['State1','RegionName']]
    df_stateregion.rename(columns={'State1':'State'},inplace=True)
    
    #join two dataframes along columns
    df_final=pd.concat([df_stateregion,dfq_mean],axis=1)
    
    df_final=df_final.set_index(['State','RegionName'])
    
    pd.options.display.float_format = '{:f}'.format #convert scientific notation to float notation
    return df_final

###############STEP5/FINAL STEP

def run_ttest():
    '''First creates new data showing the decline or growth of housing prices
    between the recession start and the recession bottom.Then runs a ttest
    comparing the university town values to the non-university towns values, 
    return whether the alternative hypothesis (that the two groups are the same)
    is true or not as well as the p-value of the confidence. 
    
    Return the tuple (different, p, better) where different=True if the t-test is
    True at a p<0.01 (we reject the null hypothesis), or different=False if 
    otherwise (we cannot reject the null hypothesis). The variable p should
    be equal to the exact p value returned from scipy.stats.ttest_ind(). The
    value for better should be either "university town" or "non-university town"
    depending on which has a lower mean price ratio (which is equivilent to a
    reduced market loss).'''
    
    #get quarter before recession
    qtr=get_recession_start()[4:]
    yr=int(get_recession_start()[0:4])
    
    if qtr == 'q1':
        qbr='q4'
        yr=str(yr-1)
    if qtr == 'q2':
        qbr='q1'
    if qtr =='q3':
        qbr='q2'
    else:
        qbr='q3'
        
    modqbr=str(yr)+str(qbr)

    #first create new data showing decline/growth of housing prices between recession start and recession bottom
    df=convert_housing_data_to_quarters().loc[:,modqbr:get_recession_bottom()]
    df=df.drop(df.columns[1:-1],axis=1)#focus on the quarter before recession and bottom
    df=df.drop_duplicates(keep=False)
   
    #compare the university town values to the non-university towns values. 
    #Get the common towns between the two dataframes.
    unitowns=pd.merge(df, get_list_of_university_towns(),how='inner',left_index=True, right_on=['State','RegionName'])
    unitowns=unitowns.set_index(['State','RegionName'])
    # get the towns from the outer portion of the venn diagram...in other words, the data that didn't intersect.
    nonunitowns=pd.merge(df,get_list_of_university_towns(),how='outer',left_index=True,right_on=['State','RegionName'],indicator=True)
    nut=nonunitowns[nonunitowns._merge != 'right_only'].query('_merge != "both" ')
    nut=nut.set_index(['State','RegionName'])
  
    #determine 'better'
    #calculate price ratio in ut
    unitowns['pr']=unitowns[modqbr]/unitowns[get_recession_bottom()]
    unitowns=unitowns.drop_duplicates(keep=False) #drop all duplicate rows
    prut=unitowns['pr'].mean()
    
    #calculate price ratio in nut
    nut['pr']=nut[modqbr]/nut[get_recession_bottom()]
    prnut=nut['pr'].mean()
    
    if prut<prnut:
        better="university town"
    else:
        better="non-university town"
        
        
    #calulate p-value of university towns
    pvalue=ttest_ind(unitowns['pr'],nut['pr'],nan_policy='omit')[1]
    
    #determine 'different'
    if pvalue<0.01:
        different=True
    else:
        different=False
        
    return (different, pvalue, better)
run_ttest()
