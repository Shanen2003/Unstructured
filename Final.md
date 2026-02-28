# LinkedinScraper.qmd


Here we have all necessary file imports in order to run the playwright
scraper.

``` python
import asyncio
import pandas as pd
# pip install playwright
# python3 -m playwright install
from playwright.sync_api import sync_playwright, Playwright
import re
import time 
from bs4 import BeautifulSoup
from pathlib import Path
```

Here we load in the csv of the LinkedIns we would like to scrape. Please
do this with a smaller number and try to do multiple small batches.
LinkedIn will eventually catch on.

This is a nice way to test to make sure the csv works, and the file path
is correct. The csv should be one column of LinkedIn URLS, and a title
called

``` python
df = pd.read_csv("C:/Users/shane/Unstructured/Book1.csv")

for i in df["LinkedIn"]:
  print(i)
```

    https://www.linkedin.com/in/tommy-gillan/
    https://www.linkedin.com/in/shanenbateman/
    https://www.linkedin.com/in/andrew-j-connelly-b40504134/
    https://www.linkedin.com/in/aidan-sullivan11/
    https://www.linkedin.com/in/markfalvey/
    https://www.linkedin.com/in/adrian-figueroa/
    https://www.linkedin.com/in/joshua-rourke/
    https://www.linkedin.com/in/thatpong-bamrungchai-3aa108296/
    https://www.linkedin.com/in/art-by-annora/
    https://www.linkedin.com/in/michael-street-71880aa7/

Here is the kind of annoying part to set up. I currently cannot get past
Captchas, even if I automatically pull the log in code from my email. I
didn’t think to start training a CNN early enough to detect images.
Instead we will use an already open browser in test mode, using
connectOverCDP!

You then need to find the browser account you would like to scrape with.
Please make sure to log out of your LinkedIn if it is signed in. You
will be banned!

You should now sign into the burner LinkedIn you do NOT care about.

Next, paste this into your URL:

brave://version/

Here you can see what profile to use, and make sure you use the correct
one. Please copy the Profile Path, and paste it somewhere, as we will
sue it shortly.

Now go into task manager and close every version of the browser you will
be using to scrape, I use brave, so I make sure they are all closed.

Return to your command prompt and run this in the powershell, after
replacing the \*\*\*\* with the port you would like, adn your file path:

“C:Files-Browser.exe” ^ –remote-debugging-port=\_\_\_\_ ^
–user-data-dir=“\_\_\_\_\_Data” ^ –profile-directory=“Profile \_”

This may need to be changed if not running in Brave. The user-data-dir
is the first section of the Profile Path, and then the Profile 6 is the
last section.

When this is ran it should open the correct browser! Please again make
sure you are only logged into the LinkedIn you do not care about, as the
next step is scraping.

Here we begin the fun parts!

You need to be logged into your burner LinkedIn (I’m sure you’re tired
of this being mentioned), and change the files paths.

The first section connects and reads in the csv. It should be titled
LinkedIn, but feel free to change the word inside the \[“”\] to your
column name.

It uses a playwright script which clicks on the page, and then begins
slowly scrolling down. After a couple times of the page not scrolling
down, the page is saved to a folder in your files as html. LinkedIn
class names change constantly, which can be very annoying. Instead of
playing tag with this, I downloaded all pages html to a folder. This
also allows for the account to be banned and not loose the pages
scraped!

The files are named after the unqiue url section of people’s LinkedIn
URLs, so there is no overlap in file names.

``` python
pw = sync_playwright().start()

# Change to your port
browser = pw.chromium.connect_over_cdp("http://127.0.0.1:****")

context = browser.contexts[0] 
page = context.pages[0] 

# Change to file path
df = pd.read_csv("C:/Users/shane/Unstructured/Book1.csv")

# Change to column title
for url in df["LinkedIn"]:

    page.goto(url)

    previousheight = 0

    nochange = 0

    try: 
        page.wait_for_load_state("domcontentloaded", timeout = 120000)
    except Exception:
        pass

    page.click("body")

    while nochange < 3:

        for i in range(10):

            page.mouse.wheel(0, 100)
            time.sleep(1)

        current_height = page.evaluate("() => document.body.scrollHeight")

        if current_height > previousheight:
            previousheight = current_height
            nochange = 0
        else:
            nochange += 1

    html = page.content()
    soup = BeautifulSoup(html, "html.parser")

    data = re.findall(r"in/(.+)/$", url)

    # Change the first section of this file path to your folder, then leave the /{data[0]}.html section. 
    path = Path(f"C:/Users/shane/Unstructured/profiles/{data[0]}.html")
    path.write_text(soup.prettify(), encoding="utf-8")
```

Here we now scrape the html we have downloaded. We read in each html
file in the folder one by one. We remove all tags from it, and create a
massive block of text of all the LinkedIn info.

Here we use regex expressions to specifically select certain
characteristics. Sometimes LinkedIn can be very annoying, as items can
be left blank or have their orders swapped. One example is pronouns
under the name, or when someone has multiple jobs at the same company
the title, company, and date; positions are swapped around.

It creates a new dataframe, with all our new found info. It leaves
blanks when there is no info.

``` python
df = pd.read_csv("C:/Users/shane/Unstructured/Book1.csv")

# Change to the same folder path from above
folder = Path("C:/Users/shane/Unstructured/profiles")

l = []
rows = []
c = 0

for file in folder.glob("*.html"):
    row = {}
    soup = BeautifulSoup(file.read_text(encoding="utf-8"), "html.parser")
    text = soup.get_text(separator="\n", strip=True)
    l.append(text)

    row["URL"] = df.loc[c, "LinkedIn"]
        
    row["Name"] = re.findall(r"^Message\s*\n(?:Connect\s*\n)?(.+)\s*\n(?:.+/.+\s*\n)?.\s*1st", text, re.MULTILINE)

    row["Location"] = re.findall(r"\n(.+)\s*\n.\s*\nContact", text)
    
    row["Experience_Role"] = re.findall(r"^Experience\s*\n(.+)\s*\n", text, re.MULTILINE)

    row["Experience_Company"] = re.findall(r"^Experience\s*\n.*\s*\n(.+)\s*\n", text, re.MULTILINE)

    row["Experience_Dates"] = re.findall(r"^Experience\s*\n.*\s*\n.*\s*\n(.*)\s*\n", text, re.MULTILINE)

    row["Experience_Location"] = re.findall(r"^Experience\s*\n.*\s*\n.*\s*\n.*\s*\n(.*)\s*\n", text, re.MULTILINE)

    row["School"] = re.findall(r"^Education\s*\n(.+)\s*\n", text, re.MULTILINE)

    row["School_Major"] = re.findall(r"^Education\s*\n.*\s*\n(.+)\s*\n", text, re.MULTILINE)

    row["School_Timeline"] = re.findall(r"^Education\s*\n.*\s*\n.*\s*\n([A-Za-z]{3}?\s.+|\d+.+)\s*\n", text, re.MULTILINE)

    if row["Experience_Company"] and any(x in row["Experience_Company"][0] for x in ["yrs", "mos"]):
        row["Experience_Role"], row["Experience_Company"], row["Experience_Dates"] = row["Experience_Dates"], row["Experience_Role"], row["Experience_Company"]

    rows.append(row)

    c+=1

df_more = pd.DataFrame(rows)
df_more
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

<table class="dataframe" data-quarto-postprocess="true" data-border="1">
<thead>
<tr style="text-align: right;">
<th data-quarto-table-cell-role="th"></th>
<th data-quarto-table-cell-role="th">URL</th>
<th data-quarto-table-cell-role="th">Name</th>
<th data-quarto-table-cell-role="th">Location</th>
<th data-quarto-table-cell-role="th">Experience_Role</th>
<th data-quarto-table-cell-role="th">Experience_Company</th>
<th data-quarto-table-cell-role="th">Experience_Dates</th>
<th data-quarto-table-cell-role="th">Experience_Location</th>
<th data-quarto-table-cell-role="th">School</th>
<th data-quarto-table-cell-role="th">School_Major</th>
<th data-quarto-table-cell-role="th">School_Timeline</th>
</tr>
</thead>
<tbody>
<tr>
<td data-quarto-table-cell-role="th">0</td>
<td>https://www.linkedin.com/in/tommy-gillan/</td>
<td>[Adrian 'Lobo' Figueroa]</td>
<td>[Leicester, Massachusetts, United States]</td>
<td>[Freelance Owner / Creative Producer]</td>
<td>[10:60 Designs · Freelance]</td>
<td>[Feb 2008 - Present · 18 yrs 1 mo]</td>
<td>[Boston, Massachusetts, United States · Remote]</td>
<td>[University of Massachusetts Amherst]</td>
<td>[Bachelor's Degree In Individual Concentration...</td>
<td>[2007 – 2009]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">1</td>
<td>https://www.linkedin.com/in/shanenbateman/</td>
<td>[Aidan Sullivan]</td>
<td>[Greater Boston]</td>
<td>[]</td>
<td>[]</td>
<td>[]</td>
<td>[]</td>
<td>[Suffolk University]</td>
<td>[Bachelor's degree, Marketing]</td>
<td>[2019 – 2023]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">2</td>
<td>https://www.linkedin.com/in/andrew-j-connelly-...</td>
<td>[Andrew J. Connelly]</td>
<td>[Scituate, Massachusetts, United States]</td>
<td>[Production Assistant]</td>
<td>[Medlive - A PlatformQ Health Brand · Full-time]</td>
<td>[Feb 2024 - Present · 2 yrs 1 mo]</td>
<td>[Owner]</td>
<td>[University of Massachusetts Amherst]</td>
<td>[Bachelor's degree with Individual Concentrati...</td>
<td>[]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">3</td>
<td>https://www.linkedin.com/in/aidan-sullivan11/</td>
<td>[Annora Lemieux]</td>
<td>[Metro Jacksonville]</td>
<td>[Murals &amp; Public Art Installations]</td>
<td>[Art by Annora Inc]</td>
<td>[4 yrs 7 mos]</td>
<td>[Self-employed]</td>
<td>[IBM]</td>
<td>[AI Engineering]</td>
<td>[Nov 2025 – Present]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">4</td>
<td>https://www.linkedin.com/in/markfalvey/</td>
<td>[Joshua Rourke]</td>
<td>[Greater Boston]</td>
<td>[Business Development Account Manager]</td>
<td>[South Bend Cubs · Full-time]</td>
<td>[Nov 2025 - Present · 4 mos]</td>
<td>[South Bend, Indiana, United States · On-site]</td>
<td>[Dean College]</td>
<td>[Bachelor's degree, Sports Management]</td>
<td>[Aug 2019 – May 2023]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">5</td>
<td>https://www.linkedin.com/in/adrian-figueroa/</td>
<td>[Mark Falvey]</td>
<td>[Greater Boston]</td>
<td>[Painter]</td>
<td>[M.A. Falvey Painting Service, Inc.]</td>
<td>[Jun 2014 - Present · 11 yrs 9 mos]</td>
<td>[Greater Boston Area]</td>
<td>[Providence College]</td>
<td>[Bachelor's degree, Marketing]</td>
<td>[2018 – 2022]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">6</td>
<td>https://www.linkedin.com/in/joshua-rourke/</td>
<td>[Michael Street]</td>
<td>[Greater Boston]</td>
<td>[On-site]</td>
<td>[Group 1 Automotive]</td>
<td>[Full-time · 3 yrs 9 mos]</td>
<td>[Service Director]</td>
<td>[Southern New Hampshire University]</td>
<td>[Bachelor of Science - BS, Business Administra...</td>
<td>[Nov 2025 – May 2026]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">7</td>
<td>https://www.linkedin.com/in/thatpong-bamrungch...</td>
<td>[Shanen Bateman]</td>
<td>[Plymouth, Massachusetts, United States]</td>
<td>[Operations Manager]</td>
<td>[Reale Built LLC]</td>
<td>[May 2025 - Oct 2025 · 6 mos]</td>
<td>[Python (Programming Language), Microsoft Exce...</td>
<td>[University of Notre Dame - Mendoza College of...</td>
<td>[Master of Science - MS]</td>
<td>[]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">8</td>
<td>https://www.linkedin.com/in/art-by-annora/</td>
<td>[Thatpong Bamrungchai]</td>
<td>[Boston, Massachusetts, United States]</td>
<td>[]</td>
<td>[]</td>
<td>[]</td>
<td>[]</td>
<td>[UMass Boston]</td>
<td>[2023 – 2027]</td>
<td>[]</td>
</tr>
<tr>
<td data-quarto-table-cell-role="th">9</td>
<td>https://www.linkedin.com/in/michael-street-718...</td>
<td>[Thomas (Tommy) Gillan]</td>
<td>[Notre Dame, Indiana, United States]</td>
<td>[Videographer/ Video Logger]</td>
<td>[Fighting Irish Media]</td>
<td>[Aug 2025 - Present · 7 mos]</td>
<td>[Notre Dame, Indiana, United States]</td>
<td>[Notre Dame MS in Business Analytics]</td>
<td>[Master of Science - MS, Business Analytics]</td>
<td>[Aug 2025 – May 2026]</td>
</tr>
</tbody>
</table>

</div>

Here we can export the data set! Now with our new info.

``` python
df_more.to_csv("C:/Users/shane/Unstructured/LinkedIn_Data.csv")
```
