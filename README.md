# Live Project

## Introduction

My last two weeks at The Tech Academy were spent working on a web scraping application using Python, Django and BeautifulSoup. Our goal was to develop a social media platform with a user dashboard for various widgets. It was good to jump in at an early stage and see how a team of developers can enhance and rework an existing code base together.  This project used Azure Devops to facilitate version control and feature updates from user stories. We had daily standups for managing the sprint. I had the chance to contribute several [front end](#front-end-stories) and [back end](#back-end-stories) features that I am proud of. 

# Back End Stories

## Searchable Craigslist Scraper

I was tasked with creating an app that searched the local craigslist "for sale" section. BeautifulSoup and Requests made this possible after taking in user input from a simple form. The following is from my view.
    
    def craigslistsearch(request):
    # Define website URL with empty spaces for input variables
    url = "https://portland.craigslist.org/search/sss?query={}&search_distance={}&postal={}"

    # Pulls up blank SearchForm when user visits page
    if request.method == 'GET':
        form = SearchForm(initial = {'keyword': 'ex: Trampoline'})
        context = {'form' : form}
        return render(request, 'CraigslistApp/craigslist.html', context)

    # Once form is filled out and "Search" button is pressed, form data is saved
    elif request.method == 'POST':
        form = SearchForm(request.POST)
        form.save()
    
        search = Search.objects.filter().order_by('-id')[0] # Gets latest data from Search model to use in building new URL

        response = requests.get(url.format(search.keyword, search.radius, search.postal)) # Formats Search model data into new URL and gets HTML info

        data = response.text # Returns the text from Request.get(url)

        soup = BeautifulSoup(data, 'lxml')

        results = soup.find_all('li', class_='result-row') # As of April 2019, Craigslist gives each unique search entry class of 'result-row'. This collects them from the HTML using Beautiful Soup

        links = [] # Empty list to store our search results as the useful text and href link

        # Loop over each result-row item to pull out our useful info 
        for result in results:
            title = result.find('a', class_= 'result-title hdrlnk')
            title = title.get_text()
            price = result.find('span', class_='result-price')
            price = price.get_text()
            boro = result.find('span', class_= 'result-hood')
            # Sometimes the neighborhood  or 'result-hood' class is empty so this eliminates the error if data-type is None
            if boro is not None:
                boro = boro.get_text()
            else:
                boro = str()

            link = title + " " + price + " " + boro # Combines all useful text into a single string
            href = result.find('a', class_= 'result-title hdrlnk')
            href = href["href"] # Pull out the actual link text from the href

            links.append([link,href]) # Populates our empty links list as another list to keep the link info and href paired
        context = {'links' : links, 'form' : form} # Combines the links list and SearchForm to render back to our HTML template 
        return render(request, 'CraigslistApp/craigslist.html', context)

## PodcastsApp

One user wanted to see the top five podcasts on Stitcher. I added the following to a view using a headless browser in Selenium to grab the dynamic content from the Stitcher top 100 site.  

    def get_top5(request):
    url = 'https://www.stitcher.com/stitcher-list/all-podcasts-top-shows'

    #Opens headless browser window with Selenium and PhantomJS to load Stitcher website with dynamic content

    options = Options()
    options.add_argument('-headless')
    driver = webdriver.Firefox(options = options)
    driver.get(url)
    html = driver.execute_script("return document.documentElement.outerHTML")
    sel_soup = BeautifulSoup(html, 'html.parser')

    #Finds the Stitcher Top 100 list to search within
    top100_data = []
    table = sel_soup.find('table', attrs={'id':'stitcher-list'})
    table_body = table.find('tbody')
    rows = table_body.find_all('tr')

    #Iterates through each row in table pulling out the anchor tag text which includes the show title
    for row in rows:
        title = row.find('a').get_text() 
        top100_data.append(title)

    #Limits the variable to the first 5 entries
    top5 = top100_data[0:5]
    print (top5)

    context = {'top5' : top5}

    return render(request, 'podcast/podcast.html', context)

# Front End Stories

## Navigation

There was an issue with the navigation links not appearing on each page with the user profile image in the top corner. Fixing this involved making sure each html template placeholder was pointing to the nav.html file and modifying the nav.html to pull the image from our user db model. 

## Dashboard Layout

I also configured the dashboard replacing various images representing each app with font awesome svg icons for a uniform feel.
### Before
![screen shot](/images/layout_before.jpg)

### After
![screen shot](/images/layout_after.jpg)



# Takeaways

* Good project management software is key to team workflow and communication. 
* Coding can seem personal as if YOU have to figure everything out on your own but making contributions and studying your team's code keeps the wheels turning.
* Sharing ideas and code logic in Standups can help someone with a similar problem get unstuck.


[Back to top](#live-project)
