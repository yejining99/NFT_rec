# 1. Data Description

## 1.1. Overview

In our study, we focus on the four top-ranked ERC-721 based NFT collections, which are Bored Apes Yacht Club (BAYC), Cool Cats, Doodles, Meebits, in terms of market capitalisation as of August 2022 according to DappRadar. We collected content features using OpenSea API and transaction data from Etherscan NFT tracker adn web3.py, covering the entire period between 1st September 2021 and 10th March 2023. Our dataset consists of implicit feedback and considers purchase history as user-item interactions. We implemented a minimum threshold of five interactions for items in our user-item matrices and incorporated item and user features to enhance the predictive power. The data includes 4,575 users, 25,014 items, and 56,981 interactions across all four collections. <br>
<br>

The number of users, items and interactions for each collection used for the experiment.<br>
| collection | users | items | interactions |
|-------|------|------|-------------|
| BAYC  | 1230 | 6726 | 13737 |
| Coolcats | 1357 | 6824 | 14890 |
| Doodles | 804 | 4771 | 7250 |
| Meebits | 1184 | 6693 | 21104 |
<br>

## 1.2. Evaluating Data Density and Transaction Thresholds

We explore our dataset by evaluating its data density and transaction thresholds as determining the optimal transaction threshold is one of the critical steps in designing effective recommender systems. Below analyses were done with complete set of transactions.<br>

<br>

- **Number of transaction left after transaction threshold on users**
<div class="image-container">
    <img src="assets/user_trxns.png" alt="Plot" style="width:500px;height:300px;">
    
</div><br>
<br>

- **Number of transactions left after transaction threshold on items**
<div class="image-container">
    <img src="assets/item_trxns.png" alt="Plot" style="width:500px;height:300px;">
    
</div><br>
<br>

- **Number of users left after user threshold**
<div class="image-container">
    <img src="assets/user_threshold.png" alt="Plot" style="width:500px;height:300px;">
    
</div><br>
<br>

- **Number of items left after item threshold**
<div class="image-container">
    <img src="assets/item_threshold.png" alt="Plot" style="width:500px;height:300px;">
</div><br>
<br>   

## 1.3. Power law distribution

Power law distributions provide insights into the relative popularity and concentration of tokens in a dataset. To gain a deeper understanding of our dataset, we analyze its power law distribution and compare it to a benchmark dataset. This comparison enables us to gauge the alignment of our dataset with standard patterns and to identify any distinctive characteristics. For a fair comparison, we have selected a random sample of 14,000 interactions from the benchmark dataset, which corresponds to the average number of interactions in our dataset.
 <br>
<br>
1. Our dataset
<br>
 <img src="assets/power_law_for_four_collections.png" alt="Plot" ><br>
<br>

<!-- - Cool cats<br>
<img src="assets/coolcats.png" alt="Plot" style="width:500px;height:300px;">
<br>

- Doodles<br>
<img src="assets/doodles.png" alt="Plot" style="width:500px;height:300px;">
<br>

- Meebits<br>
<img src="assets/meebits.png" alt="Plot" style="width:500px;height:300px;">
<br> -->

2. Benchmark datasets<br>
<br>

- Amazon<br>
<img src="assets/amazon.png" alt="Plot" style="width:500px;height:300px;">
<br>

- MovieLens20M<br>
<img src="assets/movielens20m.png" alt="Plot" style="width:500px;height:300px;">
<br>

The distribution for our datasets shows a weaker power law distribution compared to a benchmark dataset, suggesting that the popularity of items in our dataset is more evenly distributed. This could potentially make the recommendation task more challenging if relying solely on the collaborative filtering method. In a context where popularity serves as a straightforward and effective indicator for recommendations, it becomes less informative in our dataset due to this more evenly distributed popularity.<br>
<br>


## 1.4. Token price movement

In order to incorporate price movement labels for multi-task learning, we have conducted an examination of the price movements for each token. We compute the label by examining the price difference between the current transaction and the one that follows. The label is binary: '1' signifies an expected price increase in the next transaction, while '0' denotes either a price decrease or situations where subsequent transactions don't exist for comparison. The columns in the table represent the following: the number of tokens for which the latest transaction price is greater than the initial transaction price, the number of tokens for which the latest transaction price is greater than the mean of the remaining transaction prices, and the number of tokens for which the mean of the second half of the transaction prices is greater than the mean of the first half of the transaction prices. In the table, each row represents a collection, and the values indicate the percentage of tokens that exhibit the respective price movements.<br>

<br>

| Token      | Latest Price > First Price | Latest Price >Avg. Price (Excluding Last) | Avg. Price (First Half) < Avg. Price (Last Half) |
|------------|-------------------------------|--------------|--------------|
| BAYC       | 59%                         | 61%       | 59% |
| Cool Cats  | 47%                         | 47%         | 50% |
| Doodles    | 58%                          | 48%        | 59% |
| Meebits    | 45%                          | 45%         |47% | 

<br> 

# 2. Side information preparation

## 2.1. Item features
### Image
We employ Convolutional Auto-Encoder(CAE) to get representations from the NFT images. We standardise all images to a shape of `128 * 128 * 3`, where 3 represents the RGB colour spectrum. CAE model consists of an encoder and a decoder, both comprising of eight fully connected layers. The encoder utilises a 33 convolutional kernel and 22 max pooling, while the decode employs a 33 convolutional kernel along with 22 upsampling.
All non-linear functions in the model are implemented using the ReLU activation function. The model is trained for 100 epochs, with the objective of minimising the Mean Squared Error (MSE) loss. The final image embeddings are obtained by employing only the encoder of the CAE, which reduces the size down to `8 * 8 * 1`. After flattening the output, we receive a 64-dimension representation for each image. This refined data is then ready for subsequent modeling stages.<br>
<br>
### Text
The text data for each item is comprised of discrete words each describing each of the visual properties like ‘Background colour’, ‘body’, ‘outfit’, and ‘hair’. Items within the same collection share the same types of visual properties whereas they tend to vary across collections. Cool cats, for example, is a collection of blue cat avatars with each artwork possessing a unique body, hat, face, and outfit, whereas Bored Ape Yacht Club is a collection of monkey avatars having a slightly different types of properties like ‘fur’. Among all, we only have considered six types of properties with the fewest missing values for each collection apart from Cool cats, for which we considered all available 5 types of properties for generating item embeddings. We then processed each descriptive word into a 300-dimension word embedding by fetching the corresponding embeddings from a pre-trained Word2Vec model. If a particular word was not found in this model, we filled it with zero padding. It's worth noting that while a majority of visual attributes were described by a single word, those composed of multiple words, like 'short red hair' for 'Hair', we used the sum of each word's embeddings instead. Each word embedding was then concatenated with other embeddings related to the same item, hence, each item's word embedding size ranged from 1500 to 1800, depending on the number of visual traits considered.<br>
<br>
### Price, Transaction
Unlike content features, obtained representations from transaction features do not
have sufficient embedding size since there is one average value per each item. So we duplicated the same value to have 64 dimensions.<br>
<br>

## 2.2. User features
### Price
The average purchase price of each user was used to represent the user’s financial capability and willingness to pay for NFTs, calculated in the same manner as the price feature of items.<br>
<br>

### Transaction frequency
The user’s average holding period for purchased tokens can provide insight into the
user’s trading behavior. This feature was calculated in the same manner as the transaction feature of items.<br>
<br>

### Transaction count
We use each wallet address’s transaction count to represent how active the user is in the
NFT market.<br>

<br>

## 2.3. Price movement label
We generate price movement label to represent the change in the price of a token between two transactions. The purpose of this feature is to classify whether the price of a token is going to increase or not in the next transaction. We calculate the price difference of each token between the current and subsequent transaction and label it 1 for upward movement, 0 for downward movement or any instances where no subsequent transactions are available for comparison. This kind of information can be valuable in predicting user behavior, as users may behave differently based on whether they anticipate a price increase. 



