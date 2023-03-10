# Doopler

Doopler is a non-functioning, sort-of-look-a-like, Doppler Node application. It's used for the take-home exercise for the Growth Engineer interview process.

## Getting Started

```
npm install

cd server
npm install

cd ../frontend
npm install

cd ..
npm start
```

## Frontend

The frontend is a very simple React application, written in Typescript. It has two main pages, a Secrets page and a Members page. Neither page actually functions - you can neither add secrets, nor add members.

### Analytics

The frontend app is able to record user events via the `anlytics` module. This will make a request to server and record it in the SQLite database included. Use this to track user events in your A/B testing experiments.

```js
import analytics from '../lib/analytics';

analytics.track('Page.Viewed');
```

### Context

A context can be used in components to get information about the "request". The only thing present currently is a generated User ID.

```js
import {Context, getContext} from '../lib/context';

const ctx: Context = getContext();
ctx.userId;
```

## Backend

The backend is an Express API. It has three endpoints: 

The POST events endpoint is used to collect analytics events.

```http
POST /events
Content-Type: application/json

{ "name": "Event.Name" }
```

The POST experiments endpoint is used to collect experiment activation data.

```http
POST /experiments
Content-Type: application/json

{
    "userId": 12345,
    "experimentName": "add_members_exp",
    "experimentGroup": "enabled",
}
```

The POST loggings endpoint is used to collect frontend loggings.

```http
POST /loggings
Content-Type: application/json

{
    "userId": 12345,
    "component": "ADD_MEMBER_BUTTON",
    "action": "CLICK",
}
```

### analytics.sqlite

A SQLite database is already included, with a table called `events` already created.
A table called `experiments` is created to track experiment activations.
A table called `frontend_loggings` is to track frontend loggings.

### Experiment;

The Experiment class is able to activate experiments on users by bucketing users into enabled and control groups by user ids.

```js

import Experiment from '../lib/experiment'

// Create an experiment instance with experiment group to bucket users into experiments 
const newExperiment = new Experiment('add_members_exp') 

// Activate experiment for user and save experiment activation to DB experiments table
const {isEnabled} = newExperiment.activate() 

if(isEnabled) {
    // do something for enabled group users
}

```

## Experiment analysis

To analyze the experiment outcome, join `experiments` table and `frontend_loggings` table on `user_id` to track user actions.

Using Add Members Experiment as example: 

```sql

SELECT
    experiment_group,
    COUNT(DISTINCT log.user_id) AS add_member_button_clicks
FROM
    (
        SELECT 
            user_id,
            experiment_group
        FROM
            experiments
        WHERE
            experiment_name = 'add_members_exp'
    ) exp
LEFT JOIN
    (
        SELECT
            user_id
        FROM
            frontend_loggings
        WHERE
            dt >= '2023-02-06' -- eexample xperiment launch date
            AND component = 'ADD_MEMBER_BUTTON'
            AND action = 'CLICK'
    ) log ON log.user_id = exp.user_id
GROUP BY 1
 
```


