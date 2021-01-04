---
layout: post
category: develop
comments_id: 2
---
# [ : Using Graph Learning to Power Recommendations](https://eng.uber.com/uber-eats-graph-learning/)  
Uber Engineering Blog  
Ankit Jain, Isaac Liu, Ankur Sarda, and Piero Molino  
December 4, 2019

아래 글은 위 글의 비공식 약식 번역본 입니다.

*[previous post 1](https://yongqyu.github.io/gnn-food-discovery-with-uber-eats-1.html)*  
*[previous post 2](https://yongqyu.github.io/gnn-food-discovery-with-uber-eats-2.html)*

-----------------------------------------------------

### Data and training pipeline

Once we determined the positive impact of graph learning on our recommendation system, we built a scalable data pipeline to both train models and obtain predictions in a real-time production environment.

We train separate models for each city, as their graphs are only loosely connected.

In order to do this, we used anonymized, aggregated order data from the past several months available and designed a four-step data pipeline to transform the data into the networkx graph format that is required to train our models. The pipeline also extracts aggregated features not directly available in the raw order data, like the total number of times users ordered dishes, which determines the weight of the graph’s edges.

Additionally, the pipeline is also capable of creating graphs for older time frames, which can be used for offline analysis. The overall pipeline is depicted in Figure 6, below:


Figure 6: We built a data pipeline (top row) and training pipeline (bottom row) that helps us train our Uber Eats recommendation system using GNN embeddings for improved in-app dish and restaurant suggestions.


In the first step of the pipeline, multiple jobs pull data from Apache Hive tables, ingesting it into HDFS as Parquet files containing nodes and edges information respectively. Each node and edge has properties that are versioned by timestamp, which is needed for constructing back-dated graphs.

In the second step, we retain the most recent properties of each node and edge given a specific date and store them in HDFS using Cypher format. When training production models, the specified date is the current one, but the process is the same also if past dates are specified for obtaining back-dated graphs.

The third step involves using the Cypher query language in an Apache Spark execution engine to produce multiple graphs partitioned by city.

Finally, in the fourth step we convert the city graphs into the networkx graph format, which is consumed during the model training and embedding generation process, which are implemented as TensorFlow processes and executed on GPUs.

The generated embeddings are stored in a lookup table from which they can be retrieved by the ranking model when the app is opened and a request for suggestions is issued.

### Visualizing learned embeddings

In order to provide an example capable of characterizing what is learned by our graph representation learning algorithm, we show how the representation of a hypothetical user changes over time.

Assuming we have a new user on Uber Eats who ordered a Chicken Tandoori and a Vegetable Biryani (both Indian dishes), we obtain a representation for such user at this moment in time.

The same user later orders a few other dishes, including: Half Pizza, Cobb Salad, Half Dozen Donuts, Ma Po Tofu ( a Chinese dish), Chicken Tikka Masala and Garlic Naan (three Indian dishes). We obtain a representation of the user after these additional orders and we compute the distance of those two representations with respect to the most popular dishes from different cuisine types and display it in Figure 7 below using the explicit axes technique introduced in Parallax: Visualizing and Understanding the Semantics of Embedding Spaces via Algebraic Formulae.


Figure 7: We compared the representation of a hypothetical user before and after ordering dishes and compared them to popular dishes from different cuisines. The four plots highlight dishes belonging to four different subsets of cuisines. The x axis measures how much a dish is similar to the user representation before ordering additional dishes, while the y-axis measures how a dish is similar to the the user representation after ordering additional dishes.


In the bottom left section of Figure 7, clear patterns emerge. The first pattern is highlighted in the green box in the bottom right: the dishes closest to the user representation before the additional orders are almost all Indian dishes (green dots) as expected given the fact that the initial orders were both of Indian food, but also some Chinese dishes end up ranked high on the x-axis, suggesting a second order correlation between these cuisine types (i.e., users who ordered many Indian dishes also ordered Chinese ones). Chinese dishes also rank pretty high on the y-axis, suggesting that ordering Ma Po Tofu influenced the model to suggest more Chinese dishes.

In the top right section of Figure 7, a second pattern is highlighted in the orange box: American, Italian, Thai, and Korean dishes are selected, showing how they are much closer to the user representation after the user ordered additional dishes. This is due to both ordering Pizza, Doughnuts, and the Cobb Salad, but also due to second order effects from the increase of Chinese suggestions, as users ordering Chinese dishes are also more likely to order Thai and Korean.

Finally, in the top left section of the image, a third pattern is highlighted in the blue box: all the cuisines that are not among the top three closest to both user representations ended up increasing their similarity substantially after their subsequent orders, which suggests that the model learned that this specific user might like for new cuisine suggestions to be surfaced.

### Future directions

As discussed, graph learning is not just a compelling research direction, but is already a compelling option for recommendation systems deployed at scale.

While graph learning has led to significant improvements in recommendation quality and relevancy, we still have more work to do to enhance our deployed system. In particular, we are exploring ways to merge our dish and restaurant recommendation tasks, which are currently separate, because we believe they could reinforce each other. Over time, we plan to move from two bipartite graphs to one single graph that contains nodes of all the entities. This will require additional work on the loss and aggregation function to work properly, but we believe it will provide additional information to both tasks leveraging common information.

Another limitation we want to tackle is the problem of recommending reasonable items to users even in situations with data scarcity, such as in cities that are new to the Uber Eats platform. We are conducting research in this direction through the use of meta graph learning [5] with encouraging results.

Interested in how Uber leverages AI to personalize order suggestions on Uber Eats? Learn more about our Uber Eats recommendation system through our Food Discovery with Uber Eats series:

Food Discovery with Uber Eats: Recommending for the Marketplace
Food Discovery with Uber Eats: Building a Query Understanding Language


---
{: data-content="footnotes"}

[^4]: William L. Hamilton, Rex Ying and Jure Leskovec: Inductive Representation Learning on Large Graphs. NIPS 2017
