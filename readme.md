{
 "cells": [
  {
   "cell_type": "raw",
   "metadata": {},
   "source": [
    "<code><script>\n",
    "var myUrl = 'http://andresa.me/fraud_analysis_oversampling/';\n",
    " \n",
    "if(window.top.location.href !== myUrl) {\n",
    "    window.top.location.href = myUrl;\n",
    "}\n",
    "</script></code>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "###  **This is a pretty long tutorial and I know how hard it is to go through, hopefully you may skip a few blocks of code if you need**\n",
    "\n",
    "One of the oldest problem in Statistics is to deal with unbalanced data, for example, surviving data, credit risk, fraud. \n",
    "\n",
    "Basically in any data where either your success rate is too high or too low, models are almost irrelavant. This comes from the fact that we use a criteria around 1% to measure the accuracy of a model, ie, if my model predicts in the testing set 99% of the success (or failure depending on what you are trying to do), the model is a hero.\n",
    "\n",
    "![title](images.png)\n",
    "\n",
    "What happens with unbalanced data is that the success metric happening in around 1% (usually less than 10%), so if you have no model and predicts success at 1%, then the model passes the accuracy criteria.\n",
    "\n",
    "In this post you will learn a 'trick' to deal with this type of data: *oversampling* and *undersampling*.\n",
    "In this project I will skip the descriptive analysis hoping that we all want to focus on fraud analysis a bit more.\n",
    "\n",
    "\n",
    "# Importing Libraries\n",
    "\n",
    "In an early stage we will use a few of the most popular libraries such as pandas, numpy, matplotlib, sklearn, but also some for this particular type of problem such as imblearn, mlxtend and my favorite for logistic regression statsmodels.\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "metadata": {},
   "outputs": [],
   "source": [
    "#import libraries\n",
    "\n",
    "#to plot stuff\n",
    "import pandas as pd\n",
    "import matplotlib.pyplot as plt\n",
    "%matplotlib inline\n",
    "\n",
    "import scikitplot as skplt\n",
    "import numpy as np\n",
    "\n",
    "\n",
    "#decision tree\n",
    "from sklearn.datasets import make_blobs\n",
    "from sklearn.tree import DecisionTreeClassifier\n",
    "from sklearn.ensemble import RandomForestClassifier\n",
    "from sklearn.datasets import make_classification\n",
    "from sklearn.metrics import classification_report\n",
    "\n",
    "#split training and testing\n",
    "from sklearn.model_selection import train_test_split\n",
    "\n",
    "from collections import Counter\n",
    "from imblearn.over_sampling import RandomOverSampler\n",
    "from imblearn.under_sampling import RandomUnderSampler\n",
    "\n",
    "#model evaluation\n",
    "from sklearn.metrics import confusion_matrix\n",
    "\n",
    "\n",
    "\n",
    "import statsmodels.api as sm\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Data Problem and Motivation\n",
    "\n",
    "I have been working on a similar problem at work and I could not find a good source of information with I wanted. I knew I had to perform unbalanced data treatment and I wanted to use Logistic Regression and so I had to pick parts of code around and sew them together myself (thus this blog post).\n",
    "\n",
    "The data used here was available in a kaggle competition 2 years ago in this link: https://www.kaggle.com/mlg-ulb/creditcardfraud. I have no extra knowledge of the data or data source whatsoever besides what is already in there. They do mention in the data description that a PCA analysis was performed and unfortunately, due to the nature of the business and privacy, they cannot release any additional information on the dataset.\n",
    "\n",
    "This is what the data looks like:\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Time</th>\n",
       "      <th>V1</th>\n",
       "      <th>V2</th>\n",
       "      <th>V3</th>\n",
       "      <th>V4</th>\n",
       "      <th>V5</th>\n",
       "      <th>V6</th>\n",
       "      <th>V7</th>\n",
       "      <th>V8</th>\n",
       "      <th>V9</th>\n",
       "      <th>...</th>\n",
       "      <th>V21</th>\n",
       "      <th>V22</th>\n",
       "      <th>V23</th>\n",
       "      <th>V24</th>\n",
       "      <th>V25</th>\n",
       "      <th>V26</th>\n",
       "      <th>V27</th>\n",
       "      <th>V28</th>\n",
       "      <th>Amount</th>\n",
       "      <th>Class</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>0.0</td>\n",
       "      <td>-1.359807</td>\n",
       "      <td>-0.072781</td>\n",
       "      <td>2.536347</td>\n",
       "      <td>1.378155</td>\n",
       "      <td>-0.338321</td>\n",
       "      <td>0.462388</td>\n",
       "      <td>0.239599</td>\n",
       "      <td>0.098698</td>\n",
       "      <td>0.363787</td>\n",
       "      <td>...</td>\n",
       "      <td>-0.018307</td>\n",
       "      <td>0.277838</td>\n",
       "      <td>-0.110474</td>\n",
       "      <td>0.066928</td>\n",
       "      <td>0.128539</td>\n",
       "      <td>-0.189115</td>\n",
       "      <td>0.133558</td>\n",
       "      <td>-0.021053</td>\n",
       "      <td>149.62</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.191857</td>\n",
       "      <td>0.266151</td>\n",
       "      <td>0.166480</td>\n",
       "      <td>0.448154</td>\n",
       "      <td>0.060018</td>\n",
       "      <td>-0.082361</td>\n",
       "      <td>-0.078803</td>\n",
       "      <td>0.085102</td>\n",
       "      <td>-0.255425</td>\n",
       "      <td>...</td>\n",
       "      <td>-0.225775</td>\n",
       "      <td>-0.638672</td>\n",
       "      <td>0.101288</td>\n",
       "      <td>-0.339846</td>\n",
       "      <td>0.167170</td>\n",
       "      <td>0.125895</td>\n",
       "      <td>-0.008983</td>\n",
       "      <td>0.014724</td>\n",
       "      <td>2.69</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1.0</td>\n",
       "      <td>-1.358354</td>\n",
       "      <td>-1.340163</td>\n",
       "      <td>1.773209</td>\n",
       "      <td>0.379780</td>\n",
       "      <td>-0.503198</td>\n",
       "      <td>1.800499</td>\n",
       "      <td>0.791461</td>\n",
       "      <td>0.247676</td>\n",
       "      <td>-1.514654</td>\n",
       "      <td>...</td>\n",
       "      <td>0.247998</td>\n",
       "      <td>0.771679</td>\n",
       "      <td>0.909412</td>\n",
       "      <td>-0.689281</td>\n",
       "      <td>-0.327642</td>\n",
       "      <td>-0.139097</td>\n",
       "      <td>-0.055353</td>\n",
       "      <td>-0.059752</td>\n",
       "      <td>378.66</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1.0</td>\n",
       "      <td>-0.966272</td>\n",
       "      <td>-0.185226</td>\n",
       "      <td>1.792993</td>\n",
       "      <td>-0.863291</td>\n",
       "      <td>-0.010309</td>\n",
       "      <td>1.247203</td>\n",
       "      <td>0.237609</td>\n",
       "      <td>0.377436</td>\n",
       "      <td>-1.387024</td>\n",
       "      <td>...</td>\n",
       "      <td>-0.108300</td>\n",
       "      <td>0.005274</td>\n",
       "      <td>-0.190321</td>\n",
       "      <td>-1.175575</td>\n",
       "      <td>0.647376</td>\n",
       "      <td>-0.221929</td>\n",
       "      <td>0.062723</td>\n",
       "      <td>0.061458</td>\n",
       "      <td>123.50</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>2.0</td>\n",
       "      <td>-1.158233</td>\n",
       "      <td>0.877737</td>\n",
       "      <td>1.548718</td>\n",
       "      <td>0.403034</td>\n",
       "      <td>-0.407193</td>\n",
       "      <td>0.095921</td>\n",
       "      <td>0.592941</td>\n",
       "      <td>-0.270533</td>\n",
       "      <td>0.817739</td>\n",
       "      <td>...</td>\n",
       "      <td>-0.009431</td>\n",
       "      <td>0.798278</td>\n",
       "      <td>-0.137458</td>\n",
       "      <td>0.141267</td>\n",
       "      <td>-0.206010</td>\n",
       "      <td>0.502292</td>\n",
       "      <td>0.219422</td>\n",
       "      <td>0.215153</td>\n",
       "      <td>69.99</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>5 rows × 31 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "   Time        V1        V2        V3        V4        V5        V6        V7  \\\n",
       "0   0.0 -1.359807 -0.072781  2.536347  1.378155 -0.338321  0.462388  0.239599   \n",
       "1   0.0  1.191857  0.266151  0.166480  0.448154  0.060018 -0.082361 -0.078803   \n",
       "2   1.0 -1.358354 -1.340163  1.773209  0.379780 -0.503198  1.800499  0.791461   \n",
       "3   1.0 -0.966272 -0.185226  1.792993 -0.863291 -0.010309  1.247203  0.237609   \n",
       "4   2.0 -1.158233  0.877737  1.548718  0.403034 -0.407193  0.095921  0.592941   \n",
       "\n",
       "         V8        V9  ...         V21       V22       V23       V24  \\\n",
       "0  0.098698  0.363787  ...   -0.018307  0.277838 -0.110474  0.066928   \n",
       "1  0.085102 -0.255425  ...   -0.225775 -0.638672  0.101288 -0.339846   \n",
       "2  0.247676 -1.514654  ...    0.247998  0.771679  0.909412 -0.689281   \n",
       "3  0.377436 -1.387024  ...   -0.108300  0.005274 -0.190321 -1.175575   \n",
       "4 -0.270533  0.817739  ...   -0.009431  0.798278 -0.137458  0.141267   \n",
       "\n",
       "        V25       V26       V27       V28  Amount  Class  \n",
       "0  0.128539 -0.189115  0.133558 -0.021053  149.62      0  \n",
       "1  0.167170  0.125895 -0.008983  0.014724    2.69      0  \n",
       "2 -0.327642 -0.139097 -0.055353 -0.059752  378.66      0  \n",
       "3  0.647376 -0.221929  0.062723  0.061458  123.50      0  \n",
       "4 -0.206010  0.502292  0.219422  0.215153   69.99      0  \n",
       "\n",
       "[5 rows x 31 columns]"
      ]
     },
     "execution_count": 3,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "#loads data\n",
    "df = pd.read_csv('creditcard.csv')\n",
    "df.head()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Factoring the X matrix for Regression\n",
    "If we take a closer look at the amount of success and failure in the original data, we have:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Percentage of Fraud: 0.173%\n"
     ]
    }
   ],
   "source": [
    "#define our original variables\n",
    "y = df['Class']\n",
    "features =list(df.columns[1:len(df.columns)-1])\n",
    "X = pd.get_dummies(df[features], drop_first=True)\n",
    "\n",
    "p = sum(y)/len(y)\n",
    "print(\"Percentage of Fraud: \"+\"{:.3%}\".format(p));"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Now we know how low is the probability of a fradulent transactions to happen in our dataset (which here it will be treated as our success event). \n",
    "\n",
    "# Separating Test and Training\n",
    "Let's separate the data in training and testing"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Percentage of Fraud in the test set: 0.172%\n",
      "Percentage of Fraud in the train set: 0.173%\n"
     ]
    }
   ],
   "source": [
    "#define training and testing sets\n",
    "X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)\n",
    "\n",
    "\n",
    "p_test = sum(y_test)/len(y_test)\n",
    "p_train = sum(y_train)/len(y_train)\n",
    "\n",
    "print(\"Percentage of Fraud in the test set: \"+\"{:.3%}\".format(p_test))\n",
    "print(\"Percentage of Fraud in the train set: \"+\"{:.3%}\".format(p_train))\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "This way the percentage of success was kept within the split. \n",
    "\n",
    "Now because of our unbalanced data we need the dataset to have balanced success rate in order to validate the model. There are two ways of doing it: \n",
    "* Oversampling: which is basically floading the dataset with success events in a way that the percentage of success is closer to 50% (balanced) then the original set.\n",
    "* Undersampling: which is reducing the unbalanced event, forcing the dataset to be balanced.\n",
    "\n",
    "## Method: Over Sampling\n",
    "\n",
    "First let's perform the Oversampling analysis:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Percentage of Fraud in the test set: 50.000%\n",
      "Percentage of Fraud in the train set: 50.000%\n"
     ]
    }
   ],
   "source": [
    "#oversampling\n",
    "ros = RandomOverSampler(random_state=42)\n",
    "X_over, y_over = ros.fit_resample(X_train, y_train)\n",
    "\n",
    "ros = RandomUnderSampler(random_state=42)\n",
    "X_under, y_under = ros.fit_resample(X_train, y_train)\n",
    "\n",
    "#write oversample dataframe as a pandas dataframe and add the column names\n",
    "#column names were removed from the dataframe when we performed the oversampling\n",
    "#column names will be useful down the road when we do a feature selection\n",
    "\n",
    "X = pd.DataFrame(X)\n",
    "names = list(X.columns)\n",
    "X_over = pd.DataFrame(X_over)\n",
    "X_over.columns = names\n",
    "\n",
    "X_under = pd.DataFrame(X_under)\n",
    "X_under.columns = names\n",
    "\n",
    "p_over = sum(y_over)/len(y_over)\n",
    "p_under = sum(y_under)/len(y_under)\n",
    "\n",
    "print(\"Percentage of Fraud in the test set: \"+\"{:.3%}\".format(p_over))\n",
    "print(\"Percentage of Fraud in the train set: \"+\"{:.3%}\".format(p_under))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Now the data has the same amount of success and failures.\n",
    "\n",
    "# Modeling with Logistic Regression\n",
    "\n",
    "In the snipped of code below we have three function:\n",
    "\n",
    "1. **remove_pvalues():** a function to perform feature selection. It removes features with a p-value higher than 5%, which is basically the probability of the weight for that feature being equal 0. In a regression fitting, the coefficients pvalue test if each individual feature is irrelevant for the model (null hypothesis) or otherwise (alternative hypothesis). If you want to know more the Wikipedia article on p-value is pretty reasanable (https://en.wikipedia.org/wiki/P-value)\n",
    "\n",
    "<br>\n",
    "\n",
    "2. **stepwise_logistic():** now we will repeat the process of removing the \"irrelavant features\" until there is no more fetaures to be removed. This function will loop through the model interactions until stops removing features.\n",
    "\n",
    "<br>\n",
    "\n",
    "3. **logit_score():** The output of a logistic regression model is actually a vector of probabilities from 0 to 1, the closer to 0 the more unlikely is of that record to be a fraud given the current variables in the model. And on another hand the closer to 1, the more likely is to be a fraud. For the purposes of this problem, if the probability is higher than 0.95 (which it was picked by me randomly and guttely speaking) I am calling that a 1. And then at the end, this function scores the predicted success rate against the real value of the testing set.\n",
    "\n",
    "Note: The IMO the threshold to call a fraud or success depends on how many false negative/positive you are willing to accept. In statistics, it's impossible to control both at the same time, so you need to pick and choose."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 21,
   "metadata": {},
   "outputs": [],
   "source": [
    "#the functions below perform the stepwise selection using backward method based on feature weight\n",
    "#this functionality is not created in python (specially for logistic regression) within the statsmodels package\n",
    "#the function orders the p-values from the highest to the lowest and then remove one at a time refitting the model \n",
    "#each time checking if the features change their relevance. \n",
    "\n",
    "#the second function scores the model rounding likelihood higher than  0.95 to 1 (assuming it's a fraud)\n",
    "\n",
    "\n",
    "#function to do the stepwise backward selection\n",
    "def new_stepwise(X, y, alpha):\n",
    "    import statsmodels.api\n",
    "    \n",
    "    #creates empty list\n",
    "    to_remove = ['']\n",
    "    \n",
    "    n_old = len(X.columns)\n",
    "    n_new = 0\n",
    "    \n",
    "\n",
    "    #have we stop seeing high p-values?\n",
    "    while n_old - n_new>0:\n",
    "        \n",
    "        n_old = len(X.columns)\n",
    "        \n",
    "        #run the model \n",
    "        lr=statsmodels.api.Logit(y, X)\n",
    "        lr = lr.fit(disp=0)\n",
    "        lr.pvalues = lr.pvalues.sort_values(ascending=False)\n",
    "        \n",
    "        k = 0\n",
    "        #are we at the last one of the pvalues\n",
    "        while k < n_old:\n",
    "\n",
    "\n",
    "            if lr.pvalues.iloc[k]<alpha: \n",
    "\n",
    "                k +=1\n",
    "            else:\n",
    "                \n",
    "\n",
    "                to_remove.append(lr.pvalues.index[k])\n",
    "                X = X[X.columns.difference(to_remove)]\n",
    "                break\n",
    "\n",
    "                \n",
    "        n_new = len(X.columns)\n",
    "\n",
    "\n",
    "    return X.columns\n",
    "\n",
    "def logit_score(model, X, y, threshold=0.95):\n",
    "    #function to score the logit model (only works for models that support .predict function)\n",
    "    y_t = model.predict(X)\n",
    "    y_t = y_t > threshold\n",
    "    z= abs(y_t-y)\n",
    "    score = 1- sum(z)/len(z)\n",
    "    return score\n",
    "    "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Performing Logistic Regression\n",
    "Now our guns are loaded, we just need to fire, below I am performing the feature selection and then the model fitting in the final X matrix:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 22,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Optimization terminated successfully.\n",
      "         Current function value: 0.209580\n",
      "         Iterations 14\n"
     ]
    }
   ],
   "source": [
    "#execute the functions above\n",
    "\n",
    "features = new_stepwise(X_over, y_over, 0.05)\n",
    "X_over_new = X_over[features]\n",
    "\n",
    "lr=sm.Logit(y_over,X_over_new)\n",
    "lr = lr.fit()\n",
    "   "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Scoring the model\n",
    "Now the most expected part of this tutorial, which is basically checking how many \"rights and wrongs\" we are getting, considering the model was created based on a dataset with a 50% of fraud occurrences, and then tested in a set with 0.17% of fraud occurrences.<br>\n",
    "**Voila:**"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 25,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Percentage of Rights and Wrongs in the testing set 99.261%\n"
     ]
    }
   ],
   "source": [
    "\n",
    "score = logit_score(lr, X_test[features], y_test)\n",
    "y_t = lr.predict(X_test[features])\n",
    "#rounds to 1 any likehood higher than 0.95, otherwise sign 0\n",
    "y_t = y_t > 0.95\n",
    "\n",
    "print(\"Percentage of Rights and Wrongs in the testing set \"+\"{:.3%}\".format(score));\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 26,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Number of Frauds in real data 98\n",
      "Number of Frauds in prediction data 491\n"
     ]
    }
   ],
   "source": [
    "#from the number below you can see if the model is over or under predicting\n",
    "print('Number of Frauds in real data %s' % sum(y_test))\n",
    "print('Number of Frauds in prediction data %s' % sum(y_t))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Now that we know the model is ok, let's see where we are getting it wrong. The plot below shows how many false negatives and false positives we have. We see a lot more false positives (we are saying a transaction was fradulent even though it was not). This come from the 0.95 threeshold above, if you increase that value to 0.99 for example, you will increase the amount of false negatives as well. The statistician need to decide what is the cut minimizes false negatives without affecting precision.\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 28,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<matplotlib.axes._subplots.AxesSubplot at 0x25a01912c50>"
      ]
     },
     "execution_count": 28,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 2 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "#plots confusion matrix with false positives and negatives\n",
    "skplt.metrics.plot_confusion_matrix(y_test, y_t)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "let's see what variables are more important to the model in asbolute value. In the chart below you can see the top 15 most relavant features:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 29,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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CHgAAACDcIgmAADAIMzgAQAADELAAwAAGMSqWRewHHe96137wAMPnHUZAAAAM3Heeed9q7v321y/nSLgHXjggTn33HNnXQYAAMBMVNXly+nnEE0AAIBBCHgAAACDEPAAAAAGsVOcgwcAAGy7m266KVdeeWVuuOGGWZfCrdhtt91ywAEHZJdddtmq5wt4AAAwJ6688srsscceOfDAA1NVsy6HRbo7V111Va688socdNBBW7UOh2gCAMCcuOGGG7LvvvsKdzuoqsq+++67TTOsAh4AAMwR4W7Htq37R8ADAAAYhHPwAABgTh34srNWdH2XvepxK7q+HcHv/M7v5JhjjsmjHvWoW+1z0kknZfXq1XnpS196i+XXXnttzjjjjDz/+c+/vcu8mRk8AABgh/f9739/Jts9+eSTbzPc3ZZrr702b3rTm1a4otsm4AEAANvFZZddlsMPPzzPfvazs2bNmvz8z/98vvvd7+b888/PT//0T+f+979/nvzkJ+eaa65Jkjz84Q/Pb//2b+dhD3tYXv/61+f444/P8573vDziEY/IwQcfnI997GN51rOelcMPPzzHH3/8rW73ne98Z1784hcnSV7/+tfn4IMPTpJccskleehDH5okOe+88/Kwhz0sD3zgA/PoRz86X/va15Ikxx9/fM4888wkyfve974cdthheehDH5oTTzwxj3/842/exhe+8IU8/OEPz8EHH5w3vOENSZKXvexlueSSS7J27dr85m/+Zr72ta/lmGOOydq1a3O/+90vZ5999sr+gCPgAQAA29GXv/zlvOAFL8i6detyl7vcJe9617vyzGc+M69+9avzuc99LkcccURe+cpX3tz/2muvzcc+9rG85CUvSZJcc801+fCHP5zXve51ecITnpAXvehFWbduXS688MKcf/75S27zmGOOuTlMnX322dl3333z1a9+Neecc06OPvro3HTTTXnhC1+YM888M+edd16e9axn5eUvf/kt1nHDDTfkOc95Tt7//vfnnHPOyfr162/RftFFF+UDH/hAPv3pT+eVr3xlbrrpprzqVa/KT/zET+T888/Pa17zmpxxxhl59KMfnfPPPz8XXHBB1q5du5I/2iTOwQMAALajgw466OZg88AHPjCXXHJJrr322jzsYQ9Lkhx33HH5pV/6pZv7P/WpT73F85/whCekqnLEEUdk//33zxFHHJEkWbNmTS677LIlQ9Pd7373bNy4Mdddd12uuOKKPP3pT8/HP/7xnH322fnFX/zFfOlLX8rnP//5/NzP/VyS5Ac/+EF+/Md//BbruOiii3LwwQfffH+6pz3taXnrW996c/vjHve47Lrrrtl1111zt7vdLd/4xjd+pI6f+qmfyrOe9azcdNNNedKTnnS7BDwzeAAAwHaz66673vz9He94x1x77bW32X/33Xdf8vl3uMMdbrGuO9zhDrd5nt7P/MzP5G1ve1sOPfTQHH300Tn77LPziU98Ig95yEPS3VmzZk3OP//8nH/++bnwwgvzwQ9+8BbP7+4tGtdStRxzzDH5+Mc/nnve85459thjc9ppp93mOreGgAcAAMzMXnvtlb333vvmQyhPP/30m2fzVtIxxxyT1772tTnmmGPygAc8IB/5yEey6667Zq+99sqhhx6a9evX5xOf+ESS5Kabbsq6detu8fzDDjssl156aS677LIkyTve8Y7NbnOPPfbIddddd/Pjyy+/PHe7293y7Gc/O7/2a7+Wz3zmMys3wCmHaAIAwJzaUW5rcOqpp+a5z31uvvOd7+Tggw/O2972thXfxtFHH50rrrgixxxzTO54xzvmXve6Vw477LAkyZ3udKeceeaZOfHEE7Nhw4Z8//vfz2/8xm9kzZo1Nz//zne+c970pjflMY95TO5617vmQQ960Ga3ue++++YhD3lI7ne/++Wxj31s7ne/++U1r3lNdtlll6xevfp2mcGrzU017giOOuqoPvfcc2ddBgAA7NS++MUv5vDDD591GTutjRs3ZvXq1enuvOAFL8ghhxySF73oRSu+naX2U1Wd191Hbe65DtEEAABYhr/4i7/I2rVrs2bNmmzYsCHPec5zZl3Sj3CIJgAAMIwHP/jB+d73vneLZaeffvrNV9vcFi960Ytulxm7lSTgAQDAHOnuVNWsy7jdfOpTn5p1CdtkW0+hc4gmAADMid122y1XXXXVNocIbh/dnauuuiq77bbbVq/DDB4AAMyJAw44IFdeeWXWr18/61K4FbvttlsOOOCArX6+gAcAAHNil112yUEHHTTrMrgdDR/wDnzZWTPZ7o5yTxEAAGB+OAcPAABgEAIeAADAIAQ8AACAQQh4AAAAgxDwAAAABiHgAQAADELAAwAAGISABwAAMAgBDwAAYBACHgAAwCAE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\n",
      "text/plain": [
       "<Figure size 1080x720 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "#calculate the weights\n",
    "weights = lr.params\n",
    "\n",
    "#creates a dataframe for them and the coefficient names\n",
    "importance_list = pd.DataFrame(\n",
    "    {'names': features,\n",
    "     'weights': weights\n",
    "    })\n",
    "\n",
    "#normalized absolute weights\n",
    "importance_list['abs_weights'] = np.abs(importance_list['weights'])\n",
    "total = sum(importance_list['abs_weights'])\n",
    "importance_list['norm_weights'] = importance_list['abs_weights']/total\n",
    "\n",
    "#select top 10 with higher importance\n",
    "importance_list = importance_list.sort_values(by='norm_weights', ascending=False)\n",
    "importance_list = importance_list.iloc[0:14]\n",
    "\n",
    "#plot them tcharam!\n",
    "ax = importance_list['norm_weights'].plot(kind='bar', title =\"Variable importance\",figsize=(15,10),legend=True, fontsize=12)\n",
    "ax.set_xticklabels(importance_list['names'], rotation=90)\n",
    "\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Now to visualize the weights we can see in the plot below which variables decrease x increase the likelihood of having a fradulent transaction. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 30,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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AsDysucNAABgAOINAABgAA6bBAAAGIA9bwAAAAMQbwAAAANYM+8B3P72t+/169fPexgAAABzccYZZ3ynu9fu6H5zj7f169dn8+bN8x4GAADAXFTV+Tfkfg6bBAAAGIB4AwAAGIB4AwAAGMDcz3kDAADGdtVVV+XCCy/M5ZdfPu+h7NH22WefrFu3LnvttddNerx4AwAAdsqFF16Y/fffP+vXr09VzXs4e6TuzsUXX5wLL7wwBx100E2ah8MmAQCAnXL55ZfnwAMPFG7Xo6py4IEH7tTeSfEGAADsNOG2Yzv7MxJvAADAUnrWs56Vc84553rv84xnPCPvec97fuL75513Xt71rnftqqFtl3PeAACAmVp/3Mkznd95r33UTOe3zZvf/Oab/Nht8XbsscfOcETXz543AABgaH/4h3+YP/mTP0mSvPjFL84xxxyTJPnoRz+apz71qfnQhz6UI444Ive9733zhCc8IZdeemmS5Oijj87mzZuTJG95y1ty8MEH5+ijj86zn/3svOAFL7hm/p/85CfzwAc+MPe4xz2u2Qt33HHH5dRTT82GDRvyhje8IV/4whdy//vfPxs2bMhhhx2WL3/5yzNfT/EGAAAM7aijjsqpp56aJNm8eXMuvfTSXHXVVTnttNNy6KGH5vjjj89HPvKRnHnmmdm4cWNe//rXX+vx3/jGN/KqV70qp59+ej784Q/n3HPPvdb0b37zmznttNNy0kkn5bjjjkuSvPa1r82RRx6Zs846Ky9+8Ytzwgkn5EUvelHOOuusbN68OevWrZv5ejpsEgAAGNr97ne/nHHGGdm6dWv23nvv3Pe+983mzZtz6qmn5jGPeUzOOeecPOhBD0qSXHnllTniiCOu9fjPfOYzefCDH5wDDjggSfKEJzwhX/rSl66Z/rjHPS43u9nNcq973SsXXXTRdsdwxBFH5NWvfnUuvPDC/Mqv/Eruec97znw9xRsAADC0vfbaK+vXr89b3/rWPPCBD8xhhx2Wj3/84/nqV7+agw46KA972MPy7ne/+zof393XO/+99957h/c99thjc/jhh+fkk0/Owx/+8Lz5zW++5vDNWXHYJAAAMLyjjjoqr3vd63LUUUflyCOPzAknnJANGzbkAQ94QD71qU/lK1/5SpLksssuu9ZetSS5//3vn0984hO55JJLcvXVV+e9733vDpe3//77Z+vWrdfc/trXvpZ73OMeeeELX5jHPOYx+fznPz/bFYw9bwAAwAI48sgj8+pXvzpHHHFE9t133+yzzz458sgjs3bt2rztbW/Lk5/85FxxxRVJkuOPPz4HH3zwNY+9y13ukpe+9KU5/PDDc+c73zn3ute9cpvb3OZ6l3fYYYdlzZo1ufe9751nPOMZufzyy/POd74ze+21V+50pzvlFa94xczXsXa0i3BX27hxY2+7wgsAADCeL37xiznkkEPmPYydcumll2a//fbL1Vdfncc//vF55jOfmcc//vEzX872flZVdUZ3b9zRYx02CQAALL1NmzZlw4YN+YVf+IUcdNBBedzjHjfvIf0Eh00CAABL73Wve928h7BD9rwBAAAMQLwBAAA7bd7X0hjBzv6MxBsAALBT9tlnn1x88cUC7np0dy6++OLss88+N3keznkDAAB2yrp163LhhRdmy5Yt8x7KHm2fffbJunXrbvLjxRt7pENPPHQuyz376WfPZbkAACPba6+9ctBBB817GAvPYZMAAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADWDPvAQDJF3/ukLks95BzvziX5QIAcOPZ8wYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADCAmcVbVe1dVW+pqvOramtVfbaqHjGr+QMAACyzWe55W5PkgiQPTnKbJC9P8rdVtX6GywAAAFhKa2Y1o+7+YZJNK751UlX9S5L7JTlvVssBAABYRrvsnLequmOSg5N8YTvTnlNVm6tq85YtW3bVEAAAABbGLom3qtoryV8lObG7z109vbvf1N0bu3vj2rVrd8UQAAAAFsrM462qbpbkHUmuTPKCWc8fAABgGc3snLckqapK8pYkd0zyyO6+apbzBwAAWFYzjbckf57kkCQP7e4fzXjeAAAAS2uWn/N29yTPTbIhybeq6tLp11NmtQwAAIBlNcuPCjg/Sc1qfgAAAPybXfZRAQAAAMyOeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABiAeAMAABjAmnkPAGDR/dGTHj2X5f7u35w0l+UCALuGPW8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADWDPvAQCwOC487tS5LHfda4+cy3IBYHey5w0AAGAA4g0AAGAA4g0AAGAAznkDgJto06ZNS7VcAObLnjcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABiDcAAIABzDTequoFVbW5qq6oqrfNct4AAADLbM2M5/eNJMcneXiSW8543gAAAEtrpvHW3e9LkqramGTdLOcNAACwzOZyzltVPWd6eOXmLVu2zGMIAAAAQ5n1YZM3SHe/KcmbkmTjxo09jzEAAPBv1h938lyWe95rHzWX5cKIXG0SAABgAOINAABgADM9bLKq1kznefMkN6+qfZJc3d1Xz3I5AAAAy2bWe95eluRHSY5L8tTpv18242UAAAAsnVl/VMCmJJtmOU8AAACc8wYAADAE8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADAA8QYAADCAmcZbVR1QVe+vqh9W1flVdews5w8AALCs1sx4fn+a5Mokd0yyIcnJVfW57v7CjJcDAACwVGa2562q9k3yH5K8vLsv7e7TknwgydNmtQwAAIBlVd09mxlV3SfJp7v7liu+95IkD+7u/23VfZ+T5DlJcre73e1+559//k1a5vrjTr7pA94J5732UXNZbjbdZg7L/P7uXyYL70+f97G5LPf5Jxwzl+XCovjox356Lsv9pWO+Opfl3unjZ81lud96yIa5LHeZ+B1ydy13Pr9HHnrioXNZ7tlPP/smP7aqzujujTu63yzPedsvyeot9P0k+6++Y3e/qbs3dvfGtWvXznAIAAAAi2mW8XZpkluv+t6tk2yd4TIAAACW0izj7UtJ1lTVPVd8795JXKwEAABgJ80s3rr7h0nel+SVVbVvVT0oyWOTvGNWywAAAFhWs/6Q7t9Kcssk307y7iS/6WMCAAAAdt5MP+etu7+b5HGznCcAAACz3/MGAADALiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABrBm3gMAAACYlbOffva8h7DL2PMGAAAwAHveAABgkW36/rxHwIzY8wYAADAA8QYAADAA8QYAADAA8QYAADCAmcRbVb2gqjZX1RVV9bZZzBMAAIB/M6urTX4jyfFJHp7kljOaJwAAAFMzibfufl+SVNXGJOtmMU8AANgVznvto+Y9BLhJnPMGAAAwgLnEW1U9Z3qO3OYtW7bMYwgAAABD2WG8VdUpVdXX8XXaTVlod7+puzd298a1a9felFkAAAAslR2e89bdR++GcQAAAHA9ZnLBkqpaM53XzZPcvKr2SXJ1d189i/kDAAAsu1md8/ayJD9KclySp07//bIZzRsAAGDpzeqjAjYl2TSLeQEAAPCTfFQAAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAAMQbAADAANbMewAAAHuibz1kw7yHAHAt9rwBAAAMwJ43AOAG+aVjvjrvIQAsNXveAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABiDeAAAABrBm3gMAls/zTzhm3kMAABiOPW8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAADEG8AAAAD2Ol4q6q9q+otVXV+VW2tqs9W1SNmMTgAAAAmZrHnbU2SC5I8OMltkrw8yd9W1foZzBsAAIBMwmundPcPk2xa8a2TqupfktwvyXk7O38AAAB2wTlvVXXHJAcn+cKs5w0AALCsZhpvVbVXkr9KcmJ3n3s993tOVW2uqs1btmyZ5RAAAAAW0g7jrapOqaq+jq/TVtzvZknekeTKJC+4vnl295u6e2N3b1y7du1OrwQAAMCi2+E5b9199I7uU1WV5C1J7pjkkd191c4PDQAAgG12+oIlU3+e5JAkD+3uH81ongAAAEzN4nPe7p7kuUk2JPlWVV06/XrKTo8OAACAJLP5qIDzk9QMxgIAAMB1mPlHBQAAADB74g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAA4g0AAGAAa+Y9AG6ETd+f9wgAAIA5secNAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgAOINAABgADOJt6p6Z1V9s6p+UFVfqqpnzWK+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\n",
      "text/plain": [
       "<Figure size 1080x720 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "ax = importance_list['weights'].plot(kind='bar', title =\"Variable importance\",figsize=(15,10),legend=True, fontsize=12)\n",
    "ax.set_xticklabels(importance_list['names'], rotation=90)\n",
    "\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "I know this is a pretty long tutorial but hopefully you will not need to go through all the yak shaving I had to go through what I went through."
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.7.3"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}
