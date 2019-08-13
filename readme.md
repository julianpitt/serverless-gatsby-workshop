# Serverless Gatsby blog workshop
by Julian Pittas

In this workshop, we'll be using the AWS amplify command line, react and Gatsby to create our very own serverless blog. We'll set up the appropriate pipelines to deploy our blog posts and allow users to comment.


## The Stack

- Frontend:
  - React
  - Gastby
- Backend:
  - NodeJS v10
  - Amplify
- Infrastructure:
  - DynamoDB
  - Amplify Console
  - Lambda
  - AppSync
  - Github

## Requirements

### AWS Account
Please follow the instructions found at the following link. Please note that you will need a credit card.
https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/

### Github Account
If you don't already have one, please signup at github.com

### Local Machine
You will need a machine that has:
- Administrator privileges
- NodeJS runtime v10
- AWS CLI (w/ prerequisite of python)
- AWS Amplify CLI
- Gatsby CLI
- GIT CLI

To confirm that all the above is installed and working correctly, you should get no errors when running the following commands:
```bash
node -v
aws --version
amplify -v
gatsby -v
git --version
```

#### NodeJS v10

##### Installing
Please see instructions outlined at the following links and make sure to download a version of node js that starts with 10.:
- Windows: https://tecadmin.net/install-node-js-on-windows/
- Mac/Unix: https://tecadmin.net/install-nodejs-with-nvm/

##### Testing
```bash
node -v
```

#### AWS CLI

##### Installing
Please see instructions outlined at the following links:
- Windows: https://docs.aws.amazon.com/cli/latest/userguide/install-windows.html
- Mac: https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html
- Linux: https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html

##### Configuration
Make sure your configure your AWS CLI with your AWS account.
Instructions can be found at this link: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-quick-configuration

##### Testing
```bash
aws --version
aws s3 ls
```

#### AWS Amplify CLI

##### Installing
https://aws-amplify.github.io/docs/cli-toolchain/quickstart

##### Testing
```bash
amplify -v
```

#### Gatsby Command line

##### Installing

```bash
npm install -g gatsby-cli
```

##### Testing
```bash
gatsby -v
```


## Instructions

The instruction for this workshop will be broken into different sections to make the overall workshop more manageable.
The Parts of the workshop each correspond to a new technology we will be introducing into the project or a body of work for a new feature.

Overview:
- Part 1: Setting up our gatsby bloc
- Part 2: Setting up amplify
- Part 3: Deploying to Amplify and auto build on commit
- Part 4: Setting up a graphql sources
- Part 5: Implementing dynamic behaviour

### Part 1 - Setting up our gatsby bloc

1. Copy the code from the gatsby blog into a new directory

```bash
gatsby new my-serverless-blog https://github.com/gatsbyjs/gatsby-starter-blog
```

2. Change into the new directory and start up your server

```bash
cd my-serverless-blog
gatsby develop
```

3. Confirm the site is running on localhost and that you can reach the graphql page

```bash
http://localhost:8000
http://localhost:8000/___graphql
```

4. Try adding another directory to blog and make sure there's an index.md file in the directory. Confirm your blog updates when you add another entry

5. Now commit this to your github account
```
git init -y
git add .
git commit -m "Initial commit"
```


### Part 2 - Setting up amplify 

Right now, the gatsby blog page is serving content from the file system, specifically the /content/blog directory.
If you add another directory inside the content blog directory with an md file, a new blog entry will appear on the page.


Lets get our sources using GraphQL instead

1. Install the Gatsby-source-graphql package

```bash
npm install --save gatsby-source-graphql
```

### Part 3 - Setting up amplify

1. We want to make sure that we are able to post the site to amplify.
To do this, we can use the amplfy command line utility

```bash
amplify init
```

This will ask you a number of questions. Make sure you choose as below

```bash
? Enter a name for the project: my-serverless-blog
? Enter a name for the environment: dev
? Choose your default editor: {you can choose your own editor}
? Choose the type of app that you're building: javascript

Please tell us about your project
? What javascript framework are you using: react
? Source Directory Path:  src
? Distribution Directory Path: build
? Build Command:  npm run-script build
? Start Command: npm run-script start

Using default provider  awscloudformation

For more information on AWS Profiles, see:
https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html

? Do you want to use an AWS profile?: Yes
? Please choose the profile you want to use: default
```

This will then deploy the CloudFormation script to your chosen AWS environment

2. We now need to add a GraphQL API to our project so run the following amplify commend

```
amplify add api
```

Which will then guide you through creating the schema, so please choose as follows:

```bash
? Please select from one of the below mentioned services: GraphQL
? Provide API name: serverlessblogapi
? Choose an authorization type for the API: API key
? Do you have an annotated GraphQL schema?: No
? Do you want a guided schema creation?: Yes
? What best describes your project: One-to-many relationship (e.g., “Blogs” with “Posts” and “Comments”)
? Do you want to edit the schema now?: No

GraphQL schema compiled successfully.
Edit your schema at /Users/julian/Documents/GIT/demos/workshop-gatsby-serverless-blog/my-serverless-blog/amplify/backend/api/serverlessblogapi/schema.graphql or place .graphql files in a directory at /Users/julian/Documents/GIT/demos/workshop-gatsby-serverless-blog/my-serverless-blog/amplify/backend/api/serverlessblogapi/schema
Successfully added resource serverlessblogapi locally

Some next steps:
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud
```

3. This will then create the schema files we need to the following path
/amplify/backend/api/serverlessblogapi/schema.graphql

We need to edit this file so it looks like the file below

```graphql
type Post @model {
  id: ID!
  title: String!
  content: String!
  comments: [Comment] @connection(name: "PostComments")
}
type Comment @model {
  id: ID!
  content: String
  post: Post @connection(name: "PostComments")
}
```

4. Deploy the backend with

```bash
amplify push
```

This will ask a few more questions:
```bash
? Do you want to generate code for your newly created GraphQL API: Yes
? Choose the code generation language target: javascript
? Enter the file name pattern of graphql queries, mutations and subscriptions: src/graphql/**/*.js
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions: Yes
? Enter maximum statement depth [increase from default if your schema is deeply nested]: 2
```

And then it will deploy your API

5. Update the current gatsby config file to point to our new datasource
Open up `gatsby-config.js` and replace this:

```javascript
{
    resolve: `gatsby-source-filesystem`,
    options: {
    path: `${__dirname}/content/blog`,
    name: `blog`,
    },
},
```

with this:

```javascript
{
    resolve: `gatsby-source-graphql`,
    options: {
        typeName: `blog`,
        fieldName: `blog`,
        url: '',
        headers: {
            'x-api-key': ''
        }
    },
},
```

You will then need to populate the `url` and `x-api-key` fields with the values found in `src/aws-exports.js`
populate the `url` with the value of `aws_appsync_graphqlEndpoint` in the exports file and
populate the `x-apli-key` with the value of `aws_appsync_apiKey` in the exports file

Now try running the frontend with `npm start`
You should see nothing appearing but your bio

### Part 4 - Getting graphql working

1. Adding mock data to your site
Now that we have added appsync as a data source, we can test it out by adding some mock data into our database
Open up your AWS console, then navigate to AWS AppSync, click on your api, then click on Queries
In the Query box, type out the following:
```gql
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    id
    title
    slug
    content
    excerpt
    date
  }
}
```
And set the Query Variables box with the following:
```json
{
  "input": {
    "title": "Hello world",
    "slug": "hello-world",
    "content": "<h1>this is some content</h1>",
    "excerpt": "this is some content",
    "date": "12-08-2019"
  }
}
```
Then click play.

Once this worked you should see a screen like the following:

Now go back to your gatsby page and refresh
You should now see a new blog entry. But when you click on it.... the page crashes :'(

2. Adding dynamic pages for blog posts
We first need to install the dependencies for apollo client
Apollo client will allow us to call our graphql endpoint
```
npm install --save apollo-client apollo-link-http apollo-link-context apollo-cache-inmemory graphql react-apollo
``` 
Now we need to create a client in utls
```javascript
//https://www.apollographql.com/docs/react/recipes/authentication/
import AppSyncConfig from '../aws-exports';

import { ApolloClient } from 'apollo-client';
import { createHttpLink } from 'apollo-link-http';
import { setContext } from 'apollo-link-context';
import { InMemoryCache } from 'apollo-cache-inmemory';

const httpLink = createHttpLink({
  uri: AppSyncConfig.aws_appsync_graphqlEndpoint,
});

const authLink = setContext((_, { headers }) => {
  return {
    headers: {
      ...headers,
      "x-api-key": AppSyncConfig.aws_appsync_apiKey
    }
  }
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache()
});

export default client
```

Once we have the client set up, we can now call our AppSync api but we can inject this client on every page if needed using the 
gatsby-browser.js file
Open up this file and paste the following
```javascript
// custom typefaces
import "typeface-montserrat"
import "typeface-merriweather"
import React from 'react';
import { ApolloProvider } from 'react-apollo';
import { client } from './src/utils/client';

export const wrapRootElement = ({ element }) => (
  <ApolloProvider client={client}>
    {element}
  </ApolloProvider>
);
```

### Notes

- To update the schema, you can use edit the `schema.graphql` file
then push using `amplify api push`
- By default, the API will expire after 7 days if not used. To get around this, in /amplify/backend/api/{yourapiname}/parameters.json, add 
```json
{
  ... other config,
  "APIKeyExpirationEpoch": "-1"
}
```
then run `amplify push`
1. Creating a new api key in the app sync console
2. Overwriting "GraphQLAPIKeyOutput" with the new key in #current-cloud-backend/amplify-meta.json and the same file in backend/
3. I then created "APIKeyExpirationEpoch": "-1" as the above stated.
4. Executed `amplify push`
