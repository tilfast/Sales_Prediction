# Sales Prediction
Sales retail prediction

## Project FAQ

### What ML model (algorithm) did you use? 

I initially thought that a LSTM / GRU model would solve easily this problem however the architecture started to be too complex for the time available, especially with the data structures. 

The model I started to put in place was disappointing. I had relied on a basic "at glance" EDA without noticing some crucial characteristics of the Data. It quickly backfired when the RMSLE ended up being higher than the naive model's!! So I reverted to a true and tested XGBoost regression but that still gave me results with high divergence in the prediction. I started to investigate more in depth the data and noticed that close to 30% of the target sales were actually zero. Making a regression in these condition made little sense and I switched to first identifying the products that would not sell the next day. It was well worth trying and gave me good results. 
Time pending, it would be worth doublechecking whether there is an overfit as the ROC is suspiciously high. However the validation data did not show any particular deviation, I therefore went along with the results of the classification. 


![ROC](https://github.com/tilfast/Sales_Prediction/assets/6140149/ad409cf4-38d7-41d7-9ed9-a85ca7e3ab9e)

This opportunity to gain good classification nailed (Sales/No Sale), I went for a regression of the products that were expected to sell the next day. The RMSLE for these was much better once the zero sales were taken away. 

Combining both models gave us a result that beat the naive model.  

#### The model was trained according to the following architecture:

![graph1](https://github.com/tilfast/Sales_Prediction/assets/6140149/4da4e906-66d9-4b39-8d6a-c547ba259132)

#### The final model is as follow: 

![graph2](https://github.com/tilfast/Sales_Prediction/assets/6140149/889c162a-9b97-43bb-ace0-e6fc752420be)

### What features did you use as an input for this ML model and how they were prepared from raw data? 

I looked at the following features, some where extended to cover several periods (such as 3, 7, 15, 30 days)
- *Sales lag:* to gain information on sales dependencies
- *Promotion lag: having promotion might lead to less sales further as people load up on goods
- *Day in week: to exploit the potential cyclical nature of the puchase behavior eg week-ends 
- *Day in month: same ,but at day in month level, maybe people spend less at the end of the month when their salary is spent 
- *Month in year: to try and capture seasonals trend such as New year spend behaviors 
- *Store:* we may observe some differences by store in sales   
- *Product type:* some products may sell more or less according to the situation (day in week, month, store)
- *Moving average:*  reasonable estimator of the sales trend
- *Standard deviation:* might tell us how far down or up a sale could go
- *Min sales within x days:* boundaries for sale may give us some information on where the sale could be
- *Max sales within x days:* same as above

 I wish I had added a few more features such as product turnover rate or daily difference rate which have some information useful for inventories and cashflow management.

I brute forced those features and only selected those that yielded the most information both for classification and regression with XGBoost.

#### Features of importance for the Classification:
![FeatClass](https://github.com/tilfast/Sales_Prediction/assets/6140149/ae248923-0220-4dec-a511-051620678c3e)


#### Features of importance for the Regression:
![FeatReg](https://github.com/tilfast/Sales_Prediction/assets/6140149/f1864a43-2d98-4f47-abdf-23992c09cfc6)

Re-running the basic models showed almost *no loss of performance* by reducing the number of features from 137 to 27. I did not check if less features could still give us good results. I was interested in keeping those top 27 features as they may help in interpreting the model’s information. 

 
It would have been good to also have holidays, special events such as "meaningful sport events", earthquakes, typhoons in the case of Japan to understand whether they could impact our sales prediction. Daily weather conditions could also be interesting to look at. 


### What is the RMSLE metric for dummy model? 

## 0.70 

### What is the RMSLE metric for developed model? 

## 0.54 

### What other metrics you think could be useful for this model evaluation? 
In my opinion, since a chain store is interested mainly in its bottom line, it would be interesting to develop a loss function and metrics based 
- Cost of goods: we want to penalize errors made on the most expensive products 
- Shelf-space of goods: large products on shelves may prevent the placement of smaller products 
- Shelf expiration date of goods: are we going to lose the products if it was over-predicted and will perish?
- Product utility: This is a very subjective indicator and may require domain knowledge from the merchants, but for instance, from a customer's perspective it may be a worse experience to have a lack of Baby formula than a shortage of bubblegum. 


### Provided design of prediction model service implementation 

Althought deployment is not my speciality, I understand the general concepts and architecture needs as follow:

The architecture should include 
- data extraction
- data validation
- feature building
- prediction by production model
- result monitoring (both output for customer and model drift)
- scalability

For more details:

It is important to have a **version control** and tracking the model deployed. Many people may work on the model, work on a different project or simple not remember what was put in production.

The model must also **be monitored** for inputs and output. We may have:
- New stores that are oppening for which no prior data to compute the features may exist
- Disruption in service at database level which may cut some rows of data that we will need to replace for the computation of the features
- Safety controls to alert a human operator when certain threshold for new orders are crossed and seem very costly or unusual
- Monitoring of the model that may show some drift over time

Once these controls are in place, the model will need to be put in a **container** (kubernetes, docker) to be easily manipulated from a system engineering/MLOps perpective and easily accessible to third parties. The container should be able to **scale up** according to demand for service.

We may also want to be sure to test the model in a **test environment** before deployment to a **production environment**. In parallel, the model can be regularly **updated automatically** with fresh data from production

API, Architecture, Data model, data characteristics should be **documented**

Customer (merchant) **feedback loop** should also be regularly maintained to make sure no unseen changes or issues are overlooked and derail the predictions. It can also lead to further business opportunties with the merchant.

This being said, rather than having one model fits all situations, we may develop several specialized models that might be more performing at a defined task.


