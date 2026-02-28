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

We download the page content, and use BeautifulSoup's html parser to begin parsing the extracted info. We also extract the final section of all LinkedIn URLs and save the coup using this extracted section. On LinkedIn tehse are all unqiue and (almost always) based on the person's name, so they are a great way to name the files as tehy are saved. All files are saved as html documents, that way if anything happens, such as out accoutn being banned, we still have what we have scraped so far. LinkedIn's html tags also change, whihc makes parsing it very difficult. So downloading the html serves as a way to make it stay still!

Next we extract the important info out of eahc html file. We use glob to pull all html files out, so if anythign else gets accidentally put in there no problem. We create a variable to keep track of which row we are on, adn add the LinkedIn URL, as well as creating a dictionaruy and list to so we can create a final dataframe when finished. 

The beautifulsoup line reads in the data, and allows us to parse it. We tehn use get_text which takes out all text from teh document, and by using a new line seperator (\n), all text is on seperate lines. This makes it very easy to visually distinguish between elements. From here we have regex lines to find teh data we want. Some can get quite complicated (name and school timeline). Name had to account for Message beign above, sometimes connect being above, and then sometimes having pronouns on teh next line, and then 1st. Schools tiemline was dissifult as many did not include this, and we would often get unrealted data. This meant we have tio make sure if it included text it was 3 letters (as LinkedIn uses 3 letter month abbrevations) or only includes numbers. 

Another LinkedIn issue, was hwo the order of company, role, and time were swapped if an employee worked at the same company for mutliple roles. This caused (the most confusing) if statement ever. It loosk to see if the comapny has yrs or mos (LinkedIns abbrevaitions for year and amonths) and if it does reorder teh job info to be in the correct spots. 

For each LinkedIn we make a new dictionary, which is then added to a list. After the loop we turn this into a dataframe, which we can then export! 

Please contact me through my LinkedIn (if it stays standing) or any conacts on my GitHub page if any issues arrise. 





