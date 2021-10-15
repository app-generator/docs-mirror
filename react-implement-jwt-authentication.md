# JWT Authentication on React Templates

How to implement JWT Authentication to a React Template **for beginners**, step by step.

<br />

## Starting Point

To use the product, Node JS (>= 12.x) is required and GIT to clone/download the project from the public repository.

**Step #1** - Clone the given project

```bash
$ git clone https://github.com/user/project
$ cd project
```

<br >

**Step #2** - Install dependencies via NPM or yarn

```bash
$ npm i
// OR
$ yarn
```

<br />

**Step #3** - Start in development mode

```bash
$ npm run start 
// OR
$ yarn start
```

<br />

## Analyze The Codebase

**Step #1** - Look around the project and see how its content is structured

<br />

**Step #2** - Detect master pages

Often times the project is separated in 2 or 3 main sides:

- /auth
- /admin or /dashboard
- /rtl (right to left text)

<br />

**Step #3** - Detect authentication pages

Now you need to locate the authentication pages. Those are usually called **SignIn, Login, SignUp, Register**. The logout page does not always exist and you have to create it.

<br />

## Get To Coding

**Step #1** - Create the frontend API

Start by creating the frontend API using **axios**

- in **/src** you can create a new folder called "api"
- add a **index.js** file with the api configuration:

```javascript
import Axios from "axios";
import { API_SERVER } from "../config/constant";

const axios = Axios.create({
  baseURL: `${API_SERVER}`,
  headers: { "Content-Type": "application/json" },
});

axios.interceptors.request.use(
  (config) => {
    return Promise.resolve(config);
  },
  (error) => Promise.reject(error)
);

axios.interceptors.response.use(
  (response) => Promise.resolve(response),
  (error) => {
    return Promise.reject(error);
  }
);

export default axios;
});
```  

- next, add a **auth.js** file which will contain your API calls:

```javascript
import axios from "./index";

class AuthApi {

  static Login = (data) => {
    return axios.post(`users/login`, data);
  };

  // don't forget to add the register and logout methods
}

export default AuthApi;
```  

- create the backend API server address in a new folder in the **src** directory: `src/config/constant.js`

This is the **magic line** that connects the live product to the **Node JS API Server**

```javascript
export const API_SERVER = "http://localhost:5000/api/";
```

But in the development process, you'll actually need to use `https://api-server-nodejs.appseed.us/api/` in order to be able to properly test your auth calls

There, your API is all configured.