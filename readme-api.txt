1. 
when "*/crm-api-graphql/*"  pattern is called in node server the make call through apolo client with provision for query and mutation  -  use .js for app.js in server 


To set up your Node.js server so that it handles requests with the pattern */crm-api-graphql/*, and then makes calls through Apollo Client for both queries and mutations, follow these steps. I'll provide the setup using .js for your app.js file.

Step 1: Install Necessary Packages
First, ensure you have the necessary packages installed in your server directory:

bash
Copy code
cd server
npm install express axios @apollo/client graphql
Step 2: Create Apollo Client in Node.js
Create an Apollo Client instance in a new file called apolloClient.js:

javascript
Copy code
// server/apolloClient.js
const { ApolloClient, InMemoryCache, HttpLink, gql } = require('@apollo/client');
const fetch = require('node-fetch');

const apolloClient = new ApolloClient({
  link: new HttpLink({
    uri: 'https://external-graphql-api.com/graphql', // Replace with the actual GraphQL endpoint
    fetch: fetch,
  }),
  cache: new InMemoryCache(),
});

module.exports = { apolloClient, gql };
Step 3: Modify app.js to Handle the Pattern and Call Apollo Client
Update your app.js to route requests matching the pattern */crm-api-graphql/* and make the appropriate Apollo Client call:

javascript
Copy code
// server/app.js
const express = require('express');
const { apolloClient, gql } = require('./apolloClient');
const bodyParser = require('body-parser');
const http = require('http');
const https = require('https');
const fs = require('fs');
const path = require('path');

const app = express();
const port = process.env.PORT || 5000;

// Middleware to parse JSON requests
app.use(bodyParser.json());

// Route for handling CRM GraphQL API requests
app.post('/crm-api-graphql/query', async (req, res) => {
  const { query, variables } = req.body;

  try {
    const result = await apolloClient.query({
      query: gql`${query}`,
      variables: variables,
    });
    res.json(result.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/crm-api-graphql/mutation', async (req, res) => {
  const { mutation, variables } = req.body;

  try {
    const result = await apolloClient.mutate({
      mutation: gql`${mutation}`,
      variables: variables,
    });
    res.json(result.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Serve static files from the React app
app.use(express.static(path.join(__dirname, '../client/build')));

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, '../client/build/index.html'));
});

const httpServer = http.createServer(app);

const httpsOptions = {
  key: fs.readFileSync('path/to/your/private.key'),
  cert: fs.readFileSync('path/to/your/certificate.crt'),
};

const httpsServer = https.createServer(httpsOptions, app);

httpServer.listen(port, () => {
  console.log(`HTTP Server running on port ${port}`);
});

httpsServer.listen(8443, () => {
  console.log(`HTTPS Server running on port 8443`);
});
Step 4: Example of Making Requests to the Server
Your React frontend (or any client) can now make POST requests to /crm-api-graphql/query for GraphQL queries and /crm-api-graphql/mutation for mutations. Here’s an example of how you could do this with Axios:

javascript
Copy code
// client/src/api/companyService.js
import axios from 'axios';

export const fetchCompanyData = async (companyId) => {
  const query = `
    query GetCompany($id: String!) {
      company(id: $id) {
        id
        name
        industry
        description
      }
    }
  `;

  const variables = { id: companyId };

  const response = await axios.post('/crm-api-graphql/query', { query, variables });
  return response.data;
};

export const updateCompanyData = async (companyId, name, industry) => {
  const mutation = `
    mutation UpdateCompany($id: String!, $name: String!, $industry: String!) {
      updateCompany(id: $id, name: $name, industry: $industry) {
        id
        name
        industry
        description
      }
    }
  `;

  const variables = { id: companyId, name, industry };

  const response = await axios.post('/crm-api-graphql/mutation', { mutation, variables });
  return response.data;
};
Summary
Apollo Client Setup: Configured in apolloClient.js to interact with an external GraphQL API.
Express Routes: Set up routes in app.js to handle requests to /crm-api-graphql/query and /crm-api-graphql/mutation, which then make corresponding GraphQL API calls using Apollo Client.
Client Usage: Your React app can make requests to these endpoints to fetch or mutate data using GraphQL.
This setup allows your Node.js server to act as a middle layer that interacts with an external GraphQL API, handling both queries and mutations, and exposing those operations via the specified routes.






2. 
when "*/crm-api-axios/*"  pattern is called in node server the make call through axios  with provision for query and mutation  -  use .js for app.js in server 

To set up your Node.js server to handle requests matching the pattern */crm-api-axios/* and then make calls through Axios for both query (GET requests) and mutation (POST/PUT/DELETE requests), follow these steps. The code will use .js for app.js.

Step 1: Install Axios
Ensure that Axios is installed in your server directory:

bash
Copy code
cd server
npm install axios
Step 2: Modify app.js to Handle the Pattern and Call Axios
Update your app.js to route requests matching the pattern */crm-api-axios/* and make the appropriate Axios call based on the request method:

javascript
Copy code
// server/app.js
const express = require('express');
const axios = require('axios');
const bodyParser = require('body-parser');
const http = require('http');
const https = require('https');
const fs = require('fs');
const path = require('path');

const app = express();
const port = process.env.PORT || 5000;

// Middleware to parse JSON requests
app.use(bodyParser.json());

// Route for handling CRM API requests through Axios
app.all('/crm-api-axios/*', async (req, res) => {
  const apiUrl = 'https://external-api.com'; // Replace with the actual API base URL
  const apiEndpoint = req.originalUrl.replace('/crm-api-axios', ''); // Get the endpoint from the original URL

  try {
    let result;

    // Handle GET requests (queries)
    if (req.method === 'GET') {
      result = await axios.get(`${apiUrl}${apiEndpoint}`, {
        params: req.query,
      });
    }

    // Handle POST requests (mutations)
    else if (req.method === 'POST') {
      result = await axios.post(`${apiUrl}${apiEndpoint}`, req.body);
    }

    // Handle PUT requests (mutations)
    else if (req.method === 'PUT') {
      result = await axios.put(`${apiUrl}${apiEndpoint}`, req.body);
    }

    // Handle DELETE requests (mutations)
    else if (req.method === 'DELETE') {
      result = await axios.delete(`${apiUrl}${apiEndpoint}`);
    }

    // Send the response data back to the client
    res.json(result.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Serve static files from the React app
app.use(express.static(path.join(__dirname, '../client/build')));

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, '../client/build/index.html'));
});

const httpServer = http.createServer(app);

const httpsOptions = {
  key: fs.readFileSync('path/to/your/private.key'),
  cert: fs.readFileSync('path/to/your/certificate.crt'),
};

const httpsServer = https.createServer(httpsOptions, app);

httpServer.listen(port, () => {
  console.log(`HTTP Server running on port ${port}`);
});

httpsServer.listen(8443, () => {
  console.log(`HTTPS Server running on port 8443`);
});
Step 3: Example of Making Requests to the Server
Your React frontend (or any client) can now make requests to /crm-api-axios/* endpoints. Here’s how you could do this with Axios:

javascript
Copy code
// client/src/api/companyService.js
import axios from 'axios';

// Fetch company data
export const fetchCompanyData = async (companyId) => {
  const response = await axios.get(`/crm-api-axios/companies/${companyId}`);
  return response.data;
};

// Create a new company
export const createCompany = async (companyData) => {
  const response = await axios.post('/crm-api-axios/companies', companyData);
  return response.data;
};

// Update an existing company
export const updateCompany = async (companyId, companyData) => {
  const response = await axios.put(`/crm-api-axios/companies/${companyId}`, companyData);
  return response.data;
};

// Delete a company
export const deleteCompany = async (companyId) => {
  const response = await axios.delete(`/crm-api-axios/companies/${companyId}`);
  return response.data;
};
Summary
Axios Integration: The Node.js server now has routes configured to handle requests to /crm-api-axios/* by making Axios calls to an external API.
Method Handling: The app.js file dynamically handles GET, POST, PUT, and DELETE requests, enabling both queries and mutations.
Client Interaction: The React client can use the defined service functions to interact with the server, which will proxy these requests to the external API.
This setup allows your Node.js server to act as a middle layer that makes HTTP requests to external APIs via Axios, handling both data fetching (queries) and data manipulation (mutations).






