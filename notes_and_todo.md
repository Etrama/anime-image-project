###############################################################################################################
A little bit of Google-fu revealed that there is something called [Jikan](https://jikan.moe/) which is an unofficial
MAL (myanimelist) API.  

There is a python wrapper for it called [Jikanpy](https://github.com/abhinavk99/jikanpy).

From this forum [post](https://myanimelist.net/forum/?topicid=2000753) on MAL, I found out that: 
`There are lots of "missing" IDs, the huge majority of them being from queued anime submissions that were denied 
(usually duplicates).

Hitting IDs at random isn't a good idea, as barely 38% of them lead to proper anime. (This proportion varies 
heavily depending on the range ; the <3,000 are over 90% present, while the 46,000-to-47,000 range is over 90% empty.)

If you have API access, I'd suggest pre-loading and caching a full list of active anime (e.g. paging through the 
/anime/ranking endpoint), and then randomizing through that list.`

###############################################################################################################
I need a way to eliminate all the duds from the list that we obtained from the API.

For instance, if I have Demon Slayer in the list, then having Demon Slayer: Kimetsu no Yaiba / Demon Slayer S2 /
Demon Slayer Season 2 is redundant.

[TO DO] This name cleaning to retain the top 250 is a real issue eh...
I think I'll use semantic similarity to eliminate the duds.
We'll check the semantic similarity of the anime name with all the other anime names and come up with a threshold.
If the similarity is above the threshold, then we'll eliminate the anime from the list.
I will need to run this on a synthetic example to determine the threshold I guess.

I need to find a way to reduce the number of anime.
    - [DONE] Remove all movies
        - Doing this as a start
        - I assume that the original anime will still be in the list since we are just removing the movie
        - I do assume that the movie is rated lower than the anime itself
    - [OVETHINKING?] Remove all 2nd season, 3rd season, etc. of anime
        - It is simplicity itself to remove all anime which have the word Season in them
        - Since only non-initial anime seasons are named like this
        - However, I do want to retain the rank, i.e. maintain relative ordering of the anime, which is complicated
        - I guess we remove the word Season and the number with Season and then we have quite the list
        - We do semantic similarity on the list using SOME ADDITIONAL LOGIC and retain the first occurence
    - [TODO] Just YOLO and remove the season, it doesn't matter
        - What's popular will still be popular
    - [TODO] observe that similar anime occur together in the list, so if we run sbert similarity over a window...
        - Run similarity with windows normally
        - Run similarity with windows shifted after sorting the list by name
###############################################################################################################
- [DONE] This name cleaning shit is taking too long. I'm just gonna excel this and manually do it for now.....
- [TODO] Process name cleaning like above for the remaining anime
- [TODO] Alternatively, I was thinking that I could simply obtain the anime names from a recommendation flowchart....
- [TODO] Lol, just get the top n anime from IMDB instead, like we did before.
###############################################################################################################
If we really think about it, this reduction of anime to its base name could be a NER thing
##############################################################################################################
Note that we can order by rank and by popularity...I'm just gonna pick rank for now. The popularity column is also a rank FYI.
##############################################################################################################
There is an [API](https://pypi.org/project/Google-Images-Search/) offered by Google, but is paid.
We could use Javascript + headless Chrome, but that takes some setup and YMMV.
Also Google warned me not to mess around with Javascript I don't understand due to fears of self-XSS.

So, I played around with some of my old code and discovered that adding the query like shown below, followed by &tbm=isch, takes us right to the Google Images page:
    https://www.google.com/search?q=naruto&tbm=isch
    http://www.google.ca/search?q=cowboy+bebop&tbm=isch
Incidentally, the images look like they're the same ones we get when we do a typical Google Images search.
Need img src tags with the class="rg_i Q4LuWd".
So I tried this with requests and BeautifulSoup and it turned out to be a bust. We only get 20ish images from the page in this manner. Looks like I have no choice but to use Javascript + headless Chrome.
##############################################################################################################
We need to scroll on the chromdriver page to get more images...
The more the number of times we scroll on the page, the more images we get. Apparently, it more like 3 instances of high scroll.
After so much trial and error, we can now obtain ~800 image tags from the page.
##############################################################################################################
[IDEA] Can we maybe screenshot and do a manga scraper? The dataset will be pretty large too.....Lots to scrape.....
Is it allowed though?
##############################################################################################################
Nested progress bars do not work in Jupyter Notebooks. At least with this:
from tqdm.auto import trange
from time import sleep

for i in trange(4, desc='1st loop'):
    for j in trange(5, desc='2nd loop'):
        for k in trange(50, desc='3rd loop', leave=False):
            sleep(0.01)
##############################################################################################################
Should add try and catch to the image scraping....
##############################################################################################################