# Craigslist-crawler
A website crawler written with Python
import time
import smtplib
import requests
from bs4 import BeautifulSoup
import difflib

sender='****@gmail.com'
receiver='****@mail'

models=['mercedes','bmw']
min='min_price=0'
max='max_price=100000'

maxtime=7200
timestart=time.time()

cities=['boston','providence']
oldresult=[]
count=1

while(time.time()-timestart<maxtime):
        newresult=[]
        for city in cities:
                for model in models:
                        #This is the url to search through
                        url='http://'+city+'.craigslist.org/search/cto?'+min+'&'+max
                        #'auto_make_model='+model
                        print(model)
                        
                        page=1
 
                        while(page<2):
                                #Get all the content in the webpage
                                source_code=requests.get(url)
                                plain_text=source_code.text
                                soup=BeautifulSoup(plain_text,'html.parser')
                                #Get all the content of 'a' with a class' hrlnk'
                                for link in soup.findAll('a',{'class':'hdrlnk'}):
                                        #Get the url in the link
                                        href=link.get('href')
                                        #Some of the url are complete and some are not, so we add the if statement
                                        if(href[0]!='h'):
                                                href='http://'+city+'.craigslist.org/'+href
                                        
                                        
                                        newresult.append(href)
                                page+=1
        
        print('circle '+str(count)+' is done')
        difference=difflib.Differ()
        difflist=list(difference.compare(oldresult,newresult))
        # print(difflist)
        msg=''
        server=smtplib.SMTP('smtp.gmail.com',587)
        server.starttls()
        server.login('sender','password of email')
        if (count!=1):
 
                for link in difflist:
                        #'+' means there is something added into the newresult compared with oldresult,'-' means something is gone.
                        if link[0]=='+':
                                msg+='Searched on'+time.asctime()+link[2:]+'\n'
                if  (msg!=''):
                        message=msg
                        print(message)
                        server.sendmail('sender','receiver',message)
                        
                        print('Email successfully sent')
                else:
                        server.sendmail('sender','receiver','Nothing new is found')
        else:
                server.sendmail('sender','receiver','This is the first turn')
                
        oldresult=newresult
        count+=1
        time.sleep(1800)

#print 'Done: maxtime is up.' + ' '+ time.asctime()


