# AMAZON REVIEW RATING

from bs4 import BeautifulSoup as bs
from selenium import webdriver
import pandas as pd
import re
from sklearn import *
link='https://www.amazon.in/s?k=BOOKS&qid=1569766886&ref=sr_pg_1'

bookauthor=[]
bookyear=[]
bookrating=[]
bookreviews=[]
bookprice=[]
bookactualPrice=[]
nclass=[]
for j in range(1,5):
    #driver=webdriver.Chrome("C:\\Driver\\chromedriver_Win32\\chromedriver.exe")
    driver = webdriver.Chrome(executable_path='D:\\driver\\chromedriver.exe')

    
    driver.get(link)
    
    content=driver.page_source
    cont=bs(content)
    for i in cont.find_all('div',attrs={'class':'sg-col sg-col-4-of-12 sg-col-4-of-16 sg-col-4-of-20'}):
        authoryear=i.find('div',attrs={'class':'a-row a-size-base'})
        rating=i.find('div',attrs={'class':'a-row a-size-small'})
        price=i.find('span',attrs={'class':'a-price'})
        actualPrice=i.find('span',attrs={'class':'a-price a-text-price'})
        try:
            authoryear=authoryear.text
            author=re.findall(r'\n\D+\n',authoryear)
            try:
                author=author.strip()
            except:
                z=''.join(authoryear)
                sp=z.split()
                author=sp[1]
        except:
            author='various'
        try:
            year=authoryear.split()
            try:
                year=year[-1]
                year=int(year)
            except:
                year=None
        except:
            year=None
        try:    
            rating=rating.text
            rating=re.findall(r'\d.\d|\d+',rating)
            reviews=int(rating[-1])
            rating=float(rating[0])
        except:
            rating=None
            reviews=None
        try:
            price=price.text
            price=re.findall(r'\d+',price)
            price=float(price[0])
        except:
            price=None
        try:
            actualPrice=actualPrice.text
            actualPrice=re.findall(r'\d+',actualPrice)
            actualPrice=float(actualPrice[0])
        except:
            actualPrice=None
        bookauthor.append(a  uthor)
        bookyear.append(year)
        bookrating.append(rating)
        bookreviews.append(reviews)
        bookprice.append(price)
        bookactualPrice.append(actualPrice)
        
    for i in cont.find_all('ul',attrs={'class':'a-pagination'}):
        
    
        nextP=i.find('li',attrs={'class':'a-last'})
    b=nextP.a
    c=b.attrs
    link="https://www.amazon.in"+c['href']

    
#df=pd.DataFrame({'Author':bookauthor,})
#df.to_csv('books.csv',encoding='utf8')


x=pd.DataFrame({'Year':bookyear,'Price':bookprice,'Actual Price':bookactualPrice,'User Reviews':bookreviews,'Author':bookauthor})
x=x[['Actual Price', 'Price', 'User Reviews', 'Year', 'Author']]
import numpy as np
x=np.array(x)
y=np.array(bookrating)
for i in range(len(y)):
    if(y[i]==None):
        y[i]=3
from sklearn.preprocessing import LabelEncoder
lbl=LabelEncoder()
x[:,4]=lbl.fit_transform(x[:,4])

from sklearn.impute import SimpleImputer
imp=SimpleImputer()
x=imp.fit_transform(x)

from sklearn.model_selection import train_test_split
xtrain,xtest,ytrain,ytest=train_test_split(x,y,test_size=0.2)

print('Brain started getting train from past experiences')

from sklearn.ensemble import RandomForestRegressor
lr=RandomForestRegressor(n_estimators=1000)
lr.fit(xtrain,ytrain)
ypred=lr.predict(xtest)

xaxis=np.linspace(1,len(ytest),len(ytest))
import matplotlib.pyplot as plt
plt.xlabel('Time/series')
plt.ylabel('Rating')
plt.plot(xaxis,ypred,color='blue',label='Predicted')
plt.plot(xaxis,ytest,color='red',label='Actual')
plt.legend()
plt.show()

print("Congratulations brain is trained for future predictions. Press any key for entering GUI mode")
timepass=input()

from tkinter import *
window=Tk()
window.title('Timepass')
window.geometry('350x350')
lbl0=Label(window,text='Please write only integer/floating value')
lbl0.grid(row=0,column=0)   
lbl=Label(window,text='Author')
lbl.grid(row=1,column=0)   
lbl1=Label(window,text='Year')
lbl1.grid(row=2,column=0)
lbl2=Label(window,text='Price')
lbl2.grid(row=3,column=0)
'''lbl3=Label(window,text='Actual Price')
lbl3.grid(row=4,column=0)
lbl4=Label(window,text='Reviews')
lbl4.grid(row=5,column=0)'''
ent=Entry(window,width=10)
ent.grid(row=1,column=1)
ent1=Entry(window,width=10)
ent1.grid(row=2,column=1)   
ent2=Entry(window,width=10)
ent2.grid(row=3,column=1)   
'''ent3=Entry(window,width=10)
ent3.grid(row=4,column=1)   
ent4=Entry(window,width=10)
ent4.grid(row=5,column=1)'''   
def clicked():
    count=0
    value1=ent.get()
    value2=ent1.get()
    value3=ent2.get()
    #value4=ent3.get()
    #value5=ent4.get()
    try:
        value1=int(value1)
        value2=int(value2)
        value3=int(value3)
        #value4=int(value4)
        #value5=int(value5)
    except ValueError:
        #lbl3=Label(window,text="Values are not in requested format")
       # lbl3.grid(row=5,column=0)
        count=1
    if(count==0):
        win=Tk()
        window.destroy()
        win.geometry('350x350')
        lbl2=Label(win,text='The rating for inputted value will be:')
        lbl2.grid(row=0,column=0)
        res=lr.predict([[value3,value2,value1]])
        lbl2=Label(win,text=res)
        lbl2.grid(row=1,column=0)
btn=Button(window,text='Submit',command=clicked)
btn.grid(row=7,column=1)
window.mainloop()
