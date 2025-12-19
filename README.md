# DSC-x-CBRE-Datathon-2024
Github for DSC x CBRE Datathon 2024 @ NYU
Created by: Yui Cao, Tony Yu, Tiffany He, Cindy Gao

## Motivation
Within rapid development of E-Commerce during pandemic, timely construction of industrial warehouses is essential. 
A precise model for estimating construction time ensures efficient planning and resource allocation, vital for meeting the demands of rapidly growing online retail. With accurate projections, businesses can strategize their supply chain management effectively, optimizing delivery schedules and customer satisfaction. 
Developing a precise and efficient model for predicting construction completion and recognizing features from satellite images can help firms monitor their construction projects, enhance project management, and provide a more reliable forecast of supply for both businesses and the real estate industry. Firms will have more current information about their construction progress through updated satellite imageries.

## Data
We use the UC_buildings dataset, which provides building-level geographic information, including longitude and latitude coordinates, as the primary spatial reference. 
We used the longitude and latitude coordinates to manually search for satellite images (on Bing & Google Maps) of the indicated construction sites. Areas of focus are manually adjusted to target active construction zones, allowing the analysis to concentrate on regions undergoing development. 


## Approach

We divided our approach into 2 parts for this Datathon: 

1. Developing CNN models for image classification
2. Providing updated estimates of completion dates for construction projects based on the classification of satellite imageries, labeled by our developed CNN models

**1. Pretrained VGG16 model**
We used a pretrained VGG16 model from pytorch and finetuned the fully connected layers to generalize for our challenge specifically - classification of construction status between 1-5. VGG16 model has been proven to be one of the most efficient model for general image classification with its depth and computational efficiency.

**2. Pretrained and modified ResNet50 model with dropout layers**
For such a small dataset (with 222 images), overfitting becomes a problem. We modified a pretrained ResNet50 model and added a dropout layer to the fully connected layer in order to reduce overfitting and increase generalizability. 
For this model, we applied 5-fold cross validation method as well as data augmentation and normalization to solve the problem of small training and validation data size, with a scheduled learning rate between [0.001, 0.0005, 0.0001]. We trained 50 epochs for each learning rate across 5 folds. Then we picked the best performing fold and learning rate, which turned out to be 4th fold and learning rate = 0.001, and trained for another 100 epochs as our final model.


## Results


## Classification


## Construction completion date estimation
