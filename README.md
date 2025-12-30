# DSC-x-CBRE-Datathon-2024
Github for DSC x CBRE Datathon 2024 @ NYU
Created by: Yui Cao, Tony Yu, Tiffany He, Cindy Gao

## Motivation
Within rapid development of E-Commerce during pandemic, timely construction of industrial warehouses is essential. 
A precise model for estimating construction time ensures efficient planning and resource allocation, vital for meeting the demands of rapidly growing online retail. With accurate projections, businesses can strategize their supply chain management effectively, optimizing delivery schedules and customer satisfaction. 
Developing a precise and efficient model for predicting construction completion and recognizing features from satellite images can help firms monitor their construction projects, enhance project management, and provide a more reliable forecast of supply for both businesses and the real estate industry. Firms will have more current information about their construction progress through updated satellite imageries.

## Data
We use the UC_buildings dataset, which provides building-level geographic information, including longitude and latitude coordinates, as the primary spatial reference. The dataset contains 222 sites under construction, with the following information: 

| Column Name                     | Column Definition                                                                 |
|---------------------------------|-----------------------------------------------------------------------------------|
| PropertyID                      | The unique identifier for the building                                             |
| VendorID                        | The vendor from which the data were derived                                        |
| Address                         | The current street address of the building                                         |
| Size_sf                         | The size of the structure in square feet                                           |
| Available_sf                    | The amount of space available for rent in the building                             |
| QuartersTrackedInDatabase       | The length of time (in quarters) the data have appeared in the database            |
| EstimatedCompletionYearQuarter  | The year and quarter the vendor estimates the building will come online            |
| YearQuarterGroundBroken         | The year and quarter when construction began                                      |
| BuildingStatusCode              | The current status of the building                                                 |
| SubPropertyType                 | The type of industrial building (e.g., logistics, R&D, manufacturing, other)      |
| buildingName                    | The name of the project while under construction                                   |
| complex                         | The name of the building park or complex this structure belongs to                 |
| Latitude                        | The latitude at which the building is located                                      |
| Longitude                       | The longitude at which the building is located                                     |
| CeilingHeight                   | The distance in feet between the floor and the ceiling joists                      |


We used the longitude and latitude coordinates to manually search for satellite images (on Bing & Google Maps) of the indicated construction sites, and label each image with corresponding construction stage (imgname_STAGENUMBER). Areas of focus are manually adjusted to target active construction zones, allowing the analysis to concentrate on regions undergoing development. 
The construction stage is defined as below: 

| Stage & Label                        | % Complete | Visual Indicators |
|------------------------------|------------|-------------------|
| Undeveloped Land (1)             | 0%         | No structures; no land clearing; vegetation or sand undisturbed |
| Ground Broken (2)               | 1–25%      | Land movement; cleared vegetation; temporary roads; construction vehicles |
| Concrete Pad (3)                | 25–45%     | Large poured concrete surface; unfinished surrounding land; possible vehicles |
| Framing Going Up (4)             | 45–85%     | Structural framing visible; pixel color variation; wall shadows; roof present; no paved parking |
| Near Completion / Completed (5) | 85–100%    | Finished appearance; roof installed; paved parking lot; vehicles or landscaping visible |

## Approach

We divided our approach into 2 parts for this Datathon: 

1. Developing CNN models for image classification
2. Providing updated estimates of completion dates for construction projects based on the classification of satellite imageries, labeled by our developed CNN models

**1. Pretrained VGG16 model**
We used a pretrained VGG16 model from pytorch and finetuned the fully connected layers to generalize for our challenge specifically - classification of construction status between 1-5. VGG16 model has been proven to be one of the most efficient model for general image classification with its depth and computational efficiency.

**2. Pretrained and modified ResNet18 model with dropout layers**
For such a small dataset (with 222 images), overfitting becomes a problem. We modified a pretrained ResNet50 model and added a dropout layer to the fully connected layer in order to reduce overfitting and increase generalizability. 
For modified ResNet18 model, we used following steps to prepare training: 
1. Transforming & normalizing datasets: to solve the issue of small dataset and improve the model's generalization.
2. Define the pretrained, modified ResNet18 model: dropout layers added in fully connected layers to avoid overfitting.
3. 5-fold cross validation: also to avoid overfitting.
4. Scheduled Learning Rate with lr = [0.001, 0.0005, 0.0001], 50 epochs each
5. Fine-tuning: freezing layers except for the fully connected layers

We picked the best performing fold and learning rate, which turned out to be 4th fold and learning rate = 0.001, and trained for another 100 epochs as our final model (ResNet_Datathon(1)).



## Results for ResNet18 Model (Final Model) 
<img width="600" height="500" alt="Screenshot 2025-12-29 at 7 54 53 PM" src="https://github.com/user-attachments/assets/3d9fd7c1-a7d0-4512-beba-3b7990cdb7f3" />

Our modified ResNet18 model with dropout and 5-fold cross-validation demonstrated strong learning performance. As shown in the plots, the learning rate of 0.001 consistently achieved the highest validation accuracy across most folds, outperforming lower learning rates. 
While validation curves occasionally fluctuate, which is likely due to the small dataset size, the overall upward trend indicates effective feature learning. After selecting the best-performing fold **(Fold 4 at LR=0.001)** and fine-tuning for an extended 100 epochs, the model further stabilized, showing good convergence without severe overfitting. This suggests that fine-tuning pretrained weights with regularization is effective for small-sample satellite imagery classification.


<img width="600" height="500" alt="Screenshot 2025-12-29 at 7 59 17 PM" src="https://github.com/user-attachments/assets/d4c89d09-36ce-45a9-ac4e-4c3936964584" />

Across all five folds, the modified ResNet model showed a steady improvement in both training and validation accuracy, confirming that the model was learning meaningful visual features.  
However, validation performance fluctuated across folds, indicating some sensitivity to the data split.  
Among the folds, **Fold 4** demonstrated the most consistent and stable performance, achieving the highest average validation accuracy of ~0.69 after fine-tuning. This suggests that the ResNet model, especially at LR=0.001, generalizes reasonably well for construction-stage classification when trained with augmentation and dropout regularization.

## Classification
We classified the construction stage based on the following:  
1. Load building + image metadata
2. Apply trained ResNet model to each image
3. Convert model outputs to discrete stage labels (1,2,3,4,5)
4. Attach predictions back to the building table
5. Map stages to percentage completion
    - Each stage is associated with a typical completion percentage range (0–100%).
    - assign representative value per stage (e.g., around 0%, 15%, 35%, 65%, 90–100%) to treat the stage prediction as an estimated fraction of construction completed.

## Construction completion date estimation
After classifying each satellite image into one of the five construction stages (1–5), we estimated the construction progress percentage and used it to back-calculate the total project duration.

**Stage-to-progress mapping:**

| Stage | Meaning                   | Approx. % Complete |
| ----- | ------------------------- | ------------------ |
| 1     | Undeveloped Land          | 0%                 |
| 2     | Ground Broken             | 1–25%              |
| 3     | Concrete Pad              | 25–45%             |
| 4     | Framing Going Up          | 45–85%             |
| 5     | Near Completion/Completed | 85–100%            |

We then applied a simple proportional formula to estimate total construction time:

Estimated Total Duration = (Time Elapsed Since Ground Break) / (Progress % / 100)

Estimated Remaining Time = Estimated Total Duration - Time Elapsed


Where:

* **Time Elapsed** = (Current Image Date – Ground Break Date)
* **Progress %** = Stage Progress / 100

This allows us to forecast **how long the project will take to complete**, assuming linear construction progress, and estimate **future completion quarter** for each site.

## Problems & Next Steps

* **Limited dataset size**

  * Only 222 labeled construction images available.
  * Increases risk of overfitting and reduces generalization.

* **Manual labeling of satellite images**

  * Time-consuming and labor-intensive.
  * Possible mismatch between address and satellite coordinates.
  * Visual ambiguity when identifying construction stage from top-down imagery.
  * Missing/unknown timestamps for imagery affects progress estimation accuracy.

### Future Improvements

* **Increase dataset quality and size**

  * Integrate **U.S. Census block GeoJSON data** for easier geographic referencing.
  * Expand training dataset and collect multiple timestamps per site.

* **Improve model performance**

  * Conduct **hyperparameter tuning** (random search for LR, batch size, dropout, etc.).
  * Explore more architectures (EfficientNet, ViT, UNet for segmentation).

* **Enhance time-estimation accuracy**

  * Incorporate **satellite image capture date** into dataset.
  * Adjust progress-based duration formula accounting for non-linear construction stages.

* **Error investigation**

  * Analyze cases where **predicted vs. actual construction stage** differs.
  * Use explainability methods (Grad-CAM, feature visualization).

