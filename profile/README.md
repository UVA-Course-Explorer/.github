## UVA Course Explorer
This README is an overview of UVA Course Explorer, a search engine and catalog for courses at the University of Virginia.

# Search
Our Search page is powerd by OpenAI's `text-embedding-3-small` model. Given a piece of text, this model outputs a 1536-dimensional vector that encodes the semantic meaning of the text.

We have run the descriptions of all the courses at UVA thorugh this model and store the outputted vector for each course.

When a user inputs a query, we use the same model to generate a vector representation of the query text.

We then compute the cosine similarity between the query vector and the vectors of all the courses. The cosine similarity is a measure of the similarity between two vectors and is closely related to the dot product operation. Below is the formula for the cosine similarity, where A and B are vectors:

$\Large{cos(\theta) = \frac{A \cdot B}{|A| |B|}}$


Cosine similarities will be in the range [-1, 1]. A high cosine similarity implies that two vectors are similar, and that the texts corresponding to the two vectors have similar meaning.

 The similarity score reported on our site is just the cosine similarity multiplied by 100.

To generate results for a query, we return the courses whose vectors had the 10 highest cosine similarities with the query vector.


The `text-embedding-3-small` model is the encoder portion of a transformer that was trained to output similar vectors for pieces of text that appear close to each other on the internet (an approach generally referred to as contrastive learning). For more details about the model, please refer to the related blog posts [1](https://openai.com/blog/introducing-text-and-code-embeddings) [2](https://openai.com/blog/new-and-improved-embedding-model) [3](https://openai.com/index/new-embedding-models-and-api-updates/) and [paper](https://cdn.openai.com/papers/Text_and_Code_Embeddings_by_Contrastive_Pre_Training.pdf).



We also allow for users to directly search for a class using the catalog abbreviation and number (e.g. CS 1110). You can also search for a special topics class by appending the SIS session number after the catalog number (e.g. CS 4501.001).



# Catalog
We repeatedly fetch the latest course information from SIS with a GitHub Action that is run hourly.

The action makes asynchronous requests to the following SIS API endpoint:

```https://sisuva.admin.virginia.edu/psc/ihprd/UVSS/SA/s/WEBLIB_HCX_CM.H_CLASS_SEARCH.FieldFormula.IScript_ClassSearch?institution=UVA01&term={strm}&page={page}```

where `strm` is a number (e.g. 1238 for the Fall 2023 semester) representing the semester of interest and `page` is for result pagination.

After fetching the latest data for all courses in from SIS, the action saves all the information as JSON files, which are then accessed from our client-side code. End-to-end, our action takes about 1.5 mintues per run. The data-fetching code and JSON storage can be found at [UVA-Course-Explorer/course-data](https://github.com/UVA-Course-Explorer/course-data). 


# Infrastructure
The front-end code([UVA-Course-Explorer/client-app](https://github.com/UVA-Course-Explorer/client-app)) that powers both the search and catalog pages on our site is deployed using Netlify.

Our backend code that powers the search page ([UVA-Course-Explorer/server-app](https://github.com/UVA-Course-Explorer/server-app)) is deployed on [Fly.io](https://fly.io/) and is running on two servers in a datacenter in Ashburn, Virginia. The servers shut down when they have been inactive for a while. As a result, you may observe that your first search is slow (because the server needs to boot up), but subsequent searches are faster.

As mentioned in the section above, the code used to fetch data from SIS and update our JSON store is deployed as a GitHub action and can be found at [UVA-Course-Explorer/course-data](https://github.com/UVA-Course-Explorer/course-data).

You can find a spreadsheet outlining all the costs to host the app [here](https://docs.google.com/spreadsheets/d/1I0adoa030sOMjLiRc6OIMzG5uPjK9DYZ31nS11M2o3U/edit?usp=sharing). 

# Limitations
## Bias
The OpenAI embedding model we use was trained using large quantities of internet text. As a result, the model can encode biases found in internet text in the embeddings it generates.

To mitigate this issue, we pass all searches through OpenAI's Moderation API, which flags explicitly innapropriate search queries. However, there is still a chance that a non-explicit query yields biased course results.



## React Ineffeciencies
Both of us are still learning the intricacies of React. As a result, we are certain that we can optimize our front-end code to be more performant and have a smaller footprint on users' devices.


## Low Quality Search Results Due to Classes with Short Descriptions
We've found that classes with short descriptions pop up in search results quite often, even if they are not closely related to the search query.


## Lack of user authentication
We currently do not require user authentication/login to use our site. We do not want to require users to log in to use our application. We do have CORS set up to prevent other sites from hitting our API, but it is still quite easy to spam our server with requests (from either the website, or by directly accessing our publicly-exposed API).


If you have ideas to address any of the above limitations, please let us know through our [feedback form](https://forms.gle/Jq2di8Zji4tDNKZF8) or by emailing us at uvacourseexplorer@gmail.com


# Planned Future Features
- [ ] Automate search index population. Currently, we need to manually run a script to populate our search index once a semester. As a result, information on the search page may be stale if classes have been added/removed from SIS. We are planning to automate this with a GitHub action.
- [ ] Have waitlist numbers in the catalog
- [ ] Attendance/Waitlist Graphs
- [ ] Dropdown on all catalog pages to Select Semester/Term (instead of a separate page)


# HooHacks 2023
We built an initial prototype of this project for HooHacks 2023. You can view our Devpost [here](https://devpost.com/software/uva-course-explorer) and our hackathon repository [here](https://github.com/sidlakkoju/UVA-Course-Explorer).


# Related Works
We drew inspiration from several projects when building UVA Course Explorer.


- [awesome-movies.life](https://t.co/l6uyNmrXmu): Andrej Karpathy's movie recommendation site and accompanying [tweet](https://twitter.com/karpathy/status/1647374645316968449?lang=en) convinced us to use NumPy for storing our embedding matrix and performing the computations required for each search.
- [searchclasses.org](https://www.searchclasses.org/): This semantic search engine built for Stanford courses showed us evidence that semantic search for university courses could yield relevant results.
- [Lou's List](https://louslist.org/): We based our catalog heavily on Lou's List, the OG catalog for UVA courses maintained by Professor Lou Bloomfield since 2009.


We also found the [site](https://f22.cs3240.org/) for UVA's CS 3240 (Advanced Software Development) class to be helpful when learning about the SIS API during HooHacks 2023. (It looks like info about the SIS API has been removed since we last checked).



# Acknowledgements
We would like to thank DALLE-2 for creating our logo.

We would like to thank ChatGPT and GitHub Copilot for their contributions to this project. Both of you have been amazing at helping us navigate the new tools and frameworks we used. We are very excited (and a little scared) to see what you both become. 

We would also like to thank our families and friends for giving us tons of constructive feedback during the development process and also for being the best beta testers we could ask for. Thank you :)) 

And thank you for reading all the way through! Please let us know if you have any feedback or suggestions through this [form](https://forms.gle/Jq2di8Zji4tDNKZF8), or email us at uvacourseexplorer@gmail.com. We would love to hear from you! 

We hope you find this tool useful 😊. - Siddharth Lakkoju and Saahith Janapati
