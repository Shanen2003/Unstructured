This is a LinkedIn Scraper, and is uses downloading each html page, as a unique way to get around LinkedIn trying to stop scraping. 

It takes in a csv of URLs, and utilizes playwright's connect_over_cdp in order to get around logging in, and espeically Captchas. 

Unfortunately, this does include opening your browser in test mode, as well as closing all otehr browser tabs. 
Hopefully I have given in depth enough information within the markdown and qmd to allow for a step by step process on how to set this up!
Here I will walk through all code in slighlty more detail for better understanding. 

First we import all necessary packages. The two most important ones are playwright and BeautifulSoup, which make up the two biggest blocks. 
We also have command lines to install playwright, if needed, with pip for windows and python3 for mac. 

Next we have a small test block to make sure the csv of data can be accessed, before setting up the test window. 

Next we open the browser we want to use to run the playwright script. We paste brave://version/ (or Chrome, etc.) and find a very important file path. 
We will keep the "Profile Path" handy as we will shortly need this, when we begin running the script. 

Next we need to close every session of the browser we are using. Use task manager to make sure nothign is running in the background. 

Next we will open the browser in debugging mode, go to pwoershell and run this code:

"C:\Program Files\BraveSoftware\Brave-Browser\Application\brave.exe" ^
--remote-debugging-port=____ ^
--user-data-dir="_____\User Data" ^
--profile-directory="Profile _"

Pick your debug port, the file path we just copied, and the profile number. Fill those in and run it! If everything goes well a browser will open. 

Now log in manually to the LinkedIn account you do not care about, and make 100% sure you are not in your personal LinkedIn. When reading about hwo to do this every article talked about how you will be banned. I hope neither of us are, but I do not want to take that chance. 

Now we reach the playwright code section :). 

The imported fiel should be a csv, with one column titled "LinkedIn" and each value being 1 URL to an account. 

Update the file paths and port number (same as when connecting in the command prompt), and run it. 

Your browser should begin changing, and slowly scrollign down. For each page we slowly scroll dwon. This does two things, it gives the page time to load, and on LinkedIn the page doesn't laod until you scroll past it. SO using a slow scorllign feature works perfect here. In the loop we pay attention to teh scroll height, if this does not change a couple times in a row, we know we are at the bottom of the page, and can save what we have found. 

We download the page content, and use BeautifulSoup's html parser to begin parsing the extracted info. We also extract the final section of all LinkedIn URLs and save the coup using this extracted section. On LinkedIn tehse are all unqiue and (almost always) based on the person's name, so they are a great way to name the files as tehy are saved. All files are saved as html documents, that way if anything happens, such as out accoutn being banned, we still have what we have scraped so far. 

Next we extract the important info out of eahc html file. We use glob to pull all html files out, so if anythign else gets accidentally put in there no problem. 






