# Model Proposal for Adaptive Search Engines

Jason Whitfield

* Course ID: CMPLXSYS 530,
* Course Title: Computer Modeling of Complex Systems
* Term: Winter, 2019



&nbsp; 

### Goal 
*****
 
For my project, I was thinking of modeling the interactions of users with an Internet search engine. I hope to find out how data collected about the users can be used in page ranking strategies and examine how that affects the relevance of results to agent’s queries.

&nbsp;  
### Justification
****
The Internet is composed of an incredibly large number of web pages, but search engines are
able to find relevant results based on page content and popularity. So, order appears out of a complex
system based on the actions of agents. An agent-based modeling approach would work especially well, as each user of the search engine is an agent that interacts with the environment of the websites and engine results.

&nbsp; 
### Main Micro-level Processes and Macro-level Dynamics of Interest
****

I'm interested in exploring how the search engine can learn from the individual actions of each user in order to provide more relevant search results. Each user takes very simple actions. They just query the search engine, choose a web page, read it, and repeat. However, as the search engine records the user actions, it will provide more and more relevant results to queries. At first, the page orderings will be totally random and chaotic, but over time, they will become ordered and relevant. Interestingly, the agents will never interact with each other, but will benefit from the interactions of previous agents with the environment.

&nbsp; 


## Model Outline
****
&nbsp; 
### 1) Environment
The environment in my model consists of the web pages, the user data collected by the search engine, and the search engine itself. To represent information contained in web pages, I will use one-dimensional vectors of random integers. Since the information in a web page is usually centered on a certain topic, each page will have an integer topic. The page data will consist of integers normally distributed around this topic. Also, web pages are listed in search engine results by their titles, which are related to the content of the page. So, the pages will also have a title consisting of integers randomly sampled from the information within the page.

For the search engine to learn from the users, it must collect data about their actions. So, I will initially define three pieces of data collected by the search engine. The first is the number of users that have visited a given web page. The second is the number of users that stopped searching after visiting the page. The third is the amount of information on the page that was consumed by users before they left. This page data will be stored along with the user's query, since the data will be more useful for similar queries in the future.

The search engine will hold a list of web pages and the data collected from user actions. When a user makes a query, it will order this list using the data and some ranking strategy. Then, it will observe the user as they traverse the list.

Environment-Owned Variables

_Web Pages_
* topic
* page_information (vector of info integers)
* page_title (sampled from page_information)

_Web Page Action Data_
* visits
* times_accepted
* total_info_consumed

_Search Engine_
* web_pages
* web_page_action_data

Environment-Owned Procedures

_Search Engine_
* order_pages(query)
* record_actions(user)


```python
class WebPage:

    def __init__(self, max_info_int, title_length, page_length):
        self.topic = random.randrange(max_info_int)
        # TODO: what standard deviation to use?
        self.data = [round(random.normalvariate(self.topic, SIGMA))
                     for _ in range(page_length)]
        self.title = random.sample(self.data, title_length)


class WebPageData:

    def __init__(self):
        self.visits = 0
        self.accepts = 0
        self.info_read = 0


class SearchEngine:

    def __init__(self, num_pages, max_info_int, title_length, page_length):
        self.pages = [WebPage(max_info_int, title_length, page_length)
                      for _ in range(num_pages)]
        # Maps queries to web page action data.
        # Initially empty. Record user actions as they visit pages.
        # TODO: How to get data for similar queries.
        self.page_data = {}

```

&nbsp; 

### 2) Agents
 
The agents of the system are the users of the search engine. Each user will have some topic that they want to research. Like the web pages, this will be represented by a integer topic and a one-dimensional information vector of integers normally distributed around the topic. They will then query the search engine about the topic and examine the resulting list of web pages. They will choose from the web pages based on a given page's placement in the list and its title's relevance to the information the user is seeking. At each page, the user will read the page information and search for the info that they are seeking. If not enough relevant info is found, the user may leave the site before reading all of the information. Any info that is found is removed from the search vector, and the user updates their query to exclude that info. This continues until the user is satisfied with the amount of information that they have found.

Agent-Owned Variables
* topic
* sought_information (vector of info integers)
* required_info_percent

Agent-Owned Procedures
* generate_query()
* choose_webpage(pages)
* read_webpage(page)


```python
class User:

    def __init__(self, max_info_int, info_length, satisfy_pct):
        self.topic = random.randrange(max_info_int)
        # TODO: what standard deviation to use?
        self.info = [round(random.normalvariate(self.topic, SIGMA))
                     for _ in range(info_length)]
        self.satisfy_pct = satisfy_pct
```

&nbsp; 

### 3) Action and Interaction 
 
**_Interaction Topology_**

In the model, agents do not directly interact. Instead, they interact with the environment of the search engine. The data collected about agents affects subsequent search results, so the actions of each agent will indirectly affect the actions of all future agents. The agents can interact with any web page, but as with real search engines, higher-ranked web pages are much more likely to be visited.
 
**_Action Sequence_**

1. Set of web pages randomly generated, each with random topic and information vector.
2. New user generated with random topic and information vector.
3. User creates query based on sought information.
4. Search engine returns list of web pages based on query.
5. User chooses web page from list based on list placement and title relevance.
6. User reads web page until it gives up or reaches end of page.
7. User removes found information from their sought info vector.
8. Search engine records data about actions of user.
9. Repeat steps 3 through 7 until enough info has been found to satisfy user.
10. Repeat from step 2.

&nbsp; 
### 4) Model Parameters and Initialization

There are a few global parameters I will be applying in my model. The first is the total number of web pages indexed by the search engine. The second is the range of possible information that can be found on web pages, i.e. the possible range of the integers in the info vectors. The third is the lengths of the web page titles and page information. Longer titles could give a better idea of the content of the page. The last is the standard deviations of the normal distributions used by pages and users. I might want to vary this by page and user to see how it affects the relevance of a page.

The model will be initialized by first generating the web pages. The web pages will have totally random topics, independent of each other. I want to generate a large number of web pages so that users are likely able to find most of the information they seek. Next, I will initialize the user action data to be empty. Finally, I will create the users iteratively. Each user is independent of all past and future users, so I will generate them only when they are making their query.

First, the model is initialized as described above. Then, once a user has been generated, they create a query based on the information that they seek. They supply that query to the search engine, which uses it to order the web pages. This ordering is based on the data collected by the search engine about how users with similar queries interacted with the web pages. So, for example. it may order them by total the number of clicks each page got with similar queries. This ordering strategy will weight all of the data collected from users, and these weights will be adjustable to examine different kinds of strategies. Once the user gets the ordered list of pages, they then choose one page to examine. This choice is based on the page's position in the list and the title's relevance to the query, as in real search engines, people click on the most relevant thing that they see first. Then, the user reads the page until either the end or until they decide it isn't relevant enough and they give up. After reading the page, the user notes which information it found and updates their query accordingly. Then, they make a new search and repeat the process. Eventually, the user will decide that they are satisfied and stop searching. This process will repeat for a very large number of users in order to simulate the kind of large data sets that real search engines work with.


&nbsp; 

### 5) Assessment and Outcome Measures

From this model, I hope to find out how different types of data and page ranking strategies affects the relevance of results to user queries. To measure this, I will iterate over a very large number of users and measure the total number of web pages visited and information consumed before each user is satisfied with the information they have found. I will look at these as running averages and hopefully see them trend downwards over time.

Additonally, I think it might be interesting to compare the usefulness of the different kinds of data. For example, information consumed might be more useful than page clicks. However, measuring user activity on a web page is more intrusive than measure their activity on the list of search results. So, it would be interesting to see the tradeoff between intrusiveness and effectiveness of data.

&nbsp; 

### 6) Parameter Sweep

I'm most interested in sweeping through the weights that the search engine gives each kind of data. I think that the different types of data will have very different levels of usefulness, so I'm interested in how much an optimal weighting will improve search relevance.

Also, I'm interested in sweeping through the number of web pages indexed by the search engine. I suspect that initially, more web pages will increase search relevance, but there will quickly be a point of diminishing returns. With a very large number of pages, some pages may be very relevant to a query, but they will be ignored in favor of relevant enough pages that were visited by previous agents. I would like to explore from a small number of web pages (~100) to a very large number (~1,000,000).

Finally, I'm also interested in sweeping through information value ranges. With a larger range, I expect it to generally take longer to find relevant results. What I would be interested in exploring is the interaction between the range and the number of web pages needed to increase relevancy. I expect these two values to be closely connected, so I would like to sweep through both at once.
