# IMDB-Metacritic-Scraper
This project scrapes IMDB for titles, year released, ratings, votes, revenues and can easily be updated for more/applied to other sites.
It examines the top 20 pages as they each contain 50 films and extracts the content.


import pandas as pd
import requests
from IPython.core.display import clear_output
from bs4 import BeautifulSoup
import time
from random import randint

# Data we are collecting.
Titles = []
Released = []
IMDB_Ratings = []
Meta_Ratings = []
Rating_Submissions = []
Revenue = []

# For keeping track of time spent scraping as to not bother the site.
Initial_Scrape = time.time()
Total_Requests = 1
Movie_Num = 1

for i in range(0, 21):
    page = i * 50 + 1

    # IMDB URL for titles released between 1/1/2016 and 6/1/2019 sorted by the most user ratings given. 
    url = "https://www.imdb.com/search/title?release_date=2016-01-01,2019-06-01&sort=num_votes,desc," \
          "desc&start={}&ref_=adv_nxt".format(page)

    # Requests and recieves response. Fetches data.
    Response = requests.get(url)
    soup = BeautifulSoup(Response.text, "html.parser")
    lister_items = soup.find_all("div", class_="lister-item")
    for Single_Item in lister_items:
        Single_Item_Content = Single_Item.find("div", class_="lister-item-content")

        # Pulls the movie title.
        Title = Single_Item_Content.h3.a.get_text()
        # Pulls the year the movie was released.
        Date = Single_Item_Content.h3.find("span", class_="lister-item-year").get_text()
        # Overall IMDB movie rating given by voters.
        IMDB = Single_Item_Content.find("div", class_="ratings-imdb-rating")["data-value"]
        # Overall Metacritic rating given by voters.
        Meta = Single_Item_Content.find("div", class_="ratings-metascore")
        if Meta is not None:
            Meta = Meta.span.get_text()
        else:
            Meta = "Not Available"
        # Total votes cast for IMDB ratings and revenue grossed for films.
        Total_Votes = Single_Item_Content.find("p", class_="sort-num_votes-visible").find_all("span",
                                                                                             attrs={"name": "nv"})
        Rating = Total_Votes[0]["data-value"]
        if len(Total_Votes) > 1:
            Gross_Total = Total_Votes[1]["data-value"]
        else:
            Gross_Total = "Not Available"

        # Data is added to the global array. 
        Titles.append(Title)
        Released.append(Date)
        IMDB_Ratings.append(IMDB)
        Meta_Ratings.append(Meta)
        Rating_Submissions.append(Rating)
        Revenue.append(Gross_Total)

        print("Movie #{}: {} added to array.".format(Movie_Num, Title))

        # Increments the total movie number completed.
        Movie_Num += 1

    # Monitors scrape time.
    Run_Time = time.time() - Initial_Scrape
    Request_Seconds = Run_Time / Total_Requests
    print("{} Requests per second.".format(Request_Seconds))

    # Increments the total requests number completed.
    Total_Requests += 1

    # Sleep for random time between 1 and 30 seconds.
    print("\n___________________________________________________")
    rand_sec = randint(1, 30)
    time.sleep(rand_sec)
    print("\tSleeping {} seconds.".format(rand_sec))
    print("\tOn page: {}".format(page))
    print("___________________________________________________\n")
    clear_output(wait=True)

# Stores the scraped data to a csv.
data_frame = pd.DataFrame({
    "Movie Title": Titles,
    "Year Released": Released,
    "IMDB Rating": IMDB_Ratings,
    "Metacrtic Rating": Meta_Ratings,
    "Total Ratings": Rating_Submissions,
    "Revenue": Revenue
})

data_frame.to_csv("IMDB_Meta_Ratings.csv", index=False)
print("Scraping complete.")
