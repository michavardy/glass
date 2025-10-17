# Glass

## ML email spam filter: proof oc concept

This project is a proof-of-concept (PoC) for an email spam filter using machine learning. It consists of two main services:

# Overview

## Data Collection Service

- Connects to Gmail accounts via OAuth2.

-Fetches historical emails.

-Subscribes to new email events or actions (delete, move, read, label).

- Stores emails and actions in a local database (JSON for PoC).

## Spam Suggestion Service (later)

- Uses collected data to train an ML model.

- Suggests whether new emails are spam.

# Project Setup

## Setup remote Data Server

- setup remote VM server that will listen to gmail events and record them in database

## Setup remote database

- setup remote vm database that will record 
    - user OAth2 access tokens and refresh tokens
    - historical email content
    - user actions
    - push subscription events

## Setup remote user registration landing page

- simple landing page that will contain a butten that will allow a user to sign up for our service.

- a user will click on the button and allow Oath2 registration

## Gmail API + OAuth2 Setup

### App Registration

1. google cloud console create project
2. enable gmail api
3. create OAth2 creds
    a. web app
    b. client id or client secret
4. oath2 flow

    a. user clicks connect gmail

    b. redirects to google oath2 consent screen with scopes

    ```
    https://www.googleapis.com/auth/gmail.readonly
    https://www.googleapis.com/auth/gmail.modify
    https://www.googleapis.com/auth/gmail.metadata
    ```

    c. user logs in and consents -> app gets authorization code

    d. exchange auth code for access token + refresh token

    e. save refresh token securely for refreshing access

### Historical data

    a. use api

    ### Request

    ```bash
    curl GET https://gmail.googleapis.com/gmail/v1/users/me/messages/{id}
    ```

    ### Response

    ```bash
    {
      "user_id": "user123",
      "emails": [
        {
          "id": "abcd123",
          "threadId": "thread1",
          "labelIds": ["INBOX", "IMPORTANT"],
          "snippet": "Hello world",
          "historyId": "987654321",
          "internalDate": 1700000000000,
          "payload": { ... }
        }
      ]
    }
    ```
### Listening for Actions

    a. enable google cloud pub/sub

    b. setup a watch on each user mailbox

#### Subscribe to Pub/Sub

    a. setup push subscription to topic

    ```bash
    gcloud pubsub subscriptions create my-subscription \
        --topic=my-topic \
        --push-endpoint=https://your-server.com/    pubsub-endpoint \
        --push-auth-service-account=your-service-account@proje  ct.iam.gserviceaccount.com
    ```

### User Actions

I want to record the following user actions:

    - delete

    - label

    - read

# Spam Suggestion Development

## UI

- the app will expose all suggestions in a UI that will group similar suggestions together

- The app will sort suggestions based on internal confidence scoring mechanism

- all user input into the app will be used for training ML

- user can delete, ignore or label emails

## Spam suggestion Algorithems

- A list of spam suggestion algorithems can be proposed from most simple to most complex 

- each algorithem will take labeled data from user or from other users and take unlabeled emails and classifiy them as delete, ignore, label and also give them a confidence score.

- this list of algorithems will be presented as most simple to most complex with the understanding that the most simple and effective method or set of methods should always be selected over complex methods.

- it makes sense to compose, or use a multi-layer approach such that multiple algorithems could be used together.

- each algorithem is dependent on labeled data, so the more labaled data that can be collected, the better the algorithems will perform.

- an algirthem evaluation should be used to check algorithem affectivness by partially hiding some of the labels and tesitng if the algorithem correctly predicts the labels that are currently known.

### Algorithem list

#### same sender

- given a labeled email (delete, ignore or label)
-  mark the rest of the emails with the same sender as the same action

- high confidence score

#### Similar Subject

- given a labeled email (delete, ignore or label)

- embed the email subject into a vector using embedding algorithem

- embedd all of the historical email subjects using embedding algorthem

- find a minimal distance between email subjects and mark those emails by original label

- high confidence score

#### Similar Content

- given a labeled email (delete, ignore or label)

- embed the email content into a vector using embedding algorithem

- embedd all of the historical email content using embedding algorthem

- find a minimal distance between email subjects and mark those emails by original label

- high confidence score

#### Classify Email Type: LLM

- given a labeled email (delete, ignore or label)

- given a maintained list of email types

- use LLM as classifier to classify or tag email based on predefined list

- send historical emails to LLM as classifier and classify historical emails

- find similar classification between historical emails to labeled emails and propose same label

- medium confidence score

#### Classify email using frequency: using sender

- given a labeled email (delete, ignore or label)

- given a historical email list

- characterize a base frequency with which the same sender sends emails

- characterize frequency per sender

    - time of day

    - messages per month

    - is always on the same day of week?

    - is always on same date of month?

- use similar frequency to labeled email to classify historical emails with same label



#### Classify email using keywords: Cutoff

- given a labeled email (delete, ignore or label)

- given a set of keywords extracted from this email: (see keyword extraction below)

- search throughout the set of historical emails for a similar composition or presence of keywords

- given a certain cutoff historical emails can be classified like labeled emails

#### Classify email using keywords: One Hot Encoding

- given a labeled email (delete, ignore or label)

- given a set of keywords extracted from this email: (see keyword extraction below)

- generate a one hot encoding vector of these keywords

- dimensionality reduction (Optional)

    - use one hot auto encoder Neural Net
    - use embeddings to merge similar keywords

- using cosine similarity or other such vector distance similarities find similar emails from historical emails and label according to labeled email


##### Extract Keywords: LLM

- given a labeled email (delete, ignore or label)

use LLM to extract a set of keywords that characterize this email


##### Extract Keywords: NLP

- Use the following techniques over the entire sent of emails to try and extract keywords

- remove stop words using nltk

    - TF
        - frequent words are candiate for keywords
    
    - TF-IDF

        - frequent words normalized by number of emails containing term
    
    - NLP Libraries

        - RAKE: rapic automatic keyword extraction

        - KeyBERT: embeddings for semantic keyword extraction

        - YAKE: yet another keyword extractor

- **NOTE** 
    - this can be done per user or over all emails

#### ML Classification: features

- use the following features to classify

    - sender domain meta-data

        - isAccepted

        - isUnverified

        - isMissSpelled

        - isGeneric

    - sender email 

        - one-hot

        - embedding

    - sender meta-data

        - isName

        - isCompany

        - isGeneric

        - isSuspicious

    - frequency of emails

    - Subject features

        - subject embeddings similarity to labeled

        - subject length

        - subject keyword similarity to labeled

    - Content features

        - content embeddings similarity to labeled

        - content length

        - content keyword similarity to labeled 

        - contains links



#### ML Classification: LLM extract features

using LLM we can extract additional features

    - email intent / porpuse
    
        - promotion
        
        - personal
        
        - notification
        
        - phishing
        
        - scam
        
        - social
        
    - sentiment: positive, negative, neutral
    
    - isManipulative
    
    - emotion
    
        - urgency
        
        - fear
        
        - excitment
        
        - flattery
        
    - formality
    
        - informal
        
        - formal
        
        - grammatical correctness
        
        - verbosity
        
        - usuall phrasing
        

##### ML classifications: Models

###### Simple / interpretable:
- Logistic Regression
- Random Forest
- Gradient Boosting (XGBoost, LightGBM)

###### Deep Learning

- FeedForward Neural Networks

- Fine Tune LoRA / Adapters using parameter-efficent fine tuning
- inject structured features as embeddings concatenated to LLM hidden layers





