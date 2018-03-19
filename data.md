Primary indicators
1. userid
2. item (borrow)

Secondary indicators
user1 Trade IBM
user2 view AMAZON
user1 location LONDON
user1 prefer-category Equity




CCO:

LLR:
https://github.com/apache/mahout/blob/master/math-scala/src/main/scala/org/apache/mahout/math/cf/SimilarityAnalysis.scala#L300


-------------------------
Item properties are metadata about the item. For ecom they might be what category of product they are. Book, clothing, mens-clothing, electronics, tvs, etc. Properties are usually categories that you give items. When you make a recommendation and the users have navigated to a part of the site showing TVs, you might want to boost TV recommendations. To do this you ask for user based recommendations and set the bias to a positive > 1 number depending on how much you want to boost TVs.

For items with no events, you can set a user-defined ranking and use boosts on properties to get recommendations that have no events data. This is not illustrated in the examples because it is not what recommenders are for. 


I'm not quite sure what it means. So for example, Archery Class is in the "Training" type and "Outdoor" topic, then every time user purchase or view the Archery Class, should I send below to the event server? : 

Set the properties of the item to the corresponding value
	{
		"event":"$set",
		"entityType":"item",
		"entityId":"archery class",
		"properties": {
			"topic":["outdoor"],
			"type":["class"]
	}
		"eventTime" : "2016-10-05T21:02:49.228Z"
	}

Send then preference event to the event server every time user view or purchase a certain type and topic of activities:

{
	"event" : "topic-preference",
	"entityType" : "user",
	"entityId" : "1243617",
	"targetEntityType" : "item",
	"targetEntityId" : "outdoor",
	"properties" : {},
	"eventTime" : "2015-10-05T21:02:49.228Z"
}
{
	"event" : "type-preference",
	"entityType" : "user",
	"entityId" : "1243617",
	"targetEntityType" : "item",
	"targetEntityId" : "class",
	"properties" : {},
	"eventTime" : "2015-10-05T21:02:49.228Z"
}

Then, with a second matrix, which can be P again to make PtP or a matrix for a secondary indicator (say L for likes) to make PtL, we take a row from Pt (item A) and a column from the second matrix (either P or L, in this example) (item B) and we calculate the table that Ted Dunning explains on his webpage: the number of coocurrences that item A AND B have been purchased (or purchased AND liked), the number of times that item A OR B have been purchased (or purchased OR liked), and the number of times that neither item A nor B have been purchased (or purchased or liked). With this counts we calculate LLR following the formulas that Ted Dunning provides and the resulting LLR is what goes into the AB element in matrix PtP or PtL. Correct?  

keep top 50 LLR score in ElasticSearch 
LLR 3 points: 1. use data widely 2 reduce noise 3.similarity 4. weight.

Mahout builds the model by doing matrix multiplication (PtP) then calculating the LLR score for every non-zero value. We then keep the top K or use a threshold to decide whether to keep of not (both are supported in the UR). LLR is a metric for seeing how likely 2 events in a large group are correlated. Therefore LLR is only used to remove weak data from the model.

So Mahout builds the model then it is put into Elasticsearch which is used as a KNN (K-nearest Neighbors) engine. The LLR score is not put into the model only an indicator that the item survived the LLR test.

The KNN is applied using the user’s history as the query and finding items the most closely match it. Since PtP will have items in rows and the row will have correlating items, this “search” methods work quite well to find items that had very similar items purchased with it as are in the user’s history.

=============================== that is the simple explanation ========================================
Item-based recs take the model items (correlated items by the LLR test) as the query and the results are the most similar items—the items with most similar correlating items.

The model is items in rows and items in columns if you are only using one event. PtP. If you think it through, it is all purchased items in as the row key and other items purchased along with the row key. LLR filters out the weakly correlating non-zero values (0 mean no evidence of correlation anyway). If we didn’t do this it would be purely a “Cooccurrence” recommender, one of the first useful ones. But filtering based on cooccurrence strength (PtP values without LLR applied to them) produces much worse results than using LLR to filter for most highly correlated cooccurrences. You get a similar effect with Matrix Factorization but you can only use one type of event for various reasons.

Since LLR is a probabilistic metric that only looks at counts, it can be applied equally well to PtV (purchase, view), PtS (purchase, search terms), PtC (purchase, category-preferences). We did an experiment using Mean Average Precision for the UR using video “Likes” vs “Likes” and “Dislikes” so LtL vs. LtL and LtD scraped from rottentomatoes.com reviews and got a 20% lift in the MAP@k score by including data for “Dislikes”. https://developer.ibm.com/dwblog/2017/mahout-spark-correlated-cross-occurences/

So the benefit and use of LLR is to filter weak data from the model and allow us to see if dislikes, and other events, correlate with likes. Adding this type of data, that is usually thrown away is one the the most powerful reasons to use the algorithm—BTW the algorithm is called Correlated Cross-Occurrence (CCO).

The benefit of using Lucene (at the heart of Elasticsearch) to do the KNN query is that is it fast, taking the user’s realtime events into the query but also because it is is trivial to add all sorts or business rules. like give me recs based on user events but only ones from a certain category, of give me recs but only ones tagged as “in-stock” in fact the business rules can have inclusion rules, exclusion rules, and be mixed with ANDs and ORs.

-----------------------------
Only PtP or more correctly llrThresholded(PtP) is done by SimilarityAnalysis.cooccurrencesIDSs. The vector multiply is a vector of dot products of all rows and the user’s history (PtP)h_p. Returning results is the process of taking the highest valued items. This is actually called K-Nearest Neighbors since the dot products give the cosine of the angle between the high dimensionality vectors (they are # of all products long with many 0 values). This part of the Algorithm is exactly what Lucene does and so in the UR  llrThresholded(PtP) is stored in Elasticsearch and h_p is the query, returning the KNN items, so items that have had similar P behavior to the User's

cosine and TF-IDF are built in to Lucene and so happens in the indexing of the model and the query described above

The models stored are  llrThresholded(PtP),  llrThresholded(PtV), … They are in Elasticsearch so the result of applying the realtime user behavior to the model can be served in realtime. It also makes it simple to apply business rules to the query, like limiting the results to a certain category or to only “in-stock” items


