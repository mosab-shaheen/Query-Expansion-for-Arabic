# Query Expansion for Arabic

# Abstract:
Queries are usually short. In addition to that, term mismatch can happen because the words in
the document and the query may differ. To solve this issues, we need to capture the semantics
of the words rather than the exact words. Neural Language Models captures the semantics
using the contextual information. They represent the words by vectors or embeddings.

# Introduction:
Arabic language is very ambiguous [1]. A word has 19 meanings on average [1]. In case of
word mismatch we can use word embeddings, which capture the contextual information, to
find similar words that have similar contexts and thus solving the ambiguity. To find similar
words of query words there are two ways. One is finding similar words in the vocabulary of
the corpus, another one is finding similar words in the document.

# Appproach:
Traditional IR models are divided into two parts:
A(w, q): related to the weight of word w in query q
B(w, d, C), related to the weight of word w in document (d) and collection (C).
They mostly differ on B. For example the Okapi BM25 model can be written as follows:

<img width="461" height="238" alt="image" src="https://github.com/user-attachments/assets/4359b54f-0996-4110-bd5b-a45094c4f42d" />
<img width="324" height="64" alt="image" src="https://github.com/user-attachments/assets/833df2ee-4fe6-4fa2-a5b9-f14747bef88a" />

To find the similar words, you can use the similarity with the words in the corpus Sd (w) or
the words in the ducument Sc (w), during the scoring as shown below:

<img width="384" height="87" alt="image" src="https://github.com/user-attachments/assets/328b266f-fe77-4391-b41b-6c2c69034725" />


The above is selecting the top k similar words based on the threshold θs.
Similar words are integrated into the models using the normalized cosine similarity as the
weight for these words.

<img width="468" height="210" alt="image" src="https://github.com/user-attachments/assets/494d18a9-1ba5-4e09-889b-3051db89ad4f" />

RSV score becomes:

<img width="550" height="72" alt="image" src="https://github.com/user-attachments/assets/7d6f65a4-e73a-4772-adf2-3d3b2d7b8b54" />

# Implemntation Details:
Programming Language: Python
Dataset: Arabic
Model: BM25
Evaluation: MAP
Word Embedding: Continuous Bag Of Words (CBOW)

Arabic is a highly morphological language. If we do not apply stemming on Arabic, then the
top similar words, according to word embeddings, will give the morphological variants of the
word.
I used the “cltk” library (please refer: http://docs.cltk.org/en/latest/arabic.html), for doing the
following:


<img width="587" height="372" alt="image" src="https://github.com/user-attachments/assets/c4c83c9d-46a0-4c1d-bc0a-1d5c2bb21451" />

All these data structures are stored in persistent storage for later usage. After preprocessing
and loading the data I calculated the parameters of BM25. The standard BM25 form is as
follows:

<img width="452" height="251" alt="image" src="https://github.com/user-attachments/assets/2a2b475c-1696-4354-83c9-ec46e9d43306" />

The best parameters for the pre-mentioned Arabic collection is: k1=1.0, b=0.6, and k3=0.0
which gives MAP=0.5446342933702899 as the figure below shows:

<img width="607" height="297" alt="image" src="https://github.com/user-attachments/assets/e17f8a89-43d7-46ce-b0c4-a9ec2a8d653a" />

According to the results k3 did not affect the performance at all and that’s because it is related
to the term frequency in the query which is 1 almost always in the used dataset.
Now let’s focus on k1, and b. In the diagram we see six hill-shape curves, which are
corresponding to the six distinct values of k1. We notice that those hills are almost the same
height which means that k1 is playing a very minor role when calculating BM25 over the
used dataset. The only parameter that makes the difference is b, we notice that the points inone hill are almost equally spaced from the center. That means the center of the hill which is
the center of b range [0.1,1.0] is the best value, which is 0.6 here. This is reasonable because
the documents in the corpus vary in their length, where you find a document of 1 line and
another document of 20 lines.



# Implementing the extension:
After that I trained the extended proposed model, which uses the word embeddings:

<img width="550" height="72" alt="image" src="https://github.com/user-attachments/assets/1054b345-7de8-42e5-aaab-fdf93613027e" />

using the similar words in the collection:

<img width="384" height="87" alt="image" src="https://github.com/user-attachments/assets/60e6aa05-e547-423a-8bb8-e6a8f2b4c1db" />


where I used K=10, =0.5, and the weights of the similar words are as follows: ϴ


<img width="468" height="210" alt="image" src="https://github.com/user-attachments/assets/221a9271-eaf3-4628-b00f-b73abebfa127" />

The effect of parameter λc is shown in the figure below:


<img width="602" height="302" alt="image" src="https://github.com/user-attachments/assets/ed1527f7-b202-46bf-9451-402d0a3a2c13" />

The best value for parameter λc in the range [0.1,2] is 0.4 which gives
MAP=0.5518197799015225.


# Additions to the Extension:
I included the POS tags of the words so that the proposed model selects similar words of a
query term only if it has the same POS tag. The POS tagger used is the Stanford POS tagger,
which is trained on Arabic dataset.
The data structures are modified accordingly:

<img width="572" height="96" alt="image" src="https://github.com/user-attachments/assets/e7210bc3-9e69-45e7-8d3e-4e318331a5f9" />


However adding the POS tags did not improve the results and gives
MAP=0.42288111162415487 for the best parameter value λc=2.0.
The reason is that the POS tagger is not that too much accurate. In addition to that, the
document frequency of a term dft and the term frequency of a term in a document tfd depends
now on the tag. In other words, if word1 has two different tags tag1, and tag2 then
word1:tag1 and word1:tag2 are considered two different words with different document
frequencies and term frequencies. In this case if the POS tagger gave wrong tag to the word,
then its document frequency and term frequency goes extremely wrong.

# Conclusion:
In this project I have configured BM25 model to take the best parameter values for the Arabic
collection. Then I extended BM25 using the method proposed in the research article [1], which gave
better results. I included the POS tags in the extended model, but it did not give better results
because of the mistakes done by the POS tagger which extremely affected the document
frequencies and the term frequencies of a term.


# References:
[1] El Mahdaouy, Abdelkader, Saïd Ouatik El Alaoui, and Eric Gaussier. "Improving Arabic
information retrieval using word embedding similarities." International Journal of Speech
Technology (2018): 1-16










