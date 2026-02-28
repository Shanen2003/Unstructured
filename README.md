This is a LinkedIn Scraper, and is uses downloading each html page, as a unique way to get around LinkedIn trying to stop scraping. 

It takes in a csv of URLs, and utilizes playwright's connect_over_cdp in order to get around logging in, and espeically Captchas. 

Unfortunately, this does include opening your browser in test mode, as well as closing all otehr browser tabs. 
Hopefully I have given in depth enough information within the markdown and qmd to allow for a step by step process on how to set this up!
Here I will walk through all code in slighlty more detail for better understanding. 

First we import all necessary packages. The two most important ones are playwright and BeautifulSoup, which make up the two biggest blocks. 
We also have command lines to install playwright, if needed, with pip for windows and python3 for mac. 

Next we have a small test block to make sure the csv of data can be accessed, before setting up the test window. 

Next we open the browser we want to use to run the playwright script. We paste brave://version/ (or Chrome, etc.) and find a very important file path. 
We will keep the "Profile Path" handy as we will shortly need this, wehn we begin running the script. 

Next we need to close every session of the browser, 
